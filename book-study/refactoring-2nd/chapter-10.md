# 10장 — 조건부 로직 간소화

## 1. 조건문 분해하기

**요약** — 조건식과 그에 딸린 조건절 각각을 함수로 추출한다.

**절차**
1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.

**Before**
```csharp
void Function()
{
    if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
    {
        charge = quantity * plan.summerRate;
    }
    else
    {
        charge = quantity * plan.regularRate + plan.regularServiceCharge;
    }
}
```

**After** — 조건식을 함수로 추출하라
```csharp
void Function()
{
    charge = Summer() ? SummerCharge() : RegularCharge();
}

bool Summer()
{
    return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}

float SummerCharge()
{
    return quantity * plan.summerRate;
}

float RegularCharge()
{
    return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

---

## 2. 중복 조건식 통합하기

**요약** — 결과로 같은 일을 행하는 조건문들을 통합한다.

**절차**
1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트한다.
4. 조건이 하나만 남을 때까지 결합과 테스트 반복.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

**Before**
```csharp
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthDisabled < 12) return 0;
if (anEmployee.isPartTime) return 0;
```

**After** — 결과로 같은 일을 행한다면 통합하라
```csharp
if (IsNotEligibleForDisability()) return 0;

bool IsNotEligibleForDisability()
{
    return ((anEmployee.seniority < 2)
            || (anEmployee.monthDisabled < 12)
            || (anEmployee.isPartTime));
}
```

---

## 3. 중첩 조건문을 보호 구문으로 바꾸기

**요약** — 조건문의 중첩보다 보호 구문이 확인하기 편하다.

**절차**
1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 보호 구문으로의 변경과 테스트를 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

**Before**
```csharp
float GetPayAmount()
{
    var result;

    if (isDead)
        result = DeadAmount();
    else
    {
        if (isSeparated)
            result = SeparatedAmount();
        else
            if (isRetired)
                result = RetiredAmount();
            else
                result = NormalPayAmount();
    }

    return result;
}
```

**After** — 보호 구문 사용
```csharp
float GetPayAmount()
{
    if (isDead) return DeadAmount();
    if (isSeparated) return SeparatedPayAmount();
    if (isRetired) return RetiredAmount();
    return NormalPayAmount();
}
```

---

## 4. 조건부 로직을 다형성으로 바꾸기

**요약** — 같은 유형은 상속으로 관리하라.

**절차**
1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.
4. 서브클래스 중 하나를 선택하여 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다.

**Before**
```csharp
string BirdEncounter(Bird bird)
{
    switch (bird.type)
    {
        case '유럽 제비': return "보통이다";
        case '아프리카 제비': return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
        case '노르웨이 파랑 앵무': return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
        default: return "알 수 없다";
    }
}
```

**After** — 상속으로 관리
```csharp
abstract class Bird
{
    public abstract string BirdEncounter { get; }
    public abstract float BirdSpeed { get; }

    public int numberOfCoconuts;
    public int voltage;
    public bool isNailed;
}

class EuroSwallow : Bird
{
    public override string BirdEncounter => "보통이다";
    public override float BirdSpeed => 35;
}

class AfricaSwallow : Bird
{
    public override string BirdEncounter => (numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    public override float BirdSpeed => 40 - 2 * numberOfCoconuts;
}

class NorwaySwallow : Bird
{
    public override string BirdEncounter => (voltage > 100) ? "그을렸다" : "예쁘다";
    public override float BirdSpeed => isNailed ? 0 : voltage / 10;
}
```

---

## 5. 특이 케이스 추가하기

**요약** — 특정한 값에 동일한 반응을 한다면 Null Object 패턴을 활용하라.

**절차**
1. 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, `false`를 반환하게 한다.
2. 특이 케이스 객체를 만든다. 이 속성은 `true`를 반환하게 한다.
3. 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다.
4. 코드에 새로운 특이 케이스 대상을 추가한다.
5. 테스트한다.
6. 여러 함수를 클래스로 묶기나 변환 함수로 묶기를 적용하여 공통 동작을 새로운 요소로 옮긴다.
7. 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인한다.

**Null Object Pattern 예시**
```csharp
public interface IAnimal
{
    string GetName();
    string MakeSound();
}

public class Duck : IAnimal
{
    public string GetName() => "Duck";
    public string MakeSound() => "Quack quack!";
}

public sealed class NullAnimal : IAnimal
{
    private static readonly IAnimal nullAnimal = new NullAnimal();
    private NullAnimal() { }

    public string GetName() => string.Empty;
    public string MakeSound() => string.Empty;

    public static IAnimal Instance => nullAnimal;
}

public class AnimalFactory
{
    public static IAnimal GetAnimal(string name)
    {
        return name switch
        {
            "duck" => new Duck(),
            _ => NullAnimal.Instance
        };
    }
}
```

---

## 6. 어서션 추가하기

**요약** — 참이라고 가정하는 조건을 명시하는 어서션을 추가한다.

- 절대로 발생하지 않아야 하는 조건을 런타임 중에 검사
- 디버그 모드에서만 동작 (릴리즈 모드에서는 무시)
- 최종 제품의 성능 저하 없이 개발 중에 문제를 고치는 바람직한 방법

```csharp
using System.Diagnostics;

Debug.Assert(menu > 0);
if (discountRate != null)
    result = cost * discountRate;
```

---

## 7. 제어 플래그를 탈출문으로 바꾸기

**요약** — 간소화할 수 있는 제어 플래그는 제거하라.

**절차**
1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려한다.
2. 제어 플래그를 갱신하는 코드 각각을 적절한 제어문으로 바꾼다.
3. 모두 수정했다면 제어 플래그를 제거한다.

**Before**
```csharp
foreach (Person person in People)
{
    if (!found)
    {
        if (person.type == "조커")
        {
            // Function...
            found = true;
        }
    }
}
```

**After**
```csharp
foreach (Person person in People)
{
    if (person.type == "조커")
    {
        // Function...
        break;
    }
}
```

---

[← 목차](./README.md)
