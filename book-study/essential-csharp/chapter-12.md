# 12장 — 제네릭

## 개요

제네릭(generic)은 알고리즘 등의 재사용을 촉진하기 위해 C#에서 제공하는 기능. 자바의 제네릭 형식이나 C++의 템플릿과 유사하다.

데이터를 일반화하여 코드를 작성할 수 있게 만들어 주기 때문에 재사용성이 높고 유연한 코드를 만들 수 있다.  
결과적으로 비제네릭 코드에 비해 코드의 양을 줄이고, 안정성을 높이며, 캐스팅이나 박싱 등의 비용을 줄인다.

---

## 제네릭 없는 C#의 문제

```csharp
Stack stack = new Stack();

stack.Push(1);
stack.Push("Hello Generic");
stack.Push(new MyClass());

// 서로 다른 타입이 들어갔을 때 예외 상황 발생 가능
// 값 형식에 대한 박싱 작업 → 메모리 할당과 GC까지 사용해야 해서 비용이 커질 수 있음
```

---

## 제네릭 사용

```csharp
Stack<string> stack = new Stack<string>();

stack.Push("Hello Generic");    // 지정된 형식만 허용
Console.WriteLine(stack.Pop()); // 캐스팅 불필요
```

---

## 단순 제네릭 클래스 정의

`T`를 통해 모든 곳에 대처하는 단일 형식 인수를 제공한다.

```csharp
public class CustomStack<T>
{
    private T[] InternalItems { get; }

    public void Push(T data) { ... }
    public T Pop() { ... }
}
```

---

## 제네릭 인터페이스와 구조체

```csharp
public interface IPair<T>
{
    public T First { get; set; }
    public T Second { get; set; }
}

public struct Pair<T>
{
    public T First;
    public T Second;
}
```

---

## 생성자와 종료자 정의

```csharp
public class MyClass<T> : IPair<T>
{
    public MyClass(T first, T second)
    {
        First = first;
        Second = second;
    }

    public T First { get; set; }
    public T Second { get; set; }
}
```

---

## 중첩 제네릭 형식

```csharp
public class MyClass<T, U>
{
    public class Inner<V>
    {
        void Method(T param1, U param2, V param3) { ... }
    }
}
```

---

## 형식 제약 조건 where

| 제약 조건 | 설명 |
|-----------|------|
| `where T : struct` | T는 null을 허용하지 않는 값 형식이어야 한다. |
| `where T : class` | T는 참조 형식이어야 한다. |
| `where T : new()` | T는 매개 변수가 없는 public 생성자가 있어야 한다. |
| `where T : notnull` | T는 null이 아닌 형식이어야 한다. |
| `where T : unmanaged` | T는 null이 아닌 비관리형 형식이어야 한다. |
| `where T : <기본 클래스>` | T는 지정된 기본 클래스이거나 이 클래스에서 파생된 클래스여야 한다. |
| `where T : <인터페이스>` | T는 지정된 인터페이스를 구현해야 한다. |
| `where T : U` | T는 U에서 상속되어야 한다. |

---

[← 목차](./README.md)
