# 9장 — 값 형식

## 값 형식 vs 참조 형식 요약

- 값 형식은 값을 직접 메모리에 할당해서 사용
- 참조 형식은 실제 값이 들어있는 힙 메모리의 주소를 사용
- 스택은 할당 가능 크기가 힙에 비해 매우 작다.
- 참조 형식은 주소를 복사하는 얕은 복사를 진행하기 때문에 힙 데이터 변형에 주의.
- 스택 풀을 정리하는 비용은 힙의 가비지 컬렉트 작업보다 리소스를 적게 소모하지만 메모리 오버플로우는 항상 주의.

---

## 구조체

사용자는 구조체를 통해 직접 값 형식 변수를 정의할 수 있다.

```csharp
public struct Coords
{
    public Coords(double x, double y)
    {
        X = x;
        Y = y;
    }

    public double X { get; }
    public double Y { get; }

    public override string ToString() => $"({X}, {Y})";
}
```

**구조체 초기화** — 생성자를 사용할 때는 모든 필드를 초기화해야 한다.

```csharp
struct Person
{
    public string name;
    public int age;
    public string address;

    public Person(string name, int age, string address)
    {
        this.name = name;
        this.age = age;
        this.address = address;
    }
}
```

**값 형식의 new** — `new` 키워드 사용 시 매개변수가 없는 생성자를 호출하고 모든 멤버를 기본 값으로 초기화한다.  
구조체에는 종료자를 지원하지 않아 리소스 해제 시점을 알기 어렵다.

**값 형식의 상속** — `Object → ValueType → Int32` 와 같은 상속 관계를 가진다. 값 형식도 인터페이스 구현이 가능하다.

---

## 박싱 ↔ 언박싱

- **박싱**: 값 타입의 객체를 참조 타입으로 변환. 스택에 저장된 데이터를 힙으로 복사한 후 스택 포인터를 힙으로 변경.
- **언박싱**: 박싱의 역방향 동작.

```csharp
int i = 123;           // 값 타입

object obj = i;        // 박싱 → 힙 영역에 123 저장
int j = (int)obj;      // 언박싱
```

**lock** — 멀티스레드에서 스레드 동기화에 사용. 여러 스레드가 동시에 리소스에 접근하는 것을 막는다.

---

## 열거형

가독성을 위해 사용한다. 개발자가 선언할 수 있는 값 형식이다.

```csharp
enum ConnectionState
{
    Disconnected,
    Connecting,
    Connected,
    Disconnecting
}
```

**열거형 ↔ 문자열 변환**
```csharp
// enum → string
string text = Alphabet.A.ToString();  // "A"

// string → enum
Alphabet a = (Alphabet)Enum.Parse(typeof(Alphabet), "A");

// 제네릭 응용
T StringToEnum<T>(string e) => (T)Enum.Parse(typeof(T), e);
```

**플래그로서 열거형** — 고유한 비트 값을 선언할 수 있다.
```csharp
[Flags]
public enum FileAttributes
{
    None        = 0,
    ReadOnly    = 1 << 0,   // 001
    Hidden      = 1 << 1,   // 010
    System      = 1 << 2,   // 100
    // ...
}
```

---

[← 목차](./README.md)
