# ScriptableObject

## 1. ScriptableObject란?

**ScriptableObject**는 클래스 인스턴스와 별개로 대량의 데이터를 저장할 수 있는 **데이터 컨테이너**입니다.  
유니티의 `MonoBehaviour`가 게임 오브젝트에 부착되어 '동작'을 제어한다면, `ScriptableObject`는 프로젝트 폴더 내에 파일(`.asset`) 형태로 존재하며 '데이터' 자체를 관리하는 데 특화되어 있습니다.

---

## 2. 주요 장점 (Pros)

- **메모리 효율성:** 동일한 데이터를 사용하는 1,000개의 프리팹이 있을 때, `MonoBehaviour`는 데이터를 각각 복사해서 가지지만, `SO`는 단 하나의 에셋 파일을 참조하므로 메모리 낭비를 획기적으로 줄입니다.
- **워크플로우 개선:** 코드를 수정하지 않고도 에디터 인스펙터 창에서 기획자나 아티스트가 수치를 실시간으로 수정할 수 있습니다.
- **씬 간 데이터 유지:** 씬이 전환되어도 `SO`에 저장된 데이터는 파괴되지 않고 유지됩니다.
- **유연한 아키텍처:** 에셋 기반이므로 인스펙터에서 드래그 앤 드롭으로 데이터를 쉽게 교체할 수 있어 모듈화된 설계를 돕습니다.

---

## 3. 주요 단점 및 주의점 (Cons)

- **런타임 데이터 저장 불가:** 게임 실행 중에 `SO` 값을 변경하면 에디터에서는 저장되지만, **빌드된 실제 게임에서는 앱 종료 시 변경 사항이 초기화됩니다.** (영구 저장을 위해서는 JSON이나 Binary 저장을 병행해야 합니다.)
- **라이프사이클의 부재:** `Update`나 `FixedUpdate` 같은 유니티 이벤트 함수를 사용할 수 없습니다.

---

## 4. 핵심 활용 사례

1. **아이템/몬스터 데이터베이스:** 공격력, 방어력, 아이콘, 이름 등 공통 데이터 관리.
2. **게임 설정:** 오디오 볼륨, 난이도 세팅, 물리 상수값 저장.
3. **이벤트 시스템 (SO Architecture):** 스크립트 간의 결합도를 낮추기 위한 게임 이벤트 채널로 활용.

---

## 5. 예제 코드

### 데이터 구조 정의 (ItemData.cs)

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "NewItem", menuName = "ScriptableObjects/ItemData", order = 1)]
public class ItemData : ScriptableObject
{
    [Header("Basic Info")]
    public string itemName;
    public Sprite itemIcon;
    public string description;

    [Header("Stats")]
    public int price;
    public float weight;

    public void LogItemInfo()
    {
        Debug.Log($"아이템 이름: {itemName}, 가격: {price}");
    }
}
```

### 데이터 사용 (ItemHandler.cs)

```csharp
using UnityEngine;

public class ItemHandler : MonoBehaviour
{
    public ItemData data;

    void Start()
    {
        if (data != null)
        {
            Debug.Log($"장착된 아이템: {data.itemName}");
            data.LogItemInfo();
        }
    }
}
```

---

## 6. 요약 표

| 특징 | MonoBehaviour | ScriptableObject |
|------|---------------|-----------------|
| **존재 방식** | 씬 내 게임 오브젝트에 부착 | 프로젝트 창 내 에셋 파일로 존재 |
| **메모리** | 인스턴스마다 데이터 복제 | 모든 인스턴스가 하나의 에셋 참조 |
| **주 용도** | 게임 로직, 캐릭터 움직임 등 | 데이터 저장, 설정값, 상태 공유 |
| **Update 사용** | 가능 | 불가능 |

---

[← 목차](./README.md)
