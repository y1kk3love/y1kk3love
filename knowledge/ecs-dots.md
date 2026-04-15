# ECS/DOTS

## 1. DOTS 3대 핵심

성능을 극한으로 뽑아내기 위한 유니티의 차세대 기술 스택.

- **ECS (Entity Component System):** 데이터 중심의 설계 구조. OOP의 성능 한계를 극복함.
- **C# Job System:** 멀티코어 프로세서를 효율적으로 사용하는 시스템. 메인 스레드 부하를 워커 스레드로 분산함.
- **Burst Compiler:** C# 코드를 고도로 최적화된 어셈블리로 변환함. C++급 성능을 보장함.

---

## 2. ECS 기본 구조 (E-C-S)

### Entity (엔티티)
- 아무 로직 없는 **단순한 ID(정수)**임.
- 데이터(컴포넌트)들을 담는 바구니 역할임.

### Component (컴포넌트)
- **순수 데이터**만 저장하는 구조체(`struct`)임.
- `IComponentData` 인터페이스 상속 필수. 반드시 값 타입이어야 함(class 불가).

### System (시스템)
- **실제 로직**을 실행하는 주체임.
- 특정 컴포넌트 조합을 가진 엔티티를 필터링(Query)해서 연산함.

---

## 3. 엔티티 및 컴포넌트 조작 (EntityManager)

### 엔티티 관리

```csharp
// 엔티티 생성 (빈 껍데기)
Entity entity = entityManager.CreateEntity(typeof(LocalTransform), typeof(MyData));

// 프리팹으로 엔티티 생성 (가장 흔한 방식)
Entity instance = entityManager.Instantiate(prefabEntity);

// 엔티티 삭제
entityManager.DestroyEntity(entity);
```

### 컴포넌트 데이터 핸들링

```csharp
// 데이터 추가
entityManager.AddComponentData(entity, new MyData { Value = 5 });

// 데이터 제거
entityManager.RemoveComponent<MyData>(entity);

// 데이터 수정
entityManager.SetComponentData(entity, new MyData { Value = 10 });

// 데이터 읽기
MyData data = entityManager.GetComponentData<MyData>(entity);

// 컴포넌트 존재 확인 (bool)
bool exists = entityManager.HasComponent<MyData>(entity);
```

---

## 4. Burst Compiler & Job System (성능의 핵심)

### Job System 규칙
- **NativeContainer 사용:** `List`, `Dictionary` 대신 `NativeArray`, `NativeList` 등을 써야 함.
- **Blittable Type:** `int`, `float`, `bool`, `struct` 등 메모리 레이아웃이 고정된 타입만 사용 가능함.
- **참조 금지:** Job 내부에서 `class` 객체는 절대 접근 불가함.

### [BurstCompile] 효과
- C#의 오버헤드를 제거하고 하드웨어(CPU)에 최적화된 기계어로 직접 변환함.
- 반복문 자동 벡터화(SIMD) 등을 통해 연산 속도가 기하급수적으로 빨라짐.

---

## 5. 실전 예제 코드 (System + Job)

### 데이터 정의 (Component)

```csharp
using Unity.Entities;
using Unity.Mathematics;

public struct MoveSpeed : IComponentData
{
    public float Value;
}

public struct TargetPos : IComponentData
{
    public float3 Value;
}
```

### 로직 처리 (System + Job)

```csharp
using Unity.Burst;
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;

[BurstCompile]
public partial struct MovementSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float deltaTime = SystemAPI.Time.DeltaTime;

        var job = new MoveJob
        {
            DeltaTime = deltaTime
        };

        job.ScheduleParallel();
    }
}

[BurstCompile]
public partial struct MoveJob : IJobEntity
{
    public float DeltaTime;

    public void Execute(ref LocalTransform transform, in MoveSpeed speed, in TargetPos target)
    {
        float3 dir = math.normalize(target.Value - transform.Position);
        transform.Position += dir * speed.Value * DeltaTime;
    }
}
```

---

## 6. Baking (GameObject → Entity)

에디터의 편리함과 ECS의 성능을 연결하는 다리 역할임.

```csharp
public class MyAuthoring : MonoBehaviour
{
    public float speed;
    public Vector3 target;

    class Baker : Baker<MyAuthoring>
    {
        public override void Bake(MyAuthoring authoring)
        {
            Entity entity = GetEntity(TransformUsageFlags.Dynamic);

            AddComponent(entity, new MoveSpeed { Value = authoring.speed });
            AddComponent(entity, new TargetPos { Value = authoring.target });
        }
    }
}
```

---

## 7. 주요 SystemAPI 활용법

- **SystemAPI.Query:** `foreach` 문으로 특정 컴포넌트들을 빠르게 순회함.
- **SystemAPI.GetSingleton:** 월드에 딱 하나만 있는 데이터(글로벌 설정 등)를 즉시 가져옴.
- **RefRW / RefRO:** 데이터를 참조 형태로 가져와서 성능 이득을 봄.
  - `RefRW<T>`: 읽고 쓰기 가능.
  - `RefRO<T>`: 읽기 전용 (성능 최적화에 유리).

---

## 8. Collider Trigger System

```csharp
using Unity.Burst;
using Unity.Entities;
using Unity.Physics;
using UnityEngine;

public partial struct ItemTriggerSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        state.Dependency = new ItemTriggerJob().Schedule(SystemAPI.GetSingleton<SimulationSingleton>(), state.Dependency);
    }
}

[BurstCompile]
struct ItemTriggerJob : ITriggerEventsJob
{
    public void Execute(TriggerEvent triggerEvent)
    {
        Debug.Log($"Hit: {triggerEvent.EntityA} & {triggerEvent.EntityB}");
    }
}
```

---

## 9. 예제 코드 — ECS Grab (VR)

### ECSGrabHandler.cs

```csharp
using Unity.Entities;
using Unity.Physics;
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.XR.Interaction.Toolkit.Interactors;

public class ECSGrabHandler : MonoBehaviour
{
    public NearFarInteractor Interactor;

    [Header("Input Settings")]
    public InputActionReference pinchActionReference;
    [Range(0, 1)] public float pinchThreshold = 0.5f;

    private Entity _handColliderEntity;
    private Entity _grabbedEntity = Entity.Null;
    private EntityManager _entityManager;

    void Start()
    {
        _entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
    }

    void Update()
    {
        if (!UpdateHandColliderEntity()) return;

        bool isPinching = CheckIsPinching();
        HandleGrabLogic(isPinching);
    }

    private bool CheckIsPinching()
    {
        if (pinchActionReference != null && pinchActionReference.action != null)
            return pinchActionReference.action.ReadValue<float>() >= pinchThreshold;

        return false;
    }

    private void HandleGrabLogic(bool isPinching)
    {
        if (isPinching)
        {
            if (_grabbedEntity == Entity.Null)
            {
                HandData handData = _entityManager.GetComponentData<HandData>(_handColliderEntity);

                if (handData.TouchingTarget != Entity.Null)
                {
                    _grabbedEntity = handData.TouchingTarget;

                    if (!_entityManager.HasComponent<GrabbedData>(_grabbedEntity))
                        _entityManager.AddComponentData(_grabbedEntity, new GrabbedData());
                }
            }
            else
            {
                _entityManager.SetComponentData(_grabbedEntity, new GrabbedData
                {
                    TargetPosition = transform.position,
                    TargetRotation = transform.rotation
                });
            }
        }
        else
        {
            if (_grabbedEntity != Entity.Null)
                ReleaseGrabbedEntity();
        }
    }

    private void ReleaseGrabbedEntity()
    {
        if (_entityManager.HasComponent<GrabbedData>(_grabbedEntity))
            _entityManager.RemoveComponent<GrabbedData>(_grabbedEntity);

        if (_entityManager.HasComponent<PhysicsDamping>(_grabbedEntity))
            _entityManager.SetComponentData(_grabbedEntity, new PhysicsDamping { Linear = 0f, Angular = 0f });

        _grabbedEntity = Entity.Null;
    }

    private bool UpdateHandColliderEntity()
    {
        if (_handColliderEntity != Entity.Null)
        {
            _entityManager.SetComponentData(_handColliderEntity, new Unity.Transforms.LocalTransform
            {
                Position = transform.position,
                Rotation = transform.rotation,
                Scale = 0.1f
            });

            return true;
        }

        EntityQuery query = _entityManager.CreateEntityQuery(typeof(HandData));

        if (query.HasSingleton<HandData>())
        {
            _handColliderEntity = query.GetSingletonEntity();
            return true;
        }
        return false;
    }
}
```

### GrabMoveSystem.cs

```csharp
using Unity.Burst;
using Unity.Entities;
using Unity.Physics;
using Unity.Physics.Systems;
using Unity.Transforms;

[UpdateInGroup(typeof(BeforePhysicsSystemGroup))]
public partial struct GrabMoveSystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // 이전 프레임의 접촉 정보 초기화
        foreach (RefRW<HandData> hand in SystemAPI.Query<RefRW<HandData>>())
            hand.ValueRW.TouchingTarget = Entity.Null;

        // Trigger Event 처리 잡 스케줄링
        state.Dependency = new ItemTriggerJob
        {
            HandLookup = SystemAPI.GetComponentLookup<HandData>(false)
        }.Schedule(SystemAPI.GetSingleton<SimulationSingleton>(), state.Dependency);

        // Grabbed -> Follow Target
        foreach (var (transform, grabbed, velocity, damping, entity) in
                 SystemAPI.Query<RefRW<LocalTransform>, RefRO<GrabbedData>, RefRW<PhysicsVelocity>, RefRW<PhysicsDamping>>()
                 .WithEntityAccess())
        {
            transform.ValueRW.Position = grabbed.ValueRO.TargetPosition;
            transform.ValueRW.Rotation = grabbed.ValueRO.TargetRotation;

            velocity.ValueRW.Linear = 0;
            velocity.ValueRW.Angular = 0;

            damping.ValueRW.Linear = 100f;
            damping.ValueRW.Angular = 100f;
        }
    }
}

[BurstCompile]
struct ItemTriggerJob : ITriggerEventsJob
{
    public ComponentLookup<HandData> HandLookup;

    public void Execute(TriggerEvent triggerEvent)
    {
        Entity entityA = triggerEvent.EntityA;
        Entity entityB = triggerEvent.EntityB;

        if (HandLookup.HasComponent(entityA))
        {
            var hand = HandLookup.GetRefRW(entityA);
            hand.ValueRW.TouchingTarget = entityB;
        }
        else if (HandLookup.HasComponent(entityB))
        {
            var hand = HandLookup.GetRefRW(entityB);
            hand.ValueRW.TouchingTarget = entityA;
        }
    }
}
```

### 컴포넌트 데이터 정의

```csharp
using Unity.Entities;
using Unity.Mathematics;
using UnityEngine;

public struct GrabbedData : IComponentData
{
    public float3 TargetPosition;
    public quaternion TargetRotation;
}

public struct HandData : IComponentData
{
    public Entity TouchingTarget;
}

public struct ItemSpawnerComponent : IComponentData
{
    public Entity Prefab;
}

public class SubSceneAuthoring : MonoBehaviour
{
    public GameObject DominoPrefab;

    class Baker : Baker<SubSceneAuthoring>
    {
        public override void Bake(SubSceneAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.None);
            AddComponent(entity, new ItemSpawnerComponent
            {
                Prefab = GetEntity(authoring.DominoPrefab, TransformUsageFlags.Dynamic)
            });
        }
    }
}
```

### HandColliderAuthoring.cs

```csharp
using Unity.Entities;
using UnityEngine;

public class HandColliderAuthoring : MonoBehaviour
{
    public class HandColliderBaker : Baker<HandColliderAuthoring>
    {
        public override void Bake(HandColliderAuthoring authoring)
        {
            Entity entity = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent(entity, new HandData());
        }
    }
}
```

---

[← 목차](./README.md)
