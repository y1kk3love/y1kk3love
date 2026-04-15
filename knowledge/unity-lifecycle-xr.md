# 3. Unity 엔진 라이프사이클 & XR 인터랙션

---

## 주요 업데이트 루프

| 메서드 | 호출 타이밍 | 주요 용도 |
|--------|-------------|-----------|
| `Update` | 매 프레임 (프레임레이트 가변) | 입력 처리, 렌더링 관련 로직 |
| `FixedUpdate` | 고정 시간 간격 (기본 0.02s) | 물리 연산, 결정론적 시뮬레이션 |
| `LateUpdate` | 모든 Update 완료 후 | 카메라 추적, 캐릭터 최종 위치 조정 |

---

## 각 메서드 상세

### Update

```csharp
void Update()
{
    // 입력 처리
    if (Input.GetKeyDown(KeyCode.Space))
        Jump();

    // 렌더링과 연관된 오브젝트 이동
    transform.position += _velocity * Time.deltaTime;
}
```

- `Time.deltaTime`을 곱해야 프레임레이트와 무관한 이동 속도를 보장한다.
- 물리 연산을 여기서 하면 기기 성능에 따라 결과가 달라진다.

### FixedUpdate

```csharp
void FixedUpdate()
{
    // PhysX와 동기화된 힘 적용
    _rb.AddForce(Vector3.up * _jumpForce, ForceMode.Impulse);
}
```

- 물리 엔진(PhysX)은 `FixedUpdate` 주기로 스텝을 밟는다.
- `Time.fixedDeltaTime`을 사용한다 (`Time.deltaTime` X).
- 프레임레이트와 무관하게 **결정론적** 결과를 보장 → 멀티플레이어/시뮬레이션에 필수.

### LateUpdate

```csharp
void LateUpdate()
{
    // 모든 캐릭터 Update가 끝난 뒤 카메라 추적
    _camera.position = _target.position + _offset;
    _camera.LookAt(_target);
}
```

- 카메라를 `Update`에서 처리하면 캐릭터 이동과 순서 충돌로 **지터링(Jittering)** 발생.
- 핸드 트래킹 데이터를 `LateUpdate`에서 적용하면 시각적 흔들림 없음.

---

## 전체 실행 순서

```
Awake → OnEnable → Start
  ↓
FixedUpdate (물리 스텝)
  ↓
Update
  ↓
LateUpdate
  ↓
OnRenderObject
  ↓
(다음 프레임)
```

---

## XR 인터랙션 적용 사례

### 물리 안정성 — FixedUpdate

```csharp
// 수술 도구 충돌, Boids 군집 → FixedUpdate에서 처리
void FixedUpdate()
{
    foreach (var boid in _boids)
    {
        boid.velocity += ComputeFlocking(boid) * Time.fixedDeltaTime;
        boid.transform.position += boid.velocity * Time.fixedDeltaTime;
    }
}
```

기기 성능(Quest 2 vs Quest 3)에 관계없이 **동일한 시뮬레이션 결과** 보장.

### 트래킹 안정성 — LateUpdate

```csharp
// 핸드 트래킹 결과를 마지막에 반영 → Jittering 방지
void LateUpdate()
{
    if (_handTrackingData.IsTracked)
    {
        transform.position = _handTrackingData.Position;
        transform.rotation = _handTrackingData.Rotation;
    }
}
```

---

[← 목차](./README.md)
