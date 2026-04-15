# Addressable Asset System

Unity의 에셋 로딩 시스템. 에셋에 주소(address)를 부여하여 경로 대신 키로 비동기 로드하고, 메모리 해제까지 명시적으로 관리한다.

---

## 1. 기존 방식과의 비교

| | Resources.Load | AssetBundle | Addressable |
|--|---------------|-------------|-------------|
| 로드 방식 | 동기 | 비동기 (수동) | 비동기 (자동) |
| 경로 변경 | 코드 수정 필요 | 수동 관리 | 주소만 바꾸면 됨 |
| 메모리 해제 | Resources.UnloadUnused | 수동 Unload | Release로 명시 해제 |
| 원격 배포 | 불가 | 가능 | 기본 지원 |

---

## 2. 기본 로드 및 해제

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

public class AssetLoader : MonoBehaviour
{
    private AsyncOperationHandle<GameObject> _handle;

    async void Start()
    {
        // 주소로 비동기 로드
        _handle = Addressables.LoadAssetAsync<GameObject>("Prefabs/Enemy/Goblin");
        await _handle.Task;

        if (_handle.Status == AsyncOperationStatus.Succeeded)
        {
            Instantiate(_handle.Result);
        }
    }

    void OnDestroy()
    {
        // 반드시 Release — 하지 않으면 메모리 누수
        if (_handle.IsValid())
            Addressables.Release(_handle);
    }
}
```

---

## 3. InstantiateAsync — 인스턴스화까지 한 번에

```csharp
public class EnemySpawner : MonoBehaviour
{
    private List<AsyncOperationHandle<GameObject>> _spawnHandles = new();

    public async void SpawnEnemy(string address, Vector3 position)
    {
        var handle = Addressables.InstantiateAsync(address, position, Quaternion.identity);
        await handle.Task;

        _spawnHandles.Add(handle);
    }

    public void DespawnAll()
    {
        foreach (var handle in _spawnHandles)
        {
            // ReleaseInstance는 오브젝트 파괴 + 메모리 해제를 함께 처리
            Addressables.ReleaseInstance(handle);
        }
        _spawnHandles.Clear();
    }
}
```

---

## 4. 라벨(Label)로 묶음 로드

```csharp
// "UI" 라벨이 붙은 에셋 전부 로드
public async void LoadUIAssets()
{
    var handle = Addressables.LoadAssetsAsync<Sprite>(
        "UI",                           // 라벨
        sprite => {                     // 개별 에셋 로드 완료 콜백
            Debug.Log($"로드됨: {sprite.name}");
        }
    );

    await handle.Task;
    // handle.Result → IList<Sprite>
}
```

---

## 5. AssetReference — Inspector 연동

```csharp
public class WeaponHolder : MonoBehaviour
{
    [SerializeField] private AssetReferenceGameObject _weaponRef;

    private AsyncOperationHandle<GameObject> _handle;

    public async void EquipWeapon()
    {
        _handle = _weaponRef.InstantiateAsync(transform.position, transform.rotation);
        await _handle.Task;
    }

    public void UnequipWeapon()
    {
        if (_weaponRef.IsValid())
            _weaponRef.ReleaseInstance(_handle);
    }
}
```

> Inspector에서 드래그로 에셋을 연결하면 GUID로 직렬화되어 경로 변경에 안전하다.

---

## 6. 원격 배포 구조

```
로컬 빌드 앱
    ↓ 필요할 때
CDN / Remote 서버
    → 에셋 번들 다운로드 & 캐싱
```

```csharp
// 카탈로그 업데이트 확인 → 다운로드 → 사용
public async void UpdateAndLoad()
{
    // 1. 업데이트 가능한 카탈로그 확인
    var checkHandle = Addressables.CheckForCatalogUpdates(false);
    await checkHandle.Task;

    if (checkHandle.Result.Count > 0)
    {
        // 2. 카탈로그 업데이트
        var updateHandle = Addressables.UpdateCatalogs(checkHandle.Result, false);
        await updateHandle.Task;
        Addressables.Release(updateHandle);
    }
    Addressables.Release(checkHandle);
}
```

---

## 7. 메모리 관리 원칙

| 규칙 | 이유 |
|------|------|
| 로드한 핸들은 반드시 `Release` | 참조 카운트 기반 — Release 없이는 GC 안 됨 |
| `InstantiateAsync` → `ReleaseInstance` | Destroy만으로는 번들 메모리 미해제 |
| 씬 전환 시 명시적 정리 | 씬 Unload가 Addressable 메모리를 자동 해제하지 않음 |
| 라벨 묶음 해제 시 핸들 하나로 Release | 개별 에셋마다 Release 불필요 |

---

[← 목차](./README.md)
