# 6. 스트림의 구독 종료 (Dispose)

```csharp
public interface IObservable<T>
{
    IDisposable Subscribe(IObserver<T> observer);
}
```

Subscribe의 반환값이 `IDisposable`이므로, `Dispose()`를 실행하면 스트림의 구독을 종료할 수 있습니다.

---

## Dispose() — 스트림의 구독 종료

```csharp
var subject = new Subject<int>();

// IDisposable 저장
var disposable = subject.Subscribe(x => Debug.Log(x), () => Debug.Log("OnCompleted"));

subject.OnNext(1);
subject.OnNext(2);

// 구독 종료
disposable.Dispose();

subject.OnNext(3);
subject.OnCompleted();

// 실행 결과
// 1
// 2
```

Dispose를 호출하여 구독을 언제라도 중단할 수 있습니다.  
주의: `Dispose()`를 실행해서 구독이 중단되어도 `OnCompleted`가 발행되는 것은 아닙니다.

---

## 특정 스트림만 수신 거부

```csharp
var subject = new Subject<int>();

var disposable1 = subject.Subscribe(x =>
        Debug.Log("스트림1:" + x), () => Debug.Log("OnCompleted"));
var disposable2 = subject.Subscribe(x =>
        Debug.Log("스트림2:" + x), () => Debug.Log("OnCompleted"));

subject.OnNext(1);
subject.OnNext(2);

// 스트림1만 구독 종료
disposable1.Dispose();

subject.OnNext(3);
subject.OnCompleted();

// 실행 결과
// 스트림1 : 1
// 스트림2 : 1
// 스트림1 : 2
// 스트림2 : 2
// 스트림2 : 3
// 2 : OnCompleted
```

OnCompleted를 실행하면 모든 스트림을 구독 종료하게 되지만, Dispose를 사용하면 일부 스트림만 종료시킬 수 있습니다.

---

[← 목차](./README.md)
