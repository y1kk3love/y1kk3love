# 4. 오퍼레이터 & 스트림

UniRX에서는 Subject와 Subscribe 사이에 데이터를 필터링하는 과정 등을 추가할 수 있는데 이를 **오퍼레이터**라고 부릅니다.

## 오퍼레이터 예시 — Where

```csharp
// 문자열을 발행하는 Subject
Subject<string> subject = new Subject<string>();

// Enemy만 필터링
subject
    .Where(x => x == "Enemy") // 필터링 오퍼레이터
    .Subscribe(x => Debug.Log($"플레이어가 {x}에 충돌했습니다."));

// 이벤트 메시지 발급
subject.OnNext("Wall");
subject.OnNext("Wall");
subject.OnNext("Enemy");
subject.OnNext("Enemy");

// 결과
// 플레이어가 Enemy에 충돌했습니다.
// 플레이어가 Enemy에 충돌했습니다.
```

---

## 주요 오퍼레이터

- **Where** — 필터링
- **Select** — 메시지 변환
- **Distinct** — 중복 제거
- **Buffer** — 일정 개수가 올 때까지 대기
- **ThrottleFirst** — 단시간에 함께 온 경우 처음만 사용

---

## 스트림

UniRx에서 **"스트림"**은 "메시지가 발행된 후 Subscribe에 도달할 때까지의 일련의 처리 흐름"을 표현하는 단어입니다.

- "오퍼레이터를 결합하여 스트림을 구축"
- "Subscribe하여 스트림을 실행한다"
- "OnCompleted를 발행하여 스트림을 정지시킨다"

---

[← 목차](./README.md)
