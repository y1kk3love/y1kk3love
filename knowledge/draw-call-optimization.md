# 드로우 콜 최적화

드로우 콜(Draw Call)은 CPU가 GPU에 "이걸 그려라"고 보내는 명령 하나.  
VR에서는 양쪽 눈 각각 렌더링하므로 PC 대비 드로우 콜 비용이 2배. 72fps 유지를 위해 필수적으로 줄여야 한다.

---

## 1. 드로우 콜이란?

```
CPU: "이 메쉬를 이 머티리얼로 이 위치에 그려라" → GPU
```

오브젝트 1개 = 드로우 콜 최소 1회.  
머티리얼이 다르면 배칭이 불가능하여 각각 별도 드로우 콜이 발생한다.

---

## 2. Static Batching

이동하지 않는 오브젝트를 빌드 시 하나의 메쉬로 합쳐 드로우 콜을 줄인다.

- `Inspector → Static` 체크
- 같은 머티리얼을 공유하는 오브젝트끼리 자동 병합
- **단점:** 빌드 시 메모리 추가 사용, 런타임 이동 불가

---

## 3. Dynamic Batching

런타임에 작은 메쉬(보통 버텍스 수 300 이하)를 자동으로 합쳐 렌더링한다.

- `Project Settings → Graphics → Dynamic Batching` 활성화
- 같은 머티리얼, 동일 스케일 등 조건이 까다로워 실제 효과는 제한적

---

## 4. GPU Instancing

동일한 메쉬 + 머티리얼의 오브젝트를 하나의 드로우 콜로 대량 렌더링한다.

```csharp
// 머티리얼에서 GPU Instancing 활성화 후, 각 인스턴스 별 데이터 설정
private MaterialPropertyBlock _mpb;

void Start()
{
    _mpb = new MaterialPropertyBlock();
}

void Render(Renderer rend, Color color)
{
    _mpb.SetColor("_Color", color);
    rend.SetPropertyBlock(_mpb); // 같은 머티리얼이어도 개별 색상 적용 가능
}
```

```csharp
// Graphics.DrawMeshInstanced — 렌더러 컴포넌트 없이 대량 렌더링
void RenderBoids(List<Matrix4x4> matrices)
{
    // 한 번의 드로우 콜로 최대 1023개 렌더링
    Graphics.DrawMeshInstanced(_mesh, 0, _material, matrices);
}
```

**적합한 상황:** 군집 AI(Boids), 나무/풀 등 동일한 오브젝트 대량 배치

---

## 5. Occlusion Culling

카메라에 보이지 않는 오브젝트를 렌더링하지 않는다.

1. `Window → Rendering → Occlusion Culling` 베이크
2. 오브젝트에 `Occluder Static` / `Occludee Static` 설정
3. 런타임에 가려진 오브젝트 자동 컬링

**VR에서 주의:** 카메라가 2개(양쪽 눈)이므로 Occlusion Culling 효율이 떨어질 수 있다.

---

## 6. LOD (Level of Detail)

거리에 따라 메쉬 복잡도를 낮춰 GPU 부하를 줄인다.

```
거리 0 ~ 10m  → LOD 0 (고해상도)
거리 10 ~ 30m → LOD 1 (중해상도)
거리 30m+     → LOD 2 (저해상도) or Culled (렌더링 제거)
```

`LOD Group` 컴포넌트로 설정. VR 환경에서는 거리가 상대적으로 가까우므로 임계값을 조정해야 한다.

---

## 7. Atlas Texturing

여러 오브젝트의 텍스처를 하나의 큰 텍스처로 합쳐 머티리얼 수를 줄인다.

```
오브젝트 A (텍스처A) + 오브젝트 B (텍스처B) + 오브젝트 C (텍스처C)
→ 하나의 Atlas 텍스처를 공유 → 머티리얼 1개 → 배칭 가능
```

Unity의 `Sprite Atlas`(UI), 또는 수동 UV 재배치로 구현.

---

## 8. Profiler로 확인

```
Window → Analysis → Profiler → Rendering 탭
→ Batches, Draw Calls, Tris, Verts 수치 확인
```

| 수치 | Quest 2 권장 |
|------|-------------|
| Draw Calls | 100 이하 |
| Triangles | 750,000 이하 |
| Texture Memory | 제한적 (공유 텍스처 필수) |

---

[← 목차](./README.md)
