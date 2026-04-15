# 6장 — 클래스

## 객체 지향 프로그래밍

**캡슐화** — 필드와 메서드 등을 하나로 묶는 과정. 코드 수정 등에 대한 사이드 이펙트를 줄여 유지 관리를 쉽게 하고, 접근 권한을 관리해 높은 보안성을 가진다.

**상속** — "식품 → 과일 → 사과"와 같은 계층 구조를 형성.
- 특정 속성, 함수의 내용을 다르게 동작하도록 재정의할 수 있다.
- 공통 기능을 부모에서 정의하여 사용할 수 있다.
- 파생(하위 형식) ⇒ 자식 / 기본(상위 형식) ⇒ 부모

**다형성** — 다양한 형태를 가진다. 자식인 사과를 선언해도 부모의 형으로 호출이 가능한 경우.

---

## 클래스 선언과 인스턴스 생성

- **클래스**: 인스턴스 생성 시 개체의 모습에 대한 템플릿.
- **개체**: 클래스의 인스턴스.
- **인스턴스화**: 클래스에서 개체를 만드는 과정 (`new` 키워드).

C#은 C++ 등과 달리 직접적인 메모리 할당 해제는 지원하지 않고 GC(garbage collector)가 자동 회수하기 때문에 메모리 관리에 주의가 필요하다.

```csharp
public Apple apple = new Apple();

class Apple
{
    public string Name;

    void Sell() { ... }

    // 멤버 내 참조 this
    void Sell(string Name)
    {
        Apple.Name = this.Name;
    }
}
```

---

## 파일 입출력

| 클래스 | 설명 |
|--------|------|
| File | 파일에 대한 생성, 복사, 삭제, 이동 및 열기를 위한 정적 메소드 제공 |
| FileInfo | 파일에 대한 생성, 복사, 삭제, 이동 및 열기를 위한 속성 및 인스턴스 메소드 제공 |
| FileStream | 파일에 대한 스트림을 제공하여 동기/비동기 읽기·쓰기 작업 지원 |
| StreamReader | 문자열에서 읽어오는 TextReader를 구현 |
| StreamWriter | TextWriter를 구현하여 특정 인코딩의 스트림에 문자를 씀 |

---

## 액세스 한정자

| 호출자의 위치 | public | protected internal | protected | internal | private protected | private |
|---------------|--------|--------------------|-----------|----------|-------------------|---------|
| 클래스 내 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| 파생 클래스 (동일 어셈블리) | ✔ | ✔ | ✔ | ✔ | ✔ | ✗ |
| 비파생 클래스 (동일 어셈블리) | ✔ | ✔ | ✗ | ✔ | ✗ | ✗ |
| 파생 클래스 (다른 어셈블리) | ✔ | ✔ | ✔ | ✗ | ✗ | ✗ |
| 비파생 클래스 (다른 어셈블리) | ✔ | ✗ | ✗ | ✗ | ✗ | ✗ |

---

## 속성 (Property)

```csharp
public int MP;

public int HP => _hp;
private int hp { get; set; }
private int exp
{
    get { return huntcount; }
    set { level = value; }
}
```

- **프로퍼티의 ref, out**: 프로퍼티에서는 ref나 out을 통한 참조 전달은 불가능하다. 전달하고 싶다면 값을 따로 복사해서 전달하면 가능하다.

---

## 생성자

```csharp
class Apple
{
    private int seed;

    public Apple(int count)
    {
        seed = count;
    }
}

public Apple apple = new Apple(5);
```

**new 연산자의 세부 구현**
1. 메모리 관리자에서 빈 메모리를 가져온 뒤 지정된 생성자 호출
2. 빈 메모리의 참조를 암시적인 `this` 매개변수로 전달
3. 생성자 체인을 실행하고 생성자들 사이의 참조를 전달
4. 생성자 체인의 실행이 끝나면 `new` 연산자는 해당 메모리의 참조를 반환

**컬렉션 이니셜라이저**
```csharp
List<Apple> AppleList = new List<Apple>()
{
    new Apple(1),
    new Apple(3)
};
```

**생성자 체인** — 오버로딩과 달리 중복 코드를 피할 수 있다.
```csharp
class Apple
{
    private int seed;
    private string place;

    public Apple(int count) : this(count, "") { }
    public Apple(string country) : this(0, country) { }

    public Apple(int count, string country) : this()
    {
        seed = count;
        place = country;
    }
}
```

**정적 생성자** — 클래스 자체를 초기화하는 수단. 명시적으로 호출할 수 없으며 매개변수도 없다. 멤버나 메서드에 접근하기 전에 호출된다.

---

## null 허용 특성

특성을 통해 컴파일러에 null 허용 의도에 관한 단서를 제공할 수 있다.

| 특성 | 설명 |
|------|------|
| AllowNull | null을 허용하지 않는 매개 변수, 필드 또는 속성은 null일 수 있다. |
| DisallowNull | null 허용 매개 변수, 필드 또는 속성은 null일 수 없다. |
| MaybeNull | null을 허용하지 않는 반환 값은 null일 수 있다. |
| NotNull | null 허용 반환 값은 null이 아니다. |
| MemberNotNull | 메서드가 반환하는 경우 나열된 멤버는 null이 아니다. |

```csharp
[AllowNull]
public string ScreenName
{
    get => _screenName;
    set => _screenName = value ?? GenerateRandomScreenName();
}
private string _screenName = GenerateRandomScreenName();
```

---

## 분해자

생성자가 여러 매개변수를 하나로 캡슐화한다면, 분해자는 그 반대.

```csharp
public string GoodName;

// 튜플
(string kind, string name) = fruit;

// '_'은 필드 무시
fruit.Deconstruct(out _, out GoodName);

class Fruit
{
    public string Kind;
    public string Name;

    public void Deconstruct(out string kind, out string name)
    {
        kind = Kind;
        name = Name;
    }
}
```

---

## readonly

`const`가 소스 코드에 할당된 값을 컴파일 시점에 확정하는 것과 달리 `readonly`는 런타임에서 아래 두 경우에만 값을 확정한다.
1. 변수를 선언할 때
2. 생성자 안에서 값을 변경할 때

---

## 부분 클래스, 부분 메서드

동일한 파일 내에서 둘 이상으로 클래스, 메서드를 분해하여야 할 때 사용한다.

```csharp
// PersonField.cs
partial class Person
{
    private string _name;
    private int _age;
}

// PersonProperty.cs
partial class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// PersonMethod.cs
partial class Person
{
    public void DisplayPersonInfo()
    {
        Console.WriteLine("Name: " + _name + " / Age: " + _age);
    }
}
```

---

[← 목차](./README.md)
