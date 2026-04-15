# 2장 — 데이터 형식

## 자료형

### 정수

| 형식 | 크기 | 범위 | BCL 이름 | 리터럴 접미사 |
|------|------|------|----------|--------------|
| sbyte | 8 bits | -2⁷ ~ 2⁷-1 | Sbyte | |
| byte | 8 bits | 0 ~ 2⁸-1 | Byte | |
| short | 16 bits | -2¹⁵ ~ 2¹⁵-1 | Int16 | |
| ushort | 16 bits | 0 ~ 2¹⁶-1 | UInt16 | |
| int | 32 bits | -2³¹ ~ 2³¹-1 | Int32 | |
| uint | 32 bits | 0 ~ 2³²-1 | Uint32 | U, u |
| long | 64 bits | -2⁶³ ~ 2⁶³-1 | Int64 | L, l |
| ulong | 64 bits | 0 ~ 2⁶⁴-1 | UInt64 | UL, ul |

### 이진 부동 소수점

2진수 → 10진수 변환, 반올림에 의한 값의 변형이 존재함.

| 형식 | 크기 | BCL 이름 | 리터럴 접미사 |
|------|------|----------|--------------|
| float | 32 bits | Single | F, f |
| double | 64 bits | Double | D, d |

### 십진 부동 소수점

정확한 10진수 정밀도 제공, 속도 느림.

| 형식 | 크기 | BCL 이름 | 리터럴 접미사 |
|------|------|----------|--------------|
| decimal | 128 bits | Decimal | M, m |

### Bool / 문자

| 형식 | 크기 | BCL 이름 |
|------|------|----------|
| bool | 8 bits | Boolean |
| char | 16 bits | Char |
| string | ~2GB | String |

> `String`은 새로운 데이터를 넣거나 복사 반환은 받을 수 있지만 데이터 자체는 변하지 않는다.  
> 데이터를 변경하고 싶다면 `System.Text.StringBuilder`를 사용하라.

---

## Nullable

Null은 값을 참조하지 않고 있다는 이야기.  
Nullable은 Value를 가지는 기본형을 Value를 참조하는 객체로 만들어 Null을 할당할 수 있게 바꾼다.

```csharp
int? nullable = null;
```

---

## Void

메서드가 데이터를 반환하지 않음을 컴파일러에 선언함.

```csharp
public void VoidMethod()
{
    //...
}
```

---

## Casting

**명시적 캐스팅** — 정밀도와 값의 근본적인 변화를 감수하는 형 변환
```csharp
long longNum = 50918309109;
int intNum = (int)longNum;
```

**Checked & Unchecked** — 블록 안에서 오버플로우 발생 여부 확인
```csharp
checked
{
    long longNum = 50918309109;
    int intNum = (int)longNum; // 오버플로우 시 예외 발생
}

unchecked
{
    long longNum = 50918309109;
    int intNum = (int)longNum; // 오버플로우 무시
}
```

**암시적 변환** — 정밀도와 값의 근본적인 변화가 일어나지 않는 형 변환
```csharp
int intNum = 1;
long longNum = intNum;
```

**캐스팅 연산 없는 형식 변환**
```csharp
long longNum = 1;
string text = "1";
int intNum = int.Parse(longNum.ToString());
double intNum2 = System.Convert.ToDouble(text);
```

**TryParse()**
```csharp
if (int.TryParse(input, out value)) { }
```

---

[← 목차](./README.md)
