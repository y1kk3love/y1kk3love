# Coroutine vs Task

---

## 1. Coroutine

Unity에서 제공하는 비동기 처리 방식.  
`IEnumerator`를 반환하는 메서드와 `yield return` 문법으로 실행을 분할한다.  
Unity의 **생명 주기(Update 루프)** 위에서 동작하므로, 메인 스레드에서만 실행된다.

```csharp
private IEnumerator TimerCoroutine()
{
    Debug.Log("시작");
    yield return new WaitForSeconds(2f);  // 2초 대기 (메인 스레드 비점유)
    Debug.Log("2초 후");
}

private void Start()
{
    StartCoroutine(TimerCoroutine());
}
```

### 특징

- `MonoBehaviour`가 있어야 사용 가능하다.
- `GameObject`가 비활성화되거나 파괴되면 자동으로 중단된다.
- `WaitForSeconds`, `WaitUntil`, `WaitForEndOfFrame` 등 Unity 내장 대기 클래스를 사용할 수 있다.
- 메인 스레드에서만 동작하므로 Unity API를 자유롭게 호출할 수 있다.
- `yield return new WaitForSeconds()` 호출 시 메모리 할당이 발생해 GC 부하가 생긴다.

---

## 2. Task (async/await)

C# 표준 비동기 처리 방식.  
`async`/`await` 키워드로 비동기 흐름을 동기 코드처럼 작성할 수 있다.  
Unity 생명 주기와 **무관하게** 동작하며, 별도 스레드에서도 실행 가능하다.

```csharp
private async void Start()
{
    Debug.Log("시작");
    await Task.Delay(2000);  // 2초 대기
    Debug.Log("2초 후");
}
```

### 특징

- `MonoBehaviour` 없이도 사용 가능하다.
- `GameObject`가 파괴되어도 Task는 계속 실행된다. (직접 취소 처리 필요)
- `CancellationToken`으로 명시적으로 취소를 관리해야 한다.
- 멀티스레드 실행이 가능하지만, Unity API는 **메인 스레드에서만** 호출해야 한다.
- 힙 메모리 할당이 발생해 GC 부하가 있다.

```csharp
private async void Start()
{
    var cts = new CancellationTokenSource();

    try
    {
        await HeavyTask(cts.Token);
    }
    catch (OperationCanceledException)
    {
        Debug.Log("취소됨");
    }
}

private async Task HeavyTask(CancellationToken token)
{
    await Task.Run(() =>
    {
        // 별도 스레드에서 무거운 연산
        for (int i = 0; i < 1000000; i++)
        {
            token.ThrowIfCancellationRequested();
        }
    }, token);
}
```

---

## 3. 비교

| | Coroutine | Task (async/await) |
|--|-----------|-------------------|
| **동작 기반** | Unity Update 루프 | C# 스레드풀 / 메인 스레드 |
| **멀티스레드** | 불가 | 가능 |
| **Unity API 호출** | 자유롭게 가능 | 메인 스레드에서만 가능 |
| **MonoBehaviour 필요** | 필요 | 불필요 |
| **오브젝트 파괴 시** | 자동 중단 | 계속 실행됨 (수동 취소 필요) |
| **취소 방식** | `StopCoroutine()` | `CancellationToken` |
| **예외 처리** | 어려움 | `try/catch`로 처리 가능 |
| **메모리** | `WaitForSeconds` 등에서 GC 발생 | Task 생성 시 GC 발생 |
| **가독성** | `yield return` 중첩 시 복잡 | 선형 흐름으로 읽기 쉬움 |

---

## 4. 언제 무엇을 쓸까?

- **Coroutine** — Unity 생명 주기 타이밍(`WaitForEndOfFrame`, `WaitForFixedUpdate`)에 맞춰야 하거나, 간단한 딜레이/시퀀스 처리에 적합하다.
- **Task** — CPU 집약적인 작업을 백그라운드 스레드에서 처리하거나, 복잡한 비동기 흐름에 명확한 취소·예외 처리가 필요할 때 적합하다.
- **UniTask** — Unity 환경에서 `Task`의 편리함과 `Coroutine`의 Unity 친화성을 모두 원한다면 UniTask가 최선이다. GC 부하도 없다.

---

[← 목차](./README.md)
