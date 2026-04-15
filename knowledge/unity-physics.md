# Unity 물리 시스템 (Rigidbody / Collider)

Unity의 물리 엔진(PhysX)을 통해 현실적인 물리 시뮬레이션을 구현하는 핵심 시스템.

---

## 1. Rigidbody

물리 엔진의 제어를 받는 컴포넌트. 이 컴포넌트가 붙은 오브젝트만 중력, 힘, 충돌 반응을 받는다.

### 주요 속성

| 속성 | 설명 |
|------|------|
| `Mass` | 질량. 클수록 힘에 덜 반응. |
| `Drag` | 선형 저항 (공기 마찰). |
| `Angular Drag` | 회전 저항. |
| `Use Gravity` | 중력 적용 여부. |
| `Is Kinematic` | 물리 연산 비활성화. Transform으로만 제어. |
| `Interpolate` | 렌더링과 물리 스텝 사이 보간. 떨림 방지. |
| `Collision Detection` | Discrete / Continuous. 빠른 오브젝트는 Continuous. |

```csharp
private Rigidbody _rb;

void Start() => _rb = GetComponent<Rigidbody>();

void FixedUpdate()
{
    // 힘 적용 (FixedUpdate에서만)
    _rb.AddForce(Vector3.forward * 10f, ForceMode.Force);
}

void Jump()
{
    // 즉각적인 충격량 (ForceMode.Impulse)
    _rb.AddForce(Vector3.up * 5f, ForceMode.Impulse);
}
```

### ForceMode 비교

| ForceMode | 설명 | 적합한 상황 |
|-----------|------|------------|
| `Force` | 지속적인 힘 (질량 적용) | 추진력, 바람 |
| `Acceleration` | 지속적인 가속 (질량 무시) | 중력 재정의 |
| `Impulse` | 순간 충격량 (질량 적용) | 점프, 폭발 |
| `VelocityChange` | 순간 속도 변경 (질량 무시) | 텔레포트 이동 |

---

## 2. Collider

물리 충돌의 형태를 정의하는 컴포넌트. Rigidbody 없이도 정적 충돌체로 사용 가능.

### Collider 종류

| 종류 | 특징 | 비용 |
|------|------|------|
| `BoxCollider` | 박스 형태. 가장 빠름. | 낮음 |
| `SphereCollider` | 구 형태. 빠름. | 낮음 |
| `CapsuleCollider` | 캡슐 형태. 캐릭터에 주로 사용. | 낮음 |
| `MeshCollider` | 메쉬 형태 그대로. 정확하지만 비쌈. | 높음 |

**복잡한 오브젝트**는 여러 개의 primitive Collider를 조합하는 것이 MeshCollider보다 성능상 유리하다.

---

## 3. Trigger vs Collision

| | Collision | Trigger |
|--|-----------|---------|
| **물리 반응** | 있음 (튕김, 마찰) | 없음 (통과) |
| **콜백** | `OnCollisionEnter/Stay/Exit` | `OnTriggerEnter/Stay/Exit` |
| **용도** | 물리적 충돌 | 범위 감지, 데미지 존 |
| **설정** | Is Trigger = false | Is Trigger = true |

```csharp
// Collision — 물리 반응 있음
private void OnCollisionEnter(Collision collision)
{
    if (collision.gameObject.CompareTag("Enemy"))
    {
        // 충돌 지점 및 법선벡터 접근 가능
        ContactPoint contact = collision.contacts[0];
        Debug.Log($"충돌 지점: {contact.point}, 법선: {contact.normal}");
    }
}

// Trigger — 물리 반응 없음, 통과
private void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Pickup"))
    {
        other.gameObject.SetActive(false);
    }
}
```

---

## 4. Layer & Collision Matrix

`Physics Settings → Layer Collision Matrix`에서 특정 레이어 간 충돌을 활성화/비활성화한다.

```csharp
// 코드에서 레이어 마스크로 특정 레이어만 감지
int layerMask = LayerMask.GetMask("Enemy", "Obstacle");
if (Physics.Raycast(transform.position, transform.forward, out RaycastHit hit, 10f, layerMask))
{
    Debug.Log($"감지된 오브젝트: {hit.collider.name}");
}
```

---

## 5. Kinematic Rigidbody 활용

`Is Kinematic = true`이면 물리 연산을 받지 않고, 코드로만 이동한다.  
XR 핸드 트래킹으로 오브젝트를 잡아 이동시킬 때 주로 사용.

```csharp
// 오브젝트 집기: Kinematic으로 전환하여 물리 끄기
public void Grab(Rigidbody target)
{
    target.isKinematic = true;
    target.transform.SetParent(this.transform);
}

// 놓기: Kinematic 해제하여 물리 복원
public void Release(Rigidbody target)
{
    target.transform.SetParent(null);
    target.isKinematic = false;
    target.velocity = _handVelocity;  // 손 속도 전달
}
```

---

## 6. 물리 시스템 주의사항

- 물리 관련 코드는 반드시 **`FixedUpdate`** 에서 처리한다.
- `Transform.position` 직접 수정은 Rigidbody와 충돌할 수 있다. `MovePosition()`을 사용.
- 빠른 발사체(총알 등)는 `Collision Detection → Continuous Dynamic` 또는 **Raycast** 방식으로 처리한다.
- `Physics.OverlapSphere`, `Physics.BoxCast` 등 비파괴적 물리 쿼리를 적극 활용한다.

```csharp
// 반경 내 모든 오브젝트 감지
Collider[] hits = Physics.OverlapSphere(transform.position, radius, layerMask);
foreach (var hit in hits)
{
    hit.GetComponent<IDamageable>()?.TakeDamage(damage);
}
```

---

[← 목차](./README.md)
