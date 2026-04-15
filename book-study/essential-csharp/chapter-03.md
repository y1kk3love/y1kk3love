# 3장 — 고급 데이터 형식

## 값 형식 vs 참조 형식

### 값 형식
- 데이터를 바로 가지고 있음.
- 깊은 복사 — 데이터를 직접 복사함.
- 16바이트 이하로 유지해야 함.

### 참조 형식
- 데이터가 존재하는 메모리의 주소를 가지고 있음.
- 얕은 복사 — 주소를 복사함.
  - 여러 변수에서 하나의 주소를 가리켜 원하지 않은 데이터 변화에 주의.
  - 실제 데이터를 복사하는 것이 아니기 때문에 그만큼 빠름.
- 보통 힙에서 관리함. 참조라고 해서 항상 힙에 있는 것은 아님 (ex: ref 키워드).
- **역참조**: 런타임에서 데이터의 메모리 주소를 통해 실제 데이터에 접근하는 작업.

---

## 암시적 형식 지역 변수

`var`는 컴파일러가 초기화 식의 데이터에 따라 형식을 결정한다.

```csharp
var text = "Hello World!";
```

---

## 익명 형식

```csharp
public void Main()
{
    var patent = new {
        Name = "Jerry",
        Age = "25"
    };

    Console.WriteLine($"{patent.Name}");
}
// 결과: Jerry
```

---

## 튜플

익명 형식은 튜플과 같이 데이터 요소를 결합하는 일에 사용된다.

---

## 배열

**^ 연산자** — 끝에서부터 접근
```csharp
string[] text = new string[9];

// 마지막 인덱스
Console.WriteLine(text[^1]);
// 뒤에서 두 번째
Console.WriteLine(text[^2]);
```

**Rank** — C#에서 배열의 차원
```csharp
string[,] chess = new string[8, 8];
Console.WriteLine(chess.Rank); // 2
```

**Length** — 다차원 배열일 경우 전체 요소의 수 반환
```csharp
string[,] chess = new string[8, 8];
Console.WriteLine(chess.Length); // 64
```

**범위 연산자** — 배열에서 지정한 범위만큼 반환
```csharp
int[] numbers = Enumerable.Range(0, 100).ToArray();
int x = 12;
int y = 25;

Console.WriteLine($"{numbers[^x]}");             // 뒤에서 x번째
Console.WriteLine($"{numbers[x..y].Length}");    // x~y 구간 길이

Span<int> x_y = numbers[x..y];
Span<int> y_z = numbers[y..36];
```

---

[← 목차](./README.md)
