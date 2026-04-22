# 랜덤 확률 시스템

게임에서 "공정한 랜덤"을 구현하기 위한 기법들. 순수 난수 생성기는 클러스터링, 모듈로 편향 등 체감 불공정을 유발하므로, 목적에 맞는 보정이 필요하다.

---

## 1. RNG 소스 선택

C#의 난수 생성기는 알고리즘에 따라 분포 품질이 다르다.

```csharp
// 나쁨: 선형 합동 생성기(LCG) 기반 — 분포 편향, 멀티스레드 비안전
var rng = new System.Random();

// 좋음: .NET 6+ Xoshiro256** 기반 — 균등 분포, thread-safe
var value = Random.Shared.NextDouble();

// 크립토 수준이 필요할 때 (보안/결제 관련)
byte[] buf = new byte[4];
RandomNumberGenerator.Fill(buf);
uint value = BitConverter.ToUInt32(buf);
```

| 생성기 | 알고리즘 | 속도 | 품질 | 용도 |
|--------|---------|------|------|------|
| `new Random()` | LCG | 빠름 | 낮음 | 사용 지양 |
| `Random.Shared` | Xoshiro256** | 빠름 | 높음 | 일반 게임 로직 |
| `RandomNumberGenerator` | CSPRNG | 느림 | 최고 | 보안/결제 |

---

## 2. 모듈로 편향 (Modulo Bias)

`% n` 연산은 uint 최대값이 n의 배수가 아닐 때, `0 ~ (UInt32.MaxValue % n)` 범위의 숫자가 더 자주 등장하는 편향이 생긴다.

```csharp
// 편향 있음: n=6이면 0~3이 0.00000145% 더 자주 나옴
uint biased = rawValue % 6;

// 편향 없음: rejection sampling — 편향 구간을 버리고 재추출
static uint UnbiasedRandom(uint n)
{
    uint threshold = (uint.MaxValue - n + 1) % n;
    uint value;
    do
    {
        Span<byte> buf = stackalloc byte[4];
        RandomNumberGenerator.Fill(buf);
        value = BitConverter.ToUInt32(buf);
    } while (value < threshold);
    return value % n;
}
```

> 게임 로직에서는 체감이 거의 없지만, 보안/결제 시스템에서는 반드시 제거해야 한다.

---

## 3. Shuffle Bag (보장 시스템)

순수 난수는 "10번 연속 실패" 같은 클러스터링이 발생할 수 있다.  
Shuffle Bag은 아이템 풀을 섞어 꺼내는 방식으로 **실제 확률을 주기 내에서 보장**한다.

```csharp
public class ShuffleBag<T>
{
    private readonly List<T> _source = new();
    private readonly List<T> _bag = new();

    public void Add(T item, int weight)
    {
        for (int i = 0; i < weight; i++) _source.Add(item);
    }

    public T Next()
    {
        if (_bag.Count == 0) Refill();

        T item = _bag[^1];
        _bag.RemoveAt(_bag.Count - 1);
        return item;
    }

    private void Refill()
    {
        _bag.AddRange(_source);
        // Fisher-Yates shuffle
        for (int i = _bag.Count - 1; i > 0; i--)
        {
            int j = Random.Shared.Next(i + 1);
            (_bag[i], _bag[j]) = (_bag[j], _bag[i]);
        }
    }
}

// 30% 확률이면 10판 안에 정확히 3번 등장 보장
var bag = new ShuffleBag<string>();
bag.Add("rare",   3);
bag.Add("common", 7);
string result = bag.Next();
```

**적합한 상황:** 아이템 드랍, 카드 덱, 맵 랜덤 이벤트처럼 "연속 중복"을 방지해야 하는 경우

---

## 4. 피티 시스템 (Pity System)

연속 실패 횟수에 따라 확률을 누적 증가시켜 "무한 불운"을 방지하는 방식.  
소프트 피티(확률 점진 증가) + 하드 피티(특정 횟수에서 100% 보장) 조합이 일반적이다.

```csharp
public class PitySystem
{
    private int _failCount;
    private readonly float _baseProb;
    private readonly float _increment; // 실패마다 확률 증가량
    private readonly int _hardPity;    // 이 횟수에서 100% 보장

    public PitySystem(float baseProb, float increment, int hardPity)
    {
        _baseProb  = baseProb;
        _increment = increment;
        _hardPity  = hardPity;
    }

    public bool Roll()
    {
        if (_failCount >= _hardPity)
        {
            _failCount = 0;
            return true; // 하드 피티
        }

        float currentProb = Math.Min(1f, _baseProb + _increment * _failCount);
        bool success = Random.Shared.NextSingle() < currentProb;

        _failCount = success ? 0 : _failCount + 1;
        return success;
    }

    public int FailCount => _failCount; // 현재 피티 카운터 노출 (UI용)
}

// 기본 0.6%, 10번마다 +4.3%, 90번에서 100% 보장 (원신 5성 모사)
var pity = new PitySystem(baseProb: 0.006f, increment: 0.043f, hardPity: 90);
bool isSSR = pity.Roll();
```

**적합한 상황:** 가챠, 희귀 아이템 드랍처럼 기대값보다 분산이 크면 플레이어 경험이 나빠지는 경우

---

## 5. 기법 선택 기준

| 상황 | 추천 방식 |
|------|----------|
| 일반 게임 로직 | `Random.Shared` (Xoshiro256**) |
| 보안/결제 관련 | `RandomNumberGenerator` + rejection sampling |
| 연속 중복 방지, 비율 보장 | Shuffle Bag |
| 연속 실패 방지, 불운 보호 | Pity System |
| 시드 기반 재현 필요 | `new Random(seed)` + Fisher-Yates |

> 순수 랜덤보다 **플레이어가 공정하다고 느끼는 랜덤**이 목표.  
> 실전에서는 Shuffle Bag + Pity를 계층적으로 조합하는 경우가 많다.

---

[← 목차](./README.md)
