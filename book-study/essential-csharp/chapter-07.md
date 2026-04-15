# 7장 — 상속

## 상속 정의

- **파생 / 상속**: 추가 멤버나 기본 클래스 멤버의 사용자 지정을 포함하도록 기본 클래스를 구체화한다.
- **파생 형식 / 자식 형식**: 더 일반적인 형식의 멤버를 상속하는 구체화된 형식.
- **기본 형식 / 부모 형식**: 파생 형식이 상속하는 멤버를 소유한 일반 형식.

---

## 파생

- 대체 기본 클래스를 지정하지 않는 한 모든 클래스는 기본적으로 `Object`에서 파생된다.

```csharp
public class PdaItem { }
public class Contact : PdaItem { }

public class Program
{
    public static void Main()
    {
        // 파생 형식은 암시적으로 기본 형식으로 변환될 수 있음
        Contact contact = new Contact();
        PdaItem item = contact;

        // 기본 형식은 명시적으로 파생 형식으로 캐스팅 되어야 함
        contact = (Contact)item;
    }
}
```

**implicit / explicit 변환 연산자**
```csharp
public readonly struct Digit
{
    private readonly byte digit;

    public static implicit operator byte(Digit d) => d.digit;
    public static explicit operator Digit(byte b) => new Digit(b);
}
```

---

## 단일 상속 / 봉인 클래스

- 한 클래스가 2개의 클래스를 직접 상속할 수 없다.
- **봉인 클래스** — 파생할 수 없다.

```csharp
public sealed class Apple : Fruit { ... }
```

---

## 기본 클래스 재정의

**virtual / override**
```csharp
public class Fruit
{
    public virtual string Name { get; set; }
}

public class Apple : Fruit
{
    public override string Name
    {
        get => $"{FirstName} {LastName}";
        set
        {
            string[] names = value.Split(' ');
            FirstName = names[0];
            LastName = names[1];
        }
    }

    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

런타임이 가상 메서드를 만날 때마다 최하위 파생과 가상 멤버를 재정의한 구현을 호출한다.

**new 한정자** — 기본 클래스 멤버를 숨기고 새로운 멤버를 선언한다.
```csharp
public class BaseClass
{
    public void DisplayName() { Console.WriteLine("BaseClass"); }
}

public class DerivedClass : BaseClass
{
    public virtual void DisplayName() { Console.WriteLine("DerivedClass"); }
}

public class SuperDerivedClass : SubDerivedClass
{
    public new void DisplayName() { Console.WriteLine("SuperSubDerivedClass"); }
}
```

**sealed 한정자** — 해당 멤버 이상 재정의 불가
```csharp
class B : A
{
    public override sealed void Method() { ... }
}
```

**base 멤버** — 상속받은 멤버에 접근
```csharp
class Apple : Fruit
{
    public override string ToString() { return base.ToString(); }
}
```

**기본 클래스 생성자 호출**
```csharp
class Apple : Fruit
{
    public Apple(string name) : base(name) { }
}
```

---

## 추상 클래스

추상 멤버는 재정의돼야 하므로 자동으로 가상이다.

```csharp
public abstract class Fruit
{
    public abstract string Name();
}

public class Apple : Fruit
{
    public override string Name() => "Apple";
}

// 추상 클래스는 인스턴스 생성 불가
// Fruit fruit = new Fruit(); // 오류
```

---

## System.Object에서 파생된 모든 클래스

| 메서드 | 설명 |
|--------|------|
| `Equals(object o)` | 같은 값이면 true 반환 |
| `GetHashCode()` | 해시 코드 반환 |
| `GetType()` | 인스턴스 형식 반환 |
| `ReferenceEquals(a, b)` | 동일한 개체이면 true |
| `ToString()` | 문자열 표현 반환 |
| `Finalize()` | 소멸자 별칭 |
| `MemberwiseClone()` | 얕은 복사 |

---

## 패턴 매칭

**타입 패턴 매칭**
```csharp
if (GetObjectById(id) is Employee employee)
{
    Display(employee);
}
```

**상수 패턴 매칭**
```csharp
public static void Save(object data)
{
    if (data is "") { return; }
}
```

**var 패턴 매칭** — null 포함 모든 값
```csharp
if (GetObjectById(id) is var result) { ... }
```

**튜플 패턴 매칭**
```csharp
if ((args.Length, args[0]) is (1, "show"))
{
    Console.WriteLine(File.ReadAllText(DataFile));
}
```

**속성 패턴 매칭**
```csharp
if (person is { FirstName: string firstName, LastName: string lastName })
{
    Console.WriteLine($"{firstName} {lastName}");
}
```

**as 연산자** — 변환 실패 시 null 반환. 참조형만 확인 가능.

---

[← 목차](./README.md)
