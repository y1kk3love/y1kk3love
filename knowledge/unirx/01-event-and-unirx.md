# 1. event와 UniRx

UniRX의 이벤트 시스템을 이해하기 위해 기존의 C# Event로 만들어진 코드를 UniRX의 이벤트 시스템으로 변환하여 설명합니다.

## 기존 C# Event

```csharp
// 이벤트 핸들러 (이벤트 메시지의 형식 정의)
public delegate void TimerEventHandler(int time);

// 이벤트
public event TimerEventHandler OnTimeChanged;

private void Start()
{
    StartCoroutine(TimerCoroutine());
}

private IEnumerator TimerCoroutine()
{
    var time = 100;
    while (time > 0)
    {
        time--;

        // 이벤트 알림
        OnTimeChanged(time);

        yield return new WaitForSeconds(1);
    }
}
```

```csharp
private TimeCounter timeCounter;
private Text counterText;

private void Start()
{
    timeCounter.OnTimeChanged += time =>
    {
        counterText.text = time.ToString();
    };
}
```

---

## UniRx로 변환

```csharp
// 이벤트를 발행하는 핵심 인스턴스
private Subject<int> timerSubject = new Subject<int>();

// 이벤트의 구독자만 공개
public IObservable<int> OnTimeChanged => timerSubject;

private void Start() { StartCoroutine(TimerCoroutine()); }

private IEnumerator TimerCoroutine()
{
    var time = 100;
    while (time > 0)
    {
        time--;

        timerSubject.OnNext(time);

        yield return new WaitForSeconds(1);
    }
}
```

```csharp
private TimeCounterRx timeCounter;
private Text counterText;

private void Start()
{
    timeCounter.OnTimeChanged.Subscribe(time =>
    {
        counterText.text = time.ToString();
    });
}
```

---

event 대신 `Subject`라는 클래스가 이벤트 핸들러를 등록할 때 `Subscribe` 하는 방법이 등장하고 있습니다.  
즉 UniRx는 Subject가 이벤트 구조의 핵심이 되고, Subject에 값을 전달(`OnNext`)하고 Subject에 구독(`Subscribe`)해서 메시지를 전달할 수 있는 시스템으로 되어 있습니다.

---

[← 목차](./README.md)
