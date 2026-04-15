# UniTask

UniTask는 async와 await의 기존 C#에서 사용하는 비동기 작업을 사용하기 편리하게 만들어주는 라이브러리입니다.

---

## Zero Allocation

UniTask는 Zero Allocation 기능을 제공하여 메모리 할당을 최소화합니다.  
불필요한 가비지 컬렉션(GC)을 방지하여 프레임 드랍이나 렉을 줄일 수 있습니다.  
코루틴은 메모리 할당을 필요로 하며, 이는 GC 오버헤드를 초래할 수 있습니다.

```csharp
// Coroutine
private void Start() { StartCoroutine(MyCoroutine()); }

private IEnumerator MyCoroutine()
{
    while (true)
    {
        yield return new WaitForSeconds(1f);  // 메모리 할당 발생
    }
}
```

```csharp
// UniTask
private async void Start() { await MyUniTask(); }

private async UniTask MyUniTask()
{
    while (true)
    {
        await UniTask.Delay(1000);  // Zero Allocation
    }
}
```

---

## WhenAll

**WhenAll**은 여러 비동기 작업이 모두 완료될 때까지 기다리는데 사용되는 메서드입니다.

```csharp
// Coroutine — 모든 작업이 완료될 때까지 기다리기 위해 기다리는 코드를 추가해줘야 합니다.
private IEnumerator WaitForAll()
{
    Coroutine task1 = StartCoroutine(Task(2));  // 2초 대기
    Coroutine task2 = StartCoroutine(Task(3));  // 3초 대기

    yield return task1;
    yield return task2;
}
```

```csharp
// UniTask
private async void Start()
{
    await UniTask.WhenAll(Task1(), Task2());
}

private async UniTask Task1()
{
    await UniTask.Delay(2000);  // 2초 대기
}

private async UniTask Task2()
{
    await UniTask.Delay(3000);  // 3초 대기
}
```

---

## Coroutine vs UniTask 비교

| 기능 | Coroutine | UniTask |
|------|-----------|---------|
| Delay | `yield return new WaitForSeconds(3f)` | `await UniTask.Delay(TimeSpan.FromSeconds(3f))` |
| WaitUntil | `yield return new WaitUntil(() => count == 7)` | `await UniTask.WaitUntil(() => count == 7)` |
| CancellationToken | — | `await UniTask.Delay(TimeSpan.FromSeconds(3), cancellationToken: cancel.Token)` |
| Frame 대기 | `yield return null` / `yield return new WaitForEndOfFrame()` | `await UniTask.Yield()` / `await UniTask.NextFrame()` / `await UniTask.WaitForEndOfFrame(this)` |

---

## Task 중단 — CancellationTokenSource

```csharp
while (true)
{
    if (cancellationToken.IsCancellationRequested)
    {
        break;
    }

    await UniTask.Delay(100, ignoreTimeScale: true, cancellationToken: cancellationToken);
}
```

---

## 오브젝트가 파괴될 때 UniTask 취소

```csharp
var token = this.GetCancellationTokenOnDestroy();
await UniTask.Delay(1000, cancellationToken: token);
```

`GetCancellationTokenOnDestroy`는 UniTask 라이브러리의 일부로 제공되며, GameObject나 Component가 파괴될 때 발생하는 취소 토큰을 가져옵니다.

---

## Update문과 같은 역할을 하는 UniTask

```csharp
private async UniTaskVoid UpdateUniTask()
{
    while (true)
    {
        await UniTask.Yield(PlayerLoopTiming.Update);
    }
}
```

`PlayerLoopTiming`에는 `Update` 외에 `PreUpdate`, `LateUpdate` 등의 열거형 키워드가 존재합니다.

---

## Tuple 값 받기

```csharp
var (google, bing, yahoo) = await UniTask.WhenAll(task1, task2, task3);
```

---

## Timeout Handling

```csharp
var cancel = new CancellationTokenSource();
cancel.CancelAfterSlim(TimeSpan.FromSeconds(5));

try
{
    await UnityWebRequest.Get("http://...").SendWebRequest().WithCancellation(cancel.Token);
}
catch (OperationCanceledException ex)
{
    if (ex.CancellationToken == cancel.Token)
    {
        Debug.Log("Timeout");
    }
}
```

---

## Error Handling

```csharp
private async UniTaskVoid Start()
{
    try
    {
        await UniTask.Run(() => ThrowException());
    }
    catch (System.Exception e)
    {
        Debug.LogError(e.Message);
    }
}

private void ThrowException()
{
    throw new System.Exception("An error occurred!");
}
```

---

## 별도의 스레드에서 비동기적으로 실행

```csharp
private async UniTaskVoid Start()
{
    int result = await UniTask.Run(() => Compute());
}

private int Compute()
{
    int sum = 0;
    for (int i = 0; i < 1000000; i++)
    {
        sum += i;
    }
    return sum;
}
```

`UniTask.Run`을 이용해서 동기식 코드를 스레드풀에서 비동기식으로 실행시킬 수 있습니다.

---

## UniTask와 UniTaskVoid 차이

두 타입의 주요 차이는 **"반환 값의 유무"**와 관련이 있습니다.

- **UniTaskVoid**: 호출한 후 다음 줄로 넘어간다.
- **UniTask**: 비동기 작업을 호출하고 비동기 작업이 끝날 때까지 기다린다.

---

[← 목차](./README.md)
