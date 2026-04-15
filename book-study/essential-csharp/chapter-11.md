# 11장 — 예외 처리

## 예외 형식

| 예외 | 설명 |
|------|------|
| ArgumentException | 메서드에 전달되는 null이 아닌 인수가 잘못되었다. |
| ArgumentNullException | 메서드에 전달되는 인수가 null이다. |
| ArgumentOutOfRangeException | 인수가 유효한 값 범위를 벗어났다. |
| DirectoryNotFoundException | 디렉터리 경로 일부가 잘못되었다. |
| DivideByZeroException | 0으로 나누었다. |
| Exception | 모든 종류의 예외를 처리할 수 있다. |
| FileNotFoundException | 파일이 없다. |
| FormatException | 문자열에서 변환할 수 있는 적절한 형식이 아니다. |
| IndexOutOfRangeException | 인덱스가 배열 또는 컬렉션의 범위를 벗어났다. |
| InvalidOperationException | 개체의 현재 상태에서 메서드 호출을 사용할 수 없다. |
| KeyNotFoundException | 컬렉션의 멤버에 액세스 하는 데 지정된 키를 찾을 수 없다. |
| NotImplementedException | 메서드 또는 작업이 구현되지 않았다. |
| NotSupportedException | 메서드 또는 작업이 지원되지 않는다. |
| NullReferenceException | null 개체를 역참조하는 시도가 있다. |
| OverflowException | 산술, 캐스팅 또는 변환 작업을 수행하면 오버플로가 발생한다. |
| TimeoutException | 작업에 할당된 시간 간격이 만료되었다. |

---

## 예외 잡기

```csharp
private string TryTest()
{
    try
    {
        Console.WriteLine("#try");
        return "return in Try";
    }
    catch (ArgumentException ex)
    {
        Console.WriteLine("#catch ArgumentException");
        return ex.Message;
    }
    catch (Exception ex)
    {
        Console.WriteLine("#catch");
        return ex.Message;
    }
    finally
    {
        Console.WriteLine("#finally");  // 항상 실행
    }
}
```

---

## 기존 예외 다시 던지기

원래 예외에서 스택 추적 정보를 잃지 않고 이전에 던져진 예외를 던질 수 있다.

```csharp
try
{
    while (!task.Wait(100))
    {
        Console.Write(".");
    }
}
catch (AggregateException exception)
{
    exception = exception.Flatten();
    ExceptionDispatchInfo.Capture(exception.InnerException ?? exception).Throw();
}
```

---

## 예외 처리를 위한 지침

- 처리할 수 있는 예외만 처리한다.
- 완전히 처리하지 못하는 예외를 숨기지 않는다.
- `System.Exception`과 일반적인 catch 블록은 될 수 있으면 사용하지 않는다.
- 호출 스택의 하위에서 예외 보고나 로그 기록을 피한다.
- catch 블록 내에서 `throw <예외 개체>` 대신 `throw;`를 사용한다.
- 예외 필터에서 예외를 던지지 않는다.

---

## 사용자 지정 예외 정의

```csharp
class DatabaseException : Exception
{
    public DatabaseException() { }

    public DatabaseException(string? message) : base(message) { }

    public DatabaseException(string? message, Exception? exception)
        : base(message, innerException: exception) { }
}
```

---

[← 목차](./README.md)
