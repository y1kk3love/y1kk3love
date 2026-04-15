# 5장 — 메서드와 매개변수

## Using

- `using`은 namespace 안에서도 선언 가능하다.
- `using static`으로 선언하면 형식 이름 생략 가능. 단, 다른 메서드나 클래스와 이름이 같아지면 모호함이 발생하므로 주의.

```csharp
using static System.Console;

class Main
{
    void Function()
    {
        WriteLine("Hello, World!");
    }
}
```

- **별칭(aliasing)** 을 사용해 다른 이름으로 호출할 수 있다.

```csharp
using CountDownTimer = System.Timers.Timer;
```

---

## 참조 전달

데이터를 참조로 전달한다.

**ref** — 양방향 참조 전달
```csharp
Swap(ref first, ref second);

void Swap(ref string x, ref string y)
{
    string temp = x;
    x = y;
    y = temp;
}
```

**out** — 출력 전용 참조 전달
```csharp
Swap(out number);

void Swap(out int number)
{
    number = 5;
}
```

**in** — 읽기 전용 참조 전달, 데이터를 변환하려고 하면 컴파일 오류 출력
```csharp
Swap(in number);
```

---

## 오버로딩

```csharp
void Function(int count) { ... }
void Function(int count, float life) { ... }
void Function(int count, string name) { ... }
```

---

## 선택적 매개변수

```csharp
void Function(int count = 0) { ... }
```

---

[← 목차](./README.md)
