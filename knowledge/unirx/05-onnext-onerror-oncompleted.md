# 5. OnNext, OnError, OnCompleted

UniRx에서 발행되는 메시지는 모두 이 3가지 중 어느 하나입니다.

- **OnNext:** 통상 이벤트가 발행되었을 때 통지되는 메시지
- **OnError:** 스트림 처리 중 예외가 발생했음을 통지한다.
- **OnCompleted:** 스트림이 종료되었음을 통지한다.

---

## OnNext 메시지

OnNext는 UniRx에서 가장 많이 사용되는 메시지이며, 보통 "이벤트 통지"를 나타냅니다.

### "의미"없는 값을 통지 — Unit

```csharp
var subject = new Subject<Unit>();

subject.Subscribe(x => Debug.Log(x));

// Unit 형은 그 자체는 별 의미가 없다.
// 메시지의 내용에 의미가 아니라 이벤트 알림 타이밍이 중요한 순간에 사용할 수 있다.
subject.OnNext(Unit.Default);

// 실행 결과
// ()
```

예를 들어, "씬의 초기화 완료", "플레이어 사망" 등에서 사용할 수 있습니다.

---

## OnError 메시지

OnError 메시지는 예외가 스트림 도중에 발생했을 때 통지되는 메시지입니다.  
**OnError 메시지가 Subscribe까지 도달한 경우, 그 스트림 구독은 종료되고 파기됩니다.**

### 도중에 발생한 예외를 Subscribe로 받기

```csharp
var stringSubject = new Subject<string>();

stringSubject
    .Select(str => int.Parse(str))
    .Subscribe(
        x => Debug.Log("성공:" + x),       // OnNext
        ex => Debug.Log("예외가 발생:" + ex) // OnError
    );

stringSubject.OnNext("1");
stringSubject.OnNext("2");
stringSubject.OnNext("Hello"); // 이 메시지에서 예외가 발생한다.
stringSubject.OnNext("4");
stringSubject.OnCompleted();

// 실행 결과
// 성공 : 1
// 성공 : 2
// 예외가 발생 : System.FormatException : Input string was not in the correct format
```

OnError를 받은 후 `OnNext("4")`는 처리되지 않습니다.

### 도중에 예외가 발생하면 다시 구독하기

```csharp
var stringSubject = new Subject<string>();

stringSubject
    .Select(str => int.Parse(str))
    .OnErrorRetry((FormatException ex) =>
    {
        Debug.Log("예외가 발생하여 다시 구독 합니다");
    })
    .Subscribe(
        x => Debug.Log("성공:" + x),
        ex => Debug.Log("예외가 발생:" + ex)
    );

stringSubject.OnNext("1");
stringSubject.OnNext("2");
stringSubject.OnNext("Hello");
stringSubject.OnNext("4");
stringSubject.OnNext("5");
stringSubject.OnCompleted();

// 실행 결과
// 성공 : 1
// 성공 : 2
// 예외가 발생하여 다시 구독합니다
// 성공 : 4
// 성공 : 5
```

### OnError 관련 오퍼레이터

| 하고 싶은 일 | 오퍼레이터 |
|-------------|-----------|
| OnError가 오면 다시 Subscribe 하고 싶다. | `Retry` |
| OnError를 받아 에러 처리를 하고 다른 스트림으로 전환한다. | `Catch` |
| OnError를 받아 에러 처리를 한 후, OnCompleted로 대체하고 싶다. | `CatchIgnore` |
| OnError가 오면 에러 처리를 한 후, Subscribe를 다시 하고 싶다. (시간 지정 가능) | `OnErrorRetry` |

---

## OnCompleted 메시지

OnCompleted는 **"스트림이 완료되었기 때문에 이후 메시지를 발행하지 않겠다"**는 것을 통지하는 메시지입니다.  
OnCompleted가 Subscribe까지 도달하면 그 스트림의 구독은 종료되고 파기됩니다.  
한번 OnCompleted를 발행한 Subject는 재이용이 불가능합니다.

```csharp
var subject = new Subject<int>();
subject.Subscribe(
    x => Debug.Log(x),
    () => Debug.Log("OnCompleted")
);
subject.OnNext(1);
subject.OnNext(2);
subject.OnCompleted();

// 실행 결과
// 1
// 2
// OnCompleted
```

---

[← 목차](./README.md)
