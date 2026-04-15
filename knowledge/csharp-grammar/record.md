# Record

C# 9.0에서 도입된 참조 형식. 데이터를 전달하고 표현하기 위한 목적에 특화된 형식이다.

---

## 1. 선언

```csharp
// 위치 매개변수(Positional Parameters) 방식 — 가장 간결
public record PlayerResult(bool IsSuccess, int Score, string Message);

// 프로퍼티 명시 방식
public record PlayerResult
{
    public bool IsSuccess { get; init; }
    public int Score { get; init; }
    public string Message { get; init; }
}
```

`init` 접근자는 생성 시점에만 값을 설정할 수 있다. 이후에는 변경 불가.

---

## 2. 불변성 (Immutability)

`record`는 기본적으로 불변이다. 생성 후 값을 바꿀 수 없다.

```csharp
var result = new PlayerResult(true, 100, "클리어");

// 컴파일 오류 — init 프로퍼티는 생성 후 변경 불가
// result.Score = 200;
```

---

## 3. 값 비교 (Value Equality)

`class`는 참조(메모리 주소)를 비교하지만, `record`는 **내부 값**을 비교한다.

```csharp
var a = new PlayerResult(true, 100, "클리어");
var b = new PlayerResult(true, 100, "클리어");

Console.WriteLine(a == b);       // True  — 값이 같으면 같다
Console.WriteLine(a.Equals(b));  // True
Console.WriteLine(ReferenceEquals(a, b)); // False — 인스턴스는 다르다
```

`Equals`와 `==` 연산자를 자동으로 오버라이딩해주므로 별도 구현이 필요 없다.

---

## 4. with 식 — 복사 후 일부 변경

불변 객체를 유지하면서 일부 값만 바꾼 새 인스턴스를 만들 때 사용한다.

```csharp
var original = new PlayerResult(true, 100, "클리어");

// Score만 바꾼 새 인스턴스 생성
var updated = original with { Score = 200 };

Console.WriteLine(original.Score);  // 100
Console.WriteLine(updated.Score);   // 200
```

---

## 5. 구조 분해 (Deconstruction)

위치 매개변수 방식으로 선언한 `record`는 자동으로 구조 분해가 지원된다.

```csharp
public record PlayerResult(bool IsSuccess, int Score, string Message);

var result = new PlayerResult(true, 100, "클리어");

// 구조 분해로 변수에 바로 할당
var (isSuccess, score, message) = result;

Console.WriteLine(isSuccess);  // True
Console.WriteLine(score);      // 100
```

---

## 6. readonly record struct

`record class`(기본)는 힙에 할당된다.  
**극강의 성능**이 필요하다면 `readonly record struct`를 사용하면 **스택에 할당**되어 GC 부하가 없다.

```csharp
// 스택 할당 — GC 없음
public readonly record struct HitResult(bool IsHit, float Damage);

var hit = new HitResult(true, 35.5f);
var miss = new HitResult(false, 0f);

Console.WriteLine(hit == miss);  // False
```

`Vector3`처럼 자주 생성·소멸되는 경량 데이터에 적합하다.

---

## 7. 요약 표

| | class | record | readonly record struct |
|--|-------|--------|----------------------|
| **메모리** | 힙 | 힙 | 스택 |
| **비교** | 참조 비교 | 값 비교 | 값 비교 |
| **불변성** | 없음 (직접 구현) | `init`으로 불변 가능 | 완전 불변 |
| **with 식** | 미지원 | 지원 | 지원 |
| **구조 분해** | 직접 구현 필요 | 자동 지원 | 자동 지원 |
| **적합한 용도** | 상태·동작을 가진 객체 | 데이터 전달 객체 (DTO) | 경량 값 데이터 |

> 복잡한 상태를 가진 덩어리는 `class`로, 순수하게 전달할 데이터는 `record`로 분리하자.

---

[← 목차](./README.md)
