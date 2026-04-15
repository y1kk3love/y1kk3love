# 8장 — 인터페이스

- 인터페이스는 하나의 값과 구현이 없는 형식
- 한정자를 선언할 수 없으며 기본으로 public

```csharp
public interface IFruitsStore
{
    string? FruitValue { get; }
}

public class Apple : Fruit, IFruitsStore
{
    // 암시적 멤버 구현
    public string? FruitValue
    {
        get => Name;
    }
}
```

---

## 암시적 멤버 구현 vs 명시적 멤버 구현

```csharp
public interface IAction
{
    void Act();
    void React();
}

public class Person : IAction
{
    public void Act() { ... }       // 암시적: 클래스 레퍼런스로 호출 가능

    void IAction.React() { ... }    // 명시적: 인터페이스 레퍼런스로만 호출 가능
}

public class App
{
    public void Run()
    {
        Person p = new Person();
        p.Act();           // OK
        // p.React();      // 컴파일 에러

        IAction itf = p;
        itf.Act();         // OK
        itf.React();       // OK
        ((IAction)p).React(); // OK
    }
}
```

---

## 인터페이스 상속 / 다중 구현

```csharp
public interface IFruitsIsland
{
    string FruitValue(string name);
}

public interface IFruitsStore : IFruitsIsland
{
    void SliceFruit(int count);
}

// 다중 인터페이스 구현
public class Apple : Fruit, IFruitsStore, IFruitsIsland
{
    public string FruitValue(string name) { ... }
    public void SliceFruit(int count) { ... }
}
```

---

## C# 8.0 이상 — 기본 인터페이스 멤버

C# 8.0 이전에는 인터페이스에 새 멤버를 추가하면 기존 구현 타입에 모두 오류가 발생했다.  
C# 8.0부터는 새로운 멤버에 기본 구현 Body를 추가할 수 있다.

```csharp
// ILogger v1.0
public interface ILogger
{
    void Log(string message);
}

// ILogger v2.0 — 기존 구현체 수정 없이 확장 가능
public interface ILogger
{
    void Log(string message);

    // 기본 구현 추가
    void Log(Exception ex) => Log(ex.Message);
    void Log(string logType, string msg)
    {
        if (logType == "Error" || logType == "Warning" || logType == "Info")
            Log($"{logType}: {msg}");
        else
            throw new ApplicationException("Invalid LogType");
    }
}
```

> 기본 인터페이스 멤버에 접근하려면 인터페이스 타입으로 캐스팅해야 한다.

---

## 인터페이스 vs 추상 클래스

| 추상 클래스 | 인터페이스 |
|-------------|-----------|
| 비추상 파생 클래스만 인스턴스 생성 가능 | 구현 형식만 인스턴스 생성 가능 |
| 파생 클래스는 모든 추상 멤버를 구현해야 함 | 구현 형식은 모든 추상 인터페이스 멤버를 구현해야 함 |
| 비추상 멤버를 추가해도 호환성 유지 | C# 8.0부터 기본 인터페이스 멤버로 가능 |
| 메서드, 속성, 필드, 생성자 등 모두 선언 가능 | 인스턴스 멤버는 메서드, 속성만. 필드, 생성자, 종료자 불가 |
| 멤버를 가상/비가상으로 선언 가능 | 모든 (비봉인) 멤버는 가상 |
| 파생 클래스는 하나의 기본 클래스만 상속 | 구현 형식은 여러 인터페이스를 구현 가능 |

---

[← 목차](./README.md)
