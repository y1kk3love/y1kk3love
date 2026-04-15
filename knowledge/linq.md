# LINQ (Language Integrated Query)

C# 컬렉션에 쿼리 연산을 통합한 기능. 복잡한 데이터 처리를 선언적이고 간결하게 작성할 수 있다.

---

## 1. 기본 구문

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// 메서드 체인 방식 (권장)
var result = numbers
    .Where(n => n % 2 == 0)    // 짝수만
    .Select(n => n * n)         // 제곱
    .OrderByDescending(n => n); // 내림차순

// 쿼리 표현식 방식
var result2 = from n in numbers
              where n % 2 == 0
              orderby n descending
              select n * n;
```

---

## 2. 주요 메서드

### 필터링

```csharp
var enemies = FindObjectsOfType<Enemy>();

// 살아있는 적만
var alive = enemies.Where(e => e.HP > 0);

// 범위 내 적만
var inRange = enemies.Where(e => Vector3.Distance(transform.position, e.transform.position) < 10f);
```

### 변환 (Select)

```csharp
// Enemy 리스트 → Transform 리스트
var transforms = enemies.Select(e => e.transform);

// 익명 타입으로 변환
var enemyInfo = enemies.Select(e => new { e.name, e.HP, Distance = Vector3.Distance(transform.position, e.transform.position) });
```

### 집계

```csharp
int totalHP = enemies.Sum(e => e.HP);
float avgHP = enemies.Average(e => e.HP);
int maxHP = enemies.Max(e => e.HP);
Enemy weakest = enemies.MinBy(e => e.HP); // C# 10+
```

### 검색

```csharp
bool anyAlive = enemies.Any(e => e.HP > 0);
bool allDead = enemies.All(e => e.HP <= 0);
Enemy first = enemies.First(e => e.HP > 0);            // 없으면 예외
Enemy firstOrNull = enemies.FirstOrDefault(e => e.HP > 0); // 없으면 null
```

### 정렬

```csharp
// 가장 가까운 적 순서대로 정렬
var sorted = enemies
    .OrderBy(e => Vector3.Distance(transform.position, e.transform.position));

// 복합 정렬: HP 오름차순 → 이름 오름차순
var complex = enemies
    .OrderBy(e => e.HP)
    .ThenBy(e => e.name);
```

### 그룹화

```csharp
// 팀별 그룹화
var byTeam = enemies.GroupBy(e => e.Team);
foreach (var group in byTeam)
{
    Debug.Log($"팀 {group.Key}: {group.Count()}명");
}
```

---

## 3. Unity에서의 실용 예제

```csharp
// 씬에서 조건을 만족하는 오브젝트 찾기
var interactables = FindObjectsOfType<MonoBehaviour>()
    .OfType<IInteractable>()
    .Where(i => i.IsAvailable)
    .OrderBy(i => Vector3.Distance(transform.position, ((MonoBehaviour)i).transform.position))
    .ToList();

// 가장 가까운 적 찾기
Enemy nearest = FindObjectsOfType<Enemy>()
    .Where(e => e.IsAlive)
    .OrderBy(e => (e.transform.position - transform.position).sqrMagnitude)
    .FirstOrDefault();

// Inventory 아이템 필터링
var weapons = _inventory
    .Where(item => item.Type == ItemType.Weapon)
    .OrderByDescending(item => item.Damage)
    .Take(3); // 상위 3개만
```

---

## 4. 성능 주의사항

LINQ는 내부적으로 이터레이터와 클로저를 생성하므로 **GC 할당이 발생**한다.

| 상황 | 권장 |
|------|------|
| 매 프레임 반복 실행 | `for` / `foreach` 직접 사용 |
| 초기화 / 이벤트 처리 | LINQ 사용 가능 |
| 대규모 컬렉션 가공 | LINQ 편의성 > 소량의 GC |

```csharp
// Update에서는 LINQ 대신 캐싱 or 직접 루프 사용
void Update()
{
    // 나쁜 예: 매 프레임 GC 발생
    var alive = _enemies.Where(e => e.HP > 0).ToList();

    // 좋은 예: 변화가 있을 때만 갱신
    if (_listDirty)
    {
        _aliveEnemies = _enemies.Where(e => e.HP > 0).ToList();
        _listDirty = false;
    }
}
```

---

[← 목차](./README.md)
