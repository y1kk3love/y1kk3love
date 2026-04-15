# 8. 스트림 소스를 만드는 방법

---

## Subject 시리즈

`Subject`는 몇 가지 파생이 존재하고 각각 다른 행동을 취합니다.

| Subject | 기능 |
|---------|------|
| `Subject<T>` | OnNext가 실행되면 값을 발행한다. |
| `BehaviourSubject<T>` | 마지막으로 발행된 값을 캐시하고 나중에 Subscribe될 때 그 캐시를 반환한다. 초기값 설정 가능. |
| `ReplaySubject<T>` | 과거 모든 발행된 값을 캐시하고 나중에 Subscribe될 때 캐시를 모두 발행한다. |
| `AsyncSubject<T>` | OnNext를 즉시 발행하지 않고 내부에 캐시하고 OnCompleted가 실행된 시점에 마지막 OnNext 하나만 발행한다. Future/Promise와 유사. |

---

## ReactiveProperty 시리즈

### ReactiveProperty

`ReactiveProperty<T>`는 변수에 `Subject`의 기능을 붙인 것입니다.

```csharp
var rp = new ReactiveProperty<int>(10); // 초기값 지정 가능

// 일반적으로 대입하거나 값을 읽을 수 있다.
rp.Value = 20;
var currentValue = rp.Value;

// Subscribe 할 수 있다. (Subscribe시 현재 값도 발행된다)
rp.Subscribe(x => Debug.Log(x));

// 값을 다시 설정할 때 OnNext 발행된다.
rp.Value = 30;

// 실행 결과
// 20
// 30
```

인스펙터 뷰에 표시하려면 전용 타입을 사용합니다.

```csharp
[SerializeField]
private IntReactiveProperty playerHealth = new IntReactiveProperty(100);

private void Start() => playerHealth.Subscribe(x => Debug.Log(x));
```

### ReactiveCollection

`ReactiveCollection<T>`는 상태의 변화를 알리는 기능이 내장된 `List<T>`입니다.

이벤트:
- 요소 추가
- 요소의 제거
- 요소 수의 변화
- 요소 재정의
- 요소의 이동
- 목록 지우기

```csharp
var collection = new ReactiveCollection<string>();

collection
    .ObserveAdd()
    .Subscribe(x => Debug.Log($"Add [{x.Index}] = {x.Value}"));

collection
    .ObserveRemove()
    .Subscribe(x => Debug.Log($"Remove [{x.Index}] = {x.Value}"));

collection.Add("Apple");
collection.Add("Baseball");
collection.Add("Cherry");
collection.Remove("Apple");

// 실행 결과
// Add [0] = Apple
// Add [1] = Baseball
// Add [2] = Cherry
// Remove [0] = Apple
```

### ReactiveDictionary

`ReactiveDictionary<T1, T2>`는 ReactiveCollection의 Dictionary 버전입니다.

---

## 팩토리 메서드 시리즈

### Observable.Create

자유롭게 값을 발행하는 스트림을 만들 수 있는 팩토리 메서드입니다.

```csharp
Observable.Create<int>(observer =>
{
    Debug.Log("Start");

    for (int i = 0; i <= 100; i += 10)
    {
        observer.OnNext(i);
    }

    Debug.Log("Finished");
    observer.OnCompleted();

    return Disposable.Create(() =>
    {
        Debug.Log("Dispose");
    });
}).Subscribe(x => Debug.Log(x));

// 실행 결과
// Start
// 0 10 20 ... 100
// Finished
// Dispose
```

### Observable.Start

주어진 블록을 다른 스레드에서 실행하여 결과를 1개만 발급합니다.  
메인 스레드로 전환이 필요한 경우 `ObserveOnMainThread()`를 사용합니다.

```csharp
Observable.Start(() =>
{
    var req = (HttpWebRequest) WebRequest.Create("https://google.com");
    var res = (HttpWebResponse) req.GetResponse();
    using (var reader = new StreamReader(res.GetResponseStream()))
    {
        return reader.ReadToEnd();
    }
})
.ObserveOnMainThread()
.Subscribe(x => Debug.Log(x));
```

### Observable.Timer / TimerFrame

일정 시간 후에 메시지를 발행합니다.

```csharp
// 5초 후에 메시지 발행하고 종료
Observable.Timer(TimeSpan.FromSeconds(5))
    .Subscribe(_ => Debug.Log("5초 경과했습니다."));

// 5초 후 메시지 발행 후 1초 간격으로 계속 발행
Observable.Timer(TimeSpan.FromSeconds(5), TimeSpan.FromSeconds(1))
    .Subscribe(_ => Debug.Log("주기적으로 수행되고 있습니다."))
    .AddTo(gameObject);
```

---

## UniRx.Triggers 시리즈

`using UniRx.Triggers;`를 사용하는 스트림 소스입니다.  
Unity 콜백 이벤트를 UniRx의 `IObservable`로 변환하여 제공합니다.  
**GameObject가 Destroy될 때 자동으로 OnCompleted를 발급**하여 수명 관리도 걱정 없습니다.

- [GitHub – UniRx.Triggers](https://github.com/neuecc/UniRx/wiki/UniRx.Triggers)

```csharp
private void Start()
{
    bool isForceEnabled = true;
    var rb = GetComponent<Rigidbody>();

    // 플래그가 유효한 동안 위쪽에 힘을 가한다.
    this.FixedUpdateAsObservable()
        .Where(_ => isForceEnabled)
        .Subscribe(_ => rb.AddForce(Vector3.up * 20));

    // WarpZone에 침입하면 플래그를 활성화한다.
    this.OnTriggerEnterAsObservable()
        .Where(x => x.gameObject.CompareTag("WarpZone"))
        .Subscribe(_ => isForceEnabled = true);

    // WarpZone에 나오면 플래그를 해제한다.
    this.OnTriggerExitAsObservable()
        .Where(x => x.gameObject.CompareTag("WarpZone"))
        .Subscribe(_ => isForceEnabled = false);
}
```

---

## 코루틴에서 변환

`Observable.FromCoroutine`을 이용하여 코루틴을 `IObservable`로 변환할 수 있습니다.

```csharp
public bool IsPaused { get; private set; }

private void Start()
{
    Observable.FromCoroutine<int>(observer => TimerCoroutine(observer, 60))
        .Subscribe(t => Debug.Log(t));
}

IEnumerator TimerCoroutine(IObserver<int> observer, int initializeTime)
{
    var current = initializeTime;
    while (current > 0)
    {
        if (!IsPaused)
        {
            observer.OnNext(current--);
        }
        yield return new WaitForSeconds(1);
    }
    observer.OnNext(0);
    observer.OnCompleted();
}
```

---

## uGUI 이벤트에서 변환

UniRx를 using하면 uGUI 컴포넌트의 이벤트를 Observable로 바로 사용할 수 있습니다.

```csharp
private Button button;
private InputField inputField;
private Slider slider;

private void Start()
{
    button.OnClickAsObservable().Subscribe(_ =>
    {
        Debug.Log("button OnClick!");
    });

    inputField.OnValueChangedAsObservable().Subscribe(str =>
    {
        Debug.Log("inputField OnValueChanged : " + str);
    });

    slider.OnValueChangedAsObservable().Subscribe(val =>
    {
        Debug.Log("slider value changed : " + val);
    });

    // Subscribe시 초기값 포함 여부의 차이:
    inputField.OnValueChangedAsObservable(); // 초기값이 있다.
    inputField.onValueChanged.AsObservable(); // 초기값이 없다.
}
```

---

[← 목차](./README.md)
