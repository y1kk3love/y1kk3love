# 6. 데이터 안정성 및 렌더링 최적화

---

## 데이터 검증 — ScriptableObject + OnValidate

### 타입 안전성 확보

`ScriptableObject`로 데이터 형태를 정의하면 에디터 내에서 구조화된 입력이 강제된다.

```csharp
[CreateAssetMenu(fileName = "NewItem", menuName = "Data/ItemData")]
public class ItemData : ScriptableObject
{
    [Range(1, 9999)] public int price;
    [Min(0)] public float weight;
    public Sprite icon;
}
```

### OnValidate — 에디터 저장 시 유효성 검사

에디터에서 값을 저장하는 순간 자동 호출되어 **잘못된 입력을 사전 차단**한다.

```csharp
[CreateAssetMenu(fileName = "NewEnemy", menuName = "Data/EnemyData")]
public class EnemyData : ScriptableObject
{
    public float speed = 3f;
    public int maxHP = 100;
    public float attackRange = 2f;

    private void OnValidate()
    {
        if (speed < 0)
        {
            speed = 0;
            Debug.LogWarning($"[{name}] speed는 0 이상이어야 합니다.");
        }

        if (maxHP <= 0)
        {
            maxHP = 1;
            Debug.LogWarning($"[{name}] maxHP는 1 이상이어야 합니다.");
        }

        if (attackRange <= 0)
        {
            attackRange = 0.1f;
            Debug.LogWarning($"[{name}] attackRange는 0보다 커야 합니다.");
        }
    }
}
```

---

## 렌더링 제어 — URP Scriptable Renderer Feature

URP의 `Scriptable Renderer Feature`로 **레이어 단위 렌더링 순서를 직접 제어**한다.

### 주요 활용 상황

| 문제 | 해결 |
|------|------|
| 3D 모델 Z-Sorting 오류 (장기, 혈관 등) | 레이어별 Render Objects로 순서 지정 + Depth Test 조정 |
| 특정 오브젝트만 별도 렌더 효과 적용 | Layer Mask 필터링 후 커스텀 Pass 적용 |
| 투명 오브젝트가 다른 투명체를 가림 | 레이어 분리 + 별도 Sort 우선순위 |

### 커스텀 Renderer Feature 예시

```csharp
public class OutlineRendererFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class Settings
    {
        public LayerMask targetLayer = -1;
        public Material outlineMaterial;
    }

    public Settings settings = new Settings();
    private OutlinePass _pass;

    public override void Create()
    {
        _pass = new OutlinePass(settings)
        {
            renderPassEvent = RenderPassEvent.AfterRenderingOpaques
        };
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData data)
    {
        renderer.EnqueuePass(_pass);
    }
}
```

---

## 모바일 XR 성능 최적화

모바일 VR(Quest, PICO)의 특성상 GPU 예산이 타이트하다.

### Dithered Transparency

알파 블렌딩 대신 **디더링(Dithering)**으로 투명 효과를 구현 → GPU 오버드로우 비용 절감.

```hlsl
// HLSL — 디더 행렬 기반 투명도 처리
float Dither(float2 uv, float alpha)
{
    float4x4 ditherMatrix = float4x4(
        1,  9,  3, 11,
        13,  5, 15,  7,
        4, 12,  2, 10,
        16,  8, 14,  6
    ) / 17.0;

    int2 pixelPos = int2(uv * _ScreenParams.xy) % 4;
    float threshold = ditherMatrix[pixelPos.x][pixelPos.y];

    clip(alpha - threshold); // threshold 미만이면 픽셀 버림
    return 1.0;
}
```

### 렌더 오버라이드 전략

```
중요 부위 (수술 시야 내 장기)   → 커스텀 Render Feature로 하이라이트
덜 중요한 배경 오브젝트         → 낮은 LOD + Dithered Transparency
먼 거리 오브젝트                → Culling 거리 조절 or Occlusion Culling 활용
```

---

[← 목차](./README.md)
