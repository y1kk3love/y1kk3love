# 10장 — 잘 구현된 형식

## Object 재정의

`ToString()`, `GetHashCode()`, `Equals()`의 Object 클래스 함수를 재정의하는 방법을 다룬다.  
필요에 따라 재정의해주면 된다.

---

## 연산자 재정의

연산자를 재정의해서 사용하는 방법. 직접 만든 구조체나 클래스를 연산할 때 유용하다.

→ [연산자 재정의 공식 문서](https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/operator-overloading)

---

## XML 주석

```csharp
class FunctionAddNumbers
{
    /// <summary>
    /// 두 수를 더하여 그 결괏값을 반환시켜 주는 함수
    /// </summary>
    /// <param name="a">첫 번째 매개변수</param>
    /// <param name="b">두 번째 매개변수</param>
    /// <returns>a + b 결과</returns>
    static int AddNumbers(int a, int b) => a + b;
}
```

---

## 가비지 수집

가비지 컬렉트는 더 이상 참조하지 않는 메모리를 복원한다. 생성 비용보다 유지 비용이 더 큰 참조 개체용으로 설계된 것이다.

**강한 참조** — 일반적인 `new` 할당. 참조 객체에 null을 할당해도 다른 객체에서 참조하고 있으면 GC가 수집하지 않는다.

**약한 참조** — `WeakReference`를 이용. 참조 객체에 null을 할당하면 다른 객체에서 참조하고 있더라도 GC가 수집한다.

```csharp
object phone = new Stock("Phone");
object notebook = new Stock("NoteBook");

object stock1 = phone;                          // 강한 참조
WeakReference stock2 = new WeakReference(notebook); // 약한 참조

phone = null;
notebook = null;

// GC 수행 후
// stock1 → "Phone" (강한 참조이므로 유지)
// stock2 → null (약한 참조이므로 GC가 수집)
```

---

## 종료자

- 매개변수도 없고, 한정자도 사용하지 않는다.
- 오버로딩 불가, 직접 호출 불가.
- CLR의 가비지 컬렉터가 객체가 소멸되는 시점을 판단해서 종료자를 호출한다.
- 명시적으로 종료자를 구현하면 GC가 Finalize() 메서드를 클래스 족보를 타고 올라가며 호출하기 때문에 성능 저하가 발생할 수 있다. **가급적 구현하지 않는 것이 좋다.**

```csharp
~Cat()  // 소멸자
{
    Console.WriteLine($"{Name} : 잘가");
}
```

---

## using 구문을 사용한 명확한 종료

`using` 문은 `IDisposable`을 구현한 객체를 올바르게 사용할 수 있도록 한다.  
File, Font와 같이 관리되지 않는 리소스에 접근하는 클래스들은 사용을 마친 후 반드시 해제해야 한다.

```csharp
using (SqlConnection connection = new SqlConnection(connectionString))
{
    SqlCommand command = new SqlCommand(queryString, connection);
    command.Connection.Open();
    command.ExecuteNonQuery();
} // 블록을 벗어나면 자동으로 Dispose() 호출
```

---

## IDisposable 패턴

```csharp
public class MyResource : IDisposable
{
    private IntPtr Handle;
    private bool disposed = false;

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (disposed) return;

        if (disposing)
        {
            // 관리 리소스 해제
        }

        // 비관리 리소스 해제
        CloseHandle(Handle);
        Handle = IntPtr.Zero;
        disposed = true;
    }

    ~MyResource() => Dispose(disposing: false);
}
```

---

## 초기화 지연 (Lazy Initialization)

사용자가 실제로 필요할 때만 로딩하여 데이터 낭비를 줄이는 방법.  
개체를 처음 사용할 때까지 생성을 지연시킨다.

→ [공식 문서](https://learn.microsoft.com/ko-kr/dotnet/framework/performance/lazy-initialization)

---

[← 목차](./README.md)
