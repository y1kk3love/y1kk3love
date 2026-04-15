# 디자인 패턴

반복되는 설계 문제에 대한 검증된 해결 방식. Unity 개발에서 자주 활용되는 패턴 중심으로 정리.

---

## 1. Observer (옵저버)

상태 변화를 여러 객체에 자동으로 알리는 패턴.  
Unity의 `event`, `UnityEvent`, `UniRx` 모두 이 패턴의 구현이다.

```csharp
// 발행자 (Subject)
public class PlayerHP
{
    public event Action<int> OnHPChanged;
    private int _hp;

    public int HP
    {
        get => _hp;
        set
        {
            _hp = value;
            OnHPChanged?.Invoke(_hp); // 구독자에게 알림
        }
    }
}

// 구독자들
public class HPBar : MonoBehaviour
{
    [SerializeField] private Slider _slider;
    public void Subscribe(PlayerHP hp) => hp.OnHPChanged += v => _slider.value = v;
}

public class HPLogger : MonoBehaviour
{
    public void Subscribe(PlayerHP hp) => hp.OnHPChanged += v => Debug.Log($"HP: {v}");
}
```

**적합한 상황:** HP 변화 → UI 갱신, 게임 이벤트 → 여러 시스템 반응

---

## 2. Command (커맨드)

요청을 객체로 캡슐화하여 실행/취소/재실행을 가능하게 하는 패턴.

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class MoveCommand : ICommand
{
    private Transform _target;
    private Vector3 _delta;
    private Vector3 _prevPos;

    public MoveCommand(Transform target, Vector3 delta)
    {
        _target = target;
        _delta = delta;
    }

    public void Execute()
    {
        _prevPos = _target.position;
        _target.position += _delta;
    }

    public void Undo() => _target.position = _prevPos;
}

// 커맨드 관리자
public class CommandHistory
{
    private Stack<ICommand> _history = new Stack<ICommand>();

    public void Execute(ICommand cmd) { cmd.Execute(); _history.Push(cmd); }
    public void Undo() { if (_history.Count > 0) _history.Pop().Undo(); }
}
```

**적합한 상황:** Ctrl+Z 실행 취소, 에디터 툴, 리플레이 시스템

---

## 3. State (스테이트)

객체의 상태에 따라 행동이 달라지는 패턴. `if/else` 또는 `switch` 문의 복잡도를 줄인다.

```csharp
public interface IPlayerState
{
    void Enter();
    void Update();
    void Exit();
}

public class IdleState : IPlayerState
{
    public void Enter() => Debug.Log("Idle 진입");
    public void Update() { /* 대기 애니메이션 */ }
    public void Exit() => Debug.Log("Idle 종료");
}

public class RunState : IPlayerState
{
    public void Enter() => Debug.Log("달리기 시작");
    public void Update() { /* 달리기 처리 */ }
    public void Exit() { }
}

public class PlayerFSM : MonoBehaviour
{
    private IPlayerState _currentState;

    public void ChangeState(IPlayerState newState)
    {
        _currentState?.Exit();
        _currentState = newState;
        _currentState.Enter();
    }

    void Update() => _currentState?.Update();
}
```

**적합한 상황:** 캐릭터 상태 (Idle/Run/Jump/Attack), UI 흐름 관리

---

## 4. Factory (팩토리)

객체 생성 로직을 별도 클래스에 위임하여 결합도를 낮추는 패턴.

```csharp
public abstract class Enemy : MonoBehaviour
{
    public abstract void Init();
}

public class Goblin : Enemy { public override void Init() => Debug.Log("고블린 초기화"); }
public class Dragon : Enemy { public override void Init() => Debug.Log("드래곤 초기화"); }

public class EnemyFactory
{
    [SerializeField] private Goblin _goblinPrefab;
    [SerializeField] private Dragon _dragonPrefab;

    public Enemy Create(string type, Vector3 pos) => type switch
    {
        "Goblin" => Object.Instantiate(_goblinPrefab, pos, Quaternion.identity),
        "Dragon" => Object.Instantiate(_dragonPrefab, pos, Quaternion.identity),
        _ => throw new ArgumentException($"알 수 없는 타입: {type}")
    };
}
```

**적합한 상황:** 아이템/적 생성, Object Pooling 결합, 런타임 타입 결정

---

## 5. Strategy (전략)

알고리즘을 인터페이스로 분리하여 런타임에 교체 가능하게 하는 패턴.

```csharp
public interface IAttackStrategy { void Attack(Transform target); }

public class MeleeAttack : IAttackStrategy
{
    public void Attack(Transform target) => Debug.Log($"근접 공격: {target.name}");
}

public class RangedAttack : IAttackStrategy
{
    public void Attack(Transform target) => Debug.Log($"원거리 공격: {target.name}");
}

public class Enemy : MonoBehaviour
{
    private IAttackStrategy _strategy;

    public void SetStrategy(IAttackStrategy strategy) => _strategy = strategy;
    public void PerformAttack(Transform target) => _strategy?.Attack(target);
}
```

**적합한 상황:** 난이도별 AI 행동 교체, 무기별 공격 방식, 정렬/탐색 알고리즘 교체

---

## 6. Singleton (싱글턴)

인스턴스가 하나뿐임을 보장하는 패턴. 편리하지만 남용하면 결합도가 높아진다.

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this) { Destroy(gameObject); return; }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

> 씬 전환 시 상태를 유지해야 하는 시스템(AudioManager, SceneLoader 등)에만 제한적으로 사용.  
> 무분별한 싱글턴은 테스트와 모듈화를 어렵게 만든다.

---

## 7. 패턴 선택 기준

| 상황 | 추천 패턴 |
|------|----------|
| "이 사건을 여러 곳에 알려야 한다" | Observer |
| "조작 기록을 남겨 되돌리고 싶다" | Command |
| "상태에 따라 행동이 완전히 달라진다" | State |
| "어떤 타입인지 모르는 객체를 만들어야 한다" | Factory |
| "알고리즘을 런타임에 바꾸고 싶다" | Strategy |
| "어디서나 접근해야 하는 단 하나의 객체" | Singleton (최소한으로) |

---

[← 목차](./README.md)
