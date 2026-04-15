# 1. C# 메모리 관리 및 가비지 컬렉션 (GC)

---

## Managed Heap vs Stack

| | Stack | Heap |
|--|-------|------|
| **저장 대상** | 값 타입 (`int`, `float`, `struct` 등) | 참조 타입 (`class`, `string`, `object` 등) |
| **해제 방식** | 스코프를 벗어나면 즉시 자동 해제 | GC가 주기적으로 판단하여 해제 |
| **속도** | 빠름 | 상대적으로 느림 |
| **크기** | 제한적 | 동적 할당 |

```csharp
void Example()
{
    int a = 10;              // Stack — 메서드 종료 시 즉시 해제
    MyClass obj = new MyClass(); // Heap — GC가 수거할 때 해제
}
```

---

## GC 동작 원리 (Mark-and-Sweep)

1. **Marking:** Root(스택, 정적 변수 등)에서 참조가 닿는 객체를 추적하여 마킹한다.
2. **Sweeping:** 마킹되지 않은 객체(가비지)를 메모리에서 해제한다.
3. **Compaction:** 흩어진 메모리 조각을 모아 단편화(Fragmentation)를 방지한다.

---

## Generational GC (세대별 관리)

Unity의 최신 GC는 세대(Generation)별로 관리한다.

| 세대 | 설명 |
|------|------|
| **0세대** | 새로 생성된 객체. GC 검사 빈도 가장 높음. |
| **1세대** | 0세대 GC를 생존한 객체. 중간 빈도. |
| **2세대** | 오래 살아남은 객체. GC 검사 빈도 가장 낮음. |

수명이 짧은 객체는 0세대에서만 수거되므로 전체 GC 비용이 줄어든다.  
오래 살아남는 객체가 많을수록 2세대 GC 비용이 커진다.

---

## 면접 대응 전략

| 문제 상황 | 원인 | 해결 방안 |
|-----------|------|-----------|
| **GC Spike (프레임 드랍)** | 매 프레임 new 할당 누적 | 오브젝트 풀링(Object Pooling) |
| **잦은 가비지 생성** | Update 내 new, 문자열 조합 | `StringBuilder`, 캐싱 |
| **박싱(Boxing)** | `object` 타입으로 값 타입 전달 | 제네릭(Generic) 활용 |
| **Enum 비교 시 박싱** | `Enum.HasFlag` 등 내부 박싱 | 직접 비트 연산으로 대체 |

---

## 코드 예제

### Object Pooling

```csharp
public class BulletPool : MonoBehaviour
{
    [SerializeField] private GameObject _prefab;
    private Queue<GameObject> _pool = new Queue<GameObject>();

    public GameObject Get()
    {
        if (_pool.Count > 0)
        {
            var obj = _pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        return Instantiate(_prefab);
    }

    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

### StringBuilder — 문자열 GC 방지

```csharp
// 나쁜 예: 매 프레임 새 문자열 객체 생성
void Update()
{
    _text.text = "HP: " + _hp + " / " + _maxHp; // GC 발생
}

// 좋은 예: StringBuilder 재사용
private StringBuilder _sb = new StringBuilder();

void Update()
{
    _sb.Clear();
    _sb.Append("HP: ").Append(_hp).Append(" / ").Append(_maxHp);
    _text.text = _sb.ToString();
}
```

### Generic으로 박싱 방지

```csharp
// 나쁜 예: object 타입으로 박싱 발생
void Send(object value) { }
Send(42); // int → object 박싱

// 좋은 예: 제네릭으로 박싱 없음
void Send<T>(T value) { }
Send(42); // 박싱 없음
```

---

[← 목차](./README.md)
