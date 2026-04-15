# Record

## 1. record의 등장

유니티(C# 8.0 이하) 환경에서 데이터를 묶어 보낼 때마다 매번 클래스를 만들고, 필드 선언하고, 생성자에서 값 대입해주던 그 번거로운 과정이 네이티브 C#의 `record`를 만나면서 완전히 달라졌다.

- **한 줄의 마법:** `public record DiggingResult(bool IsSuccess, long Amount, long Balance);` 이 한 줄이면 끝난다.
- **값 비교의 편리함:** 클래스는 메모리 주소가 다르면 `false`라 `Equals`를 일일이 오버라이딩해야 했는데, 레코드는 내부 값만 같으면 알아서 `true`를 뱉어준다.
- **사이드 이펙트 차단:** 기본적으로 불변(Immutable)이라 데이터가 도중에 변할 걱정이 없다.

---

## 2. 구조 분해와 반환 방식

- **구조 분해(Deconstruction):**

```csharp
var (isSuccess, amount, balance) = _economyService.ProcessDigging(userId);
```

객체를 즉시 변수로 쪼개서 받을 수 있어 코드가 직관적으로 변한다.

- **성능 최적화:** 극강의 성능이 필요하다면 `readonly record struct`를 사용하면 된다. 유니티의 `Vector3`처럼 스택 메모리를 쓰면서 레코드의 편리함을 유지할 수 있다.

---

### 요약

> "복잡한 상태를 가진 덩어리는 Class로, 순수하게 전달할 데이터는 Record로 분리하자. 그리고 그 결과값은 구조 분해로 깔끔하게 받아내는 것."

---

[← 목차](./README.md)
