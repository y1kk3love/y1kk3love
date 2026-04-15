# 2. OnNext와 Subscribe

OnNext와 Subscribe는 모두 "Subject에 구현된 메서드"이며, 각각 다음과 같은 동작을 합니다.

- **Subscribe:** 메시지 수신시 실행을 함수에 등록한다.
- **OnNext:** Subscribe에 등록된 함수에 메시지를 전달하고 실행한다.

```csharp
// Subject 작성
Subject<string> subject = new Subject<string>();

// 3회 Subscribe
subject.Subscribe(msg => Debug.Log("Subscribe1 : " + msg));
subject.Subscribe(msg => Debug.Log("Subscribe2 : " + msg));
subject.Subscribe(msg => Debug.Log("Subscribe3 : " + msg));

// 이벤트 메시지 발행
subject.OnNext("안녕하세요");
subject.OnNext("안녕");

// 결과
// Subscribe1 : 안녕하세요
// Subscribe2 : 안녕하세요
// Subscribe3 : 안녕하세요
// Subscribe1 : 안녕
// Subscribe2 : 안녕
// Subscribe3 : 안녕
```

event와 같이 구독되어 있는 메서드를 순서대로 처리해 나가고 있음을 볼 수 있습니다.

---

[← 목차](./README.md)
