# Photon Fusion 네트워크 동기화

Photon Fusion은 Unity 멀티플레이어를 위한 서버 권위 기반 네트워킹 솔루션. Rollback 또는 State Authority 모델로 지연을 처리한다.

---

## 1. Shared Mode vs Host Mode

| | Shared Mode | Host Mode |
|--|-------------|-----------|
| 서버 권위 | 클라이언트 분산 소유 | 호스트가 최종 권위 |
| 구현 난이도 | 낮음 | 높음 |
| 적합한 게임 | 협동, 소규모 | 경쟁, 치팅 방지 필요 |
| State Authority | 오브젝트별 분산 | 호스트 중앙 집중 |

```csharp
// Host Mode 연결 예시
public async void StartGame()
{
    var runner = gameObject.AddComponent<NetworkRunner>();
    runner.ProvideInput = true;

    await runner.StartGame(new StartGameArgs
    {
        GameMode = GameMode.Host,   // 또는 GameMode.Client
        SessionName = "MyRoom",
        Scene = SceneRef.FromIndex(1),
        SceneManager = gameObject.AddComponent<NetworkSceneManagerDefault>()
    });
}
```

---

## 2. NetworkObject & NetworkBehaviour

네트워크 동기화가 필요한 오브젝트에는 반드시 `NetworkObject` 컴포넌트를 추가한다.

```csharp
// NetworkBehaviour 상속 — 동기화 로직 작성
public class PlayerController : NetworkBehaviour
{
    // [Networked] 프로퍼티 → 변경 시 자동 동기화
    [Networked] public int HP { get; set; }
    [Networked] public Vector3 Position { get; set; }

    // 자신의 입력인지 확인
    public override void FixedUpdateNetwork()
    {
        if (GetInput(out NetworkInputData data))
        {
            // 입력 처리 (서버 + 클라이언트 모두 실행 = Prediction)
            Vector3 move = data.direction * Runner.DeltaTime * Speed;
            transform.position += move;
            Position = transform.position;
        }
    }
}
```

---

## 3. 입력 처리 구조

```csharp
// 입력 데이터 구조체 정의
public struct NetworkInputData : INetworkInput
{
    public Vector3 direction;
    public NetworkButtons buttons;  // 버튼 상태 (비트 플래그)
}

// INetworkRunnerCallbacks 구현체에서 입력 제공
public void OnInput(NetworkRunner runner, NetworkInput input)
{
    var data = new NetworkInputData();
    data.direction = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical"));

    if (Input.GetButton("Fire1"))
        data.buttons.Set(MyButtons.Fire, true);

    input.Set(data);
}
```

---

## 4. [Networked] 변경 감지

```csharp
public class HealthDisplay : NetworkBehaviour
{
    [Networked(OnChanged = nameof(OnHPChanged))]
    public int HP { get; set; }

    // HP가 변경될 때 자동 호출 (모든 클라이언트)
    private static void OnHPChanged(Changed<HealthDisplay> changed)
    {
        changed.Behaviour.UpdateHPBar(changed.Behaviour.HP);
    }

    private void UpdateHPBar(int hp)
    {
        _hpSlider.value = hp / (float)MaxHP;
    }
}
```

---

## 5. RPC — 이벤트성 통신

동기화 데이터가 아니라 "이벤트"를 특정 클라이언트에게 보낼 때.

```csharp
// RPC 선언
[Rpc(RpcSources.StateAuthority, RpcTargets.All)]
public void RPC_PlayHitEffect(Vector3 hitPosition)
{
    Instantiate(_hitEffectPrefab, hitPosition, Quaternion.identity);
}

// 호출 (State Authority인 호스트에서)
void TakeDamage(int damage, Vector3 position)
{
    HP -= damage;
    RPC_PlayHitEffect(position);  // 모든 클라이언트에서 이펙트 재생
}
```

| RpcSources | RpcTargets | 용도 |
|-----------|-----------|------|
| StateAuthority | All | 서버 → 모든 클라이언트 브로드캐스트 |
| InputAuthority | StateAuthority | 클라이언트 → 서버 요청 |
| All | All | 모든 peer에서 모든 peer로 |

---

## 6. 지연 보상 (Lag Compensation)

```csharp
// 히트스캔 무기: 클라이언트가 쏜 시점의 과거 위치로 판정
public void FireRaycast()
{
    if (!HasInputAuthority) return;

    // Runner.LagCompensation → 클라이언트 RTT 기준 과거 씬 상태 사용
    var hitOptions = HitOptions.IncludePhysX | HitOptions.SubtickAccuracy;

    if (Runner.LagCompensation.Raycast(
        transform.position,
        transform.forward,
        100f,
        Object.InputAuthority,
        out var hit,
        ~0,
        hitOptions))
    {
        hit.Hitbox?.Root.GetComponent<HealthDisplay>()?.TakeDamage(10, hit.Point);
    }
}
```

---

## 7. 주의사항

- `FixedUpdateNetwork`는 서버와 클라이언트 모두에서 실행된다. 예측(Prediction)과 재조정(Reconciliation)이 자동 처리된다.
- `[Networked]` 프로퍼티는 `FixedUpdateNetwork` 또는 `Spawned`에서만 수정해야 한다. `Update`에서 수정하면 동기화가 틀어진다.
- `NetworkObject`가 없는 오브젝트는 네트워크 동기화 불가. 씬에 배치된 오브젝트도 `NetworkObject`가 필요하면 사전 등록(In-Scene Placed Network Object)해야 한다.

---

[← 목차](./README.md)
