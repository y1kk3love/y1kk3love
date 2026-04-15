# 5. 아키텍처 및 시스템 설계 (MVC/MVP)

---

## 관심사의 분리 (SoC)

**Separation of Concerns** — 데이터, 로직, 출력을 독립적인 레이어로 분리한다.

- 수정 범위를 줄인다 (View 변경이 Model에 영향 X).
- 테스트와 재사용이 쉬워진다.
- 팀 병렬 작업 시 코드 충돌을 최소화한다.

---

## MVC 패턴

| 역할 | 담당 | 예시 |
|------|------|------|
| **Model** | 데이터·상태 | `PlayerData`, `InventoryData` |
| **View** | 화면 출력 | `PlayerHUD`, `ItemSlotUI` |
| **Controller** | 로직·중재 | `PlayerController`, `GameManager` |

```csharp
// Model — 순수 데이터
public class PlayerModel
{
    public int HP { get; private set; }
    public int MaxHP { get; private set; }

    public PlayerModel(int maxHp) { HP = MaxHP = maxHp; }

    public void TakeDamage(int amount) => HP = Mathf.Max(0, HP - amount);
}

// View — UI만 담당, 데이터 없음
public class PlayerHUD : MonoBehaviour
{
    [SerializeField] private Slider _hpSlider;

    public void Refresh(int hp, int maxHp)
    {
        _hpSlider.value = (float)hp / maxHp;
    }
}

// Controller — Model을 조작하고 View를 갱신
public class PlayerController : MonoBehaviour
{
    private PlayerModel _model;
    private PlayerHUD _hud;

    void Start()
    {
        _model = new PlayerModel(100);
        _hud = FindObjectOfType<PlayerHUD>();
        _hud.Refresh(_model.HP, _model.MaxHP);
    }

    public void OnHit(int damage)
    {
        _model.TakeDamage(damage);
        _hud.Refresh(_model.HP, _model.MaxHP);
    }
}
```

**적용 영역:** 게임플레이 콘텐츠 전반, 씬 단위 시스템.

---

## MVP 패턴

MVC에서 Controller 대신 **Presenter**가 View와 Model을 완전히 분리.  
View는 Presenter에 이벤트를 전달하고, Presenter만 Model을 알고 View를 갱신한다.

```csharp
// View — Presenter에 이벤트만 전달, 로직 없음
public class SettingsView : MonoBehaviour
{
    public event Action<float> OnVolumeChanged;

    [SerializeField] private Slider _volumeSlider;

    void Start()
    {
        _volumeSlider.onValueChanged.AddListener(v => OnVolumeChanged?.Invoke(v));
    }

    public void SetVolume(float value) => _volumeSlider.value = value;
}

// Presenter — 모든 로직 담당
public class SettingsPresenter
{
    private readonly SettingsModel _model;
    private readonly SettingsView _view;

    public SettingsPresenter(SettingsModel model, SettingsView view)
    {
        _model = model;
        _view = view;

        _view.OnVolumeChanged += OnVolumeChanged;
        _view.SetVolume(_model.Volume); // 초기값 표시
    }

    private void OnVolumeChanged(float value)
    {
        _model.Volume = value;
        AudioManager.Instance.SetVolume(value);
    }
}
```

**적용 영역:** UI 시스템, 설정 화면, 팝업 등 복잡한 UI 로직.

---

## 계층적 아키텍처 — 싱글톤 남용 피하기

싱글톤을 남용하면 의존 관계가 복잡해지고 테스트가 어려워진다.

```csharp
// 나쁜 예: 어디서나 접근하는 싱글톤
GameManager.Instance.PlayerData.HP -= 10;
UIManager.Instance.HUD.Refresh();
AudioManager.Instance.PlaySFX("hit");

// 좋은 예: 상위 시스템이 하위 컨트롤러 인스턴스를 관리
public class GameplaySystem : MonoBehaviour
{
    [SerializeField] private PlayerController _player;
    [SerializeField] private UIController _ui;
    [SerializeField] private AudioController _audio;

    public void OnPlayerHit(int damage)
    {
        _player.TakeDamage(damage);
        _ui.RefreshHP(_player.HP);
        _audio.PlayHitSFX();
    }
}
```

---

## 협업 전략

- **기능 단위 모듈화:** View/Model/Presenter를 기능 폴더 단위로 분리 → 병렬 작업 시 충돌 최소화.
- **기술 부채 관리:** 대규모 기능 완료 후 리팩토링 세션을 주기적으로 진행.
- **인터페이스 계약:** 팀원 간 의존하는 부분은 인터페이스로 명세하여 구현과 분리.

---

[← 목차](./README.md)
