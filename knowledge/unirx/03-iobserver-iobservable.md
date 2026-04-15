# 3. IObserver와 IObservable

Subject는 `IObserver` 인터페이스와 `IObservable` 인터페이스 2개를 구현하고 있습니다.

---

## IObserver 인터페이스

`IObserver`는 Rx에서 "이벤트 메시지를 발행할 수 있다"라는 행동을 정의한 인터페이스입니다.

```csharp
using System;

namespace UniRx
{
    public interface IObserver<T>
    {
        void OnCompleted();
        void OnError(Exception error);
        void OnNext(T value);
    }
}
```

`OnNext`, `OnError`, `OnCompleted`의 3개의 메서드 정의만 있는 간단한 인터페이스입니다.

- **OnError:** 발생한 오류(Exception)를 알리는 메시지를 발행하는 메서드.
- **OnCompleted:** 메시지의 발행이 완료되었음을 알리는 메서드.

---

## IObservable 인터페이스

`IObservable`은 Rx에서 "이벤트 메시지를 구독할 수 있다"라는 행동을 정의한 인터페이스입니다.

```csharp
namespace UniRx
{
    public interface IObservable<T>
    {
        IDisposable Subscribe(IObserver<T> observer);
    }
}
```

---

## Subscribe 생략 호출 예

```csharp
// OnNext만
subject.Subscribe(msg => Debug.Log("Subscribe1:" + msg));

// OnNext & OnError
subject.Subscribe(
    msg => Debug.Log("Subscribe1:" + msg),
    error => Debug.LogError("Error" + error));

// OnNext & OnCompleted
subject.Subscribe(
    msg => Debug.Log("Subscribe1:" + msg),
    () => Debug.Log("Completed"));

// OnNext & OnError & OnCompleted
subject.Subscribe(
    msg => Debug.Log("Subscribe1:" + msg),
    error => Debug.LogError("Error" + error),
    () => Debug.Log("Completed"));
```

---

[← 목차](./README.md)
