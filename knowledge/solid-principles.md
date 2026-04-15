# SOLID 원칙

객체지향 설계의 5가지 핵심 원칙. 유지보수 가능하고 확장하기 쉬운 코드를 만들기 위한 기준.

---

## 1. SRP — 단일 책임 원칙 (Single Responsibility Principle)

> 클래스는 변경 이유가 하나뿐이어야 한다.

```csharp
// 나쁜 예: 하나의 클래스가 데이터 + 저장 + UI를 모두 담당
public class Player
{
    public int HP;
    public void SaveToFile() { /* 저장 로직 */ }
    public void UpdateHPUI() { /* UI 갱신 */ }
}

// 좋은 예: 책임 분리
public class PlayerData { public int HP; }
public class PlayerSaver { public void Save(PlayerData data) { } }
public class PlayerHUD { public void Refresh(int hp) { } }
```

---

## 2. OCP — 개방/폐쇄 원칙 (Open/Closed Principle)

> 확장에는 열려 있고, 수정에는 닫혀 있어야 한다.

```csharp
// 나쁜 예: 새 무기 추가 시 Attack()을 매번 수정해야 함
public class AttackHandler
{
    public void Attack(string weaponType)
    {
        if (weaponType == "Sword") { /* 검 공격 */ }
        else if (weaponType == "Gun") { /* 총 공격 */ }
        // 추가될 때마다 이 메서드 수정 필요
    }
}

// 좋은 예: 인터페이스로 확장
public interface IWeapon { void Attack(); }

public class Sword : IWeapon { public void Attack() { /* 검 */ } }
public class Gun : IWeapon { public void Attack() { /* 총 */ } }

public class AttackHandler
{
    public void Attack(IWeapon weapon) => weapon.Attack(); // 수정 불필요
}
```

---

## 3. LSP — 리스코프 치환 원칙 (Liskov Substitution Principle)

> 자식 클래스는 부모 클래스를 대체할 수 있어야 한다.

```csharp
// 나쁜 예: Bird를 기대하는 곳에 Penguin을 넣으면 예외 발생
public class Bird { public virtual void Fly() { } }
public class Penguin : Bird
{
    public override void Fly() => throw new NotSupportedException("펭귄은 못 납니다");
}

// 좋은 예: 계층 구조 재설계
public abstract class Bird { }
public interface IFlyable { void Fly(); }

public class Eagle : Bird, IFlyable { public void Fly() { } }
public class Penguin : Bird { /* Fly 없음 */ }
```

---

## 4. ISP — 인터페이스 분리 원칙 (Interface Segregation Principle)

> 사용하지 않는 메서드에 의존하지 않도록 인터페이스를 분리해야 한다.

```csharp
// 나쁜 예: 모든 기능을 하나의 인터페이스에
public interface IEnemy
{
    void Move();
    void Attack();
    void Fly();    // 날지 않는 적도 구현해야 함
    void Swim();   // 수영 안 하는 적도 구현해야 함
}

// 좋은 예: 기능별 분리
public interface IMovable { void Move(); }
public interface IAttackable { void Attack(); }
public interface IFlyable { void Fly(); }

public class GroundEnemy : IMovable, IAttackable { /* Fly 불필요 */ }
public class FlyingEnemy : IMovable, IAttackable, IFlyable { }
```

---

## 5. DIP — 의존 역전 원칙 (Dependency Inversion Principle)

> 고수준 모듈은 저수준 모듈에 의존하면 안 된다. 둘 다 추상화에 의존해야 한다.

```csharp
// 나쁜 예: 구체 클래스에 직접 의존
public class GameManager
{
    private MySQLDatabase _db = new MySQLDatabase(); // 구체 클래스
    public void Save() => _db.Save();
}

// 좋은 예: 추상화(인터페이스)에 의존
public interface IDatabase { void Save(); }
public class MySQLDatabase : IDatabase { public void Save() { } }
public class LocalDatabase : IDatabase { public void Save() { } }

public class GameManager
{
    private readonly IDatabase _db;
    public GameManager(IDatabase db) { _db = db; } // 외부에서 주입
    public void Save() => _db.Save();
}
```

---

## 6. Unity에서의 SOLID 적용

| 원칙 | Unity 적용 사례 |
|------|----------------|
| SRP | MonoBehaviour는 로직 진입점만. 데이터는 Model, UI는 View로 분리. |
| OCP | 새 아이템/적 타입 추가 시 기존 코드 수정 없이 ScriptableObject + 인터페이스로 확장. |
| LSP | 컴포넌트 인터페이스(`IDamageable`, `IInteractable`) 기반 설계. |
| ISP | `IPickable`, `IUsable`, `IDroppable` 기능별 소규모 인터페이스. |
| DIP | `FindObjectOfType` 대신 인스펙터 직접 주입 또는 DI 컨테이너 활용. |

---

[← 목차](./README.md)
