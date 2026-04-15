# 13장 — 대리자와 람다 식

## 대리자 (Delegate)

특정 매개 변수 목록 및 반환 형식이 있는 메서드에 대한 참조를 나타내는 형식.  
대리자를 인스턴스화하면 호환되는 시그니처와 반환 형식의 모든 메서드에 연결할 수 있다.

```csharp
// 선언
public delegate bool Callback(int data1, string data2);

static void Main(string[] args)
{
    // 초기화
    Callback callback = Method;

    // 호출
    if (callback(1, "1"))
    {
        Console.WriteLine("Good");
    }
}

// 참조 메서드
public static bool Method(int data1, string data2)
{
    return true;
}
```

---

## 람다 식 & 익명 메서드

람다 식은 코드 형식을 줄여 더 간결하게 만들어 주지만, 가독성이 낮아지는 단점이 있다.

```csharp
public delegate bool Callback(int data1, string data2);

static void Main(string[] args)
{
    Callback callback = Method;

    // 람다 식
    callback += (data1, data2) => { return true; };

    // 익명 메서드
    callback += delegate (int data1, string data2) { return true; };
}
```

---

## Action과 Func 대리자

`delegate`에 비해 코드를 더 간결하게 만든다.

- **Action** — 반환을 하지 않는 메서드를 캡슐화
- **Func** — 값을 하나 반환하는 메서드를 캡슐화. `Func<T,...,TResult>` 형태로 사용

```csharp
// 선언과 초기화
Action<int> action = (num) => { num = 0; };
Func<int, int> func = (num) => { return num; };

// 호출
action.Invoke(5);
int result = func.Invoke(10);
```

---

[← 목차](./README.md)
