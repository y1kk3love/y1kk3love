# Unity Profiler & 최적화 워크플로

Unity Profiler는 CPU/GPU/메모리/렌더링의 병목을 식별하는 도구. "측정 → 원인 파악 → 수정" 순서를 지키는 것이 핵심이다.

---

## 1. Profiler 기본 사용법

```
Window → Analysis → Profiler  (단축키: Ctrl+7)
```

| 탭 | 측정 대상 |
|----|----------|
| CPU Usage | 각 프레임에서 소요된 CPU 시간, 스레드별 분포 |
| GPU Usage | GPU 렌더링 시간 (Direct3D/Vulkan 등) |
| Memory | 힙 메모리, GC Alloc, 오브젝트별 사용량 |
| Rendering | Draw Calls, SetPass Calls, Triangles |
| Physics | 물리 스텝 소요 시간, 활성 Rigidbody 수 |

---

## 2. CPU 병목 파악

### 타임라인 뷰에서 프레임 분석

1. **Record** 버튼 → 게임 실행 → 중단
2. CPU Usage 그래프에서 스파이크 구간 클릭
3. 하단 Timeline / Hierarchy 뷰에서 가장 오래 걸린 항목 확인

```
BehaviourUpdate           12.4 ms  ← 스크립트 Update 총합
  PlayerController.Update  8.1 ms  ← 이 스크립트가 주범
  EnemyAI.Update            3.2 ms
```

### 자체 마커 추가 (Profiler.BeginSample)

```csharp
using UnityEngine.Profiling;

void Update()
{
    Profiler.BeginSample("PathFinding");
    RunAStar();
    Profiler.EndSample();

    Profiler.BeginSample("EnemyDecision");
    MakeDecision();
    Profiler.EndSample();
}
```

Timeline에서 직접 명명된 구간이 표시되어 어느 메서드가 문제인지 바로 파악 가능.

---

## 3. GC Alloc 추적 (메모리)

GC Alloc이 매 프레임 발생하면 GC 수거 시 프레임 드롭(히치)이 생긴다.

```
Memory 탭 → GC Allocated per Frame 확인
CPU 탭 → GC.Collect 호출 시점 = 히치 원인
```

### 흔한 GC Alloc 원인과 해결

```csharp
// 나쁜 예: 매 프레임 new 리스트 생성
void Update()
{
    var enemies = new List<Enemy>(); // GC Alloc!
    FindEnemies(enemies);
}

// 좋은 예: 재사용
private List<Enemy> _enemies = new List<Enemy>();
void Update()
{
    _enemies.Clear(); // 할당 없음
    FindEnemies(_enemies);
}

// 나쁜 예: 문자열 연결
void Update()
{
    Debug.Log("HP: " + _hp); // GC Alloc (string concat)
}

// 좋은 예
void Update()
{
    Debug.Log($"HP: {_hp}"); // 여전히 alloc 있음 → Release 빌드에서 Debug.Log 제거
}
```

---

## 4. Rendering 병목 파악

```
Rendering 탭 → Draw Calls / SetPass Calls / Batches
```

| 수치 | 의미 | Quest 2 권장 |
|------|------|-------------|
| Batches | 실제 드로우 콜 수 | 100 이하 |
| SetPass Calls | 머티리얼 전환 횟수 | Batches보다 적을수록 좋음 |
| Triangles | 렌더된 폴리곤 수 | 750k 이하 |

```
Frame Debugger (Window → Analysis → Frame Debugger)
→ 프레임을 드로우 콜 단위로 분해해서 각 단계를 직접 확인 가능
```

---

## 5. GPU 병목 확인

CPU 시간은 짧은데 GPU 시간이 길면 GPU 바운드.

```
Stats 창 (Game 뷰 우상단 Stats 버튼)
→ GPU time 수치 확인

또는 Platform별 GPU Profiler:
- Android/Quest: Snapdragon Profiler
- PC: RenderDoc, PIX (DirectX)
```

---

## 6. Physics 최적화

```
Physics 탭 → Physics.Simulate 시간 확인
```

- `Fixed Timestep` 낮출수록 정확하지만 비쌈 (`Project Settings → Time`)
- 불필요한 Rigidbody는 `Sleep` 상태 유지 (자동 처리되지만 Wake 조건 최소화)
- `Physics.OverlapSphere` 등 쿼리 결과를 캐싱하여 매 프레임 쿼리 방지

---

## 7. 최적화 워크플로 체크리스트

```
1. Profiler로 측정 (추측 금지)
2. 가장 비싼 구간 1개 선택
3. 원인 파악 (GC? Draw Call? 복잡한 로직?)
4. 수정
5. 재측정하여 개선 확인
6. 목표 수치(fps, ms) 달성까지 반복
```

> 측정 없는 최적화는 시간 낭비다. 항상 Profiler를 먼저 켠다.

---

[← 목차](./README.md)
