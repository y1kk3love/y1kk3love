# HLSL 셰이더 프로그래밍 기초

GPU에서 실행되는 프로그램. 정점 변환(Vertex Shader)과 픽셀 색상 결정(Fragment Shader)으로 렌더링 결과를 직접 제어한다. Unity URP 기준.

---

## 1. 셰이더 파이프라인

```
CPU (Unity)
  ↓ 메쉬 + 머티리얼 + 변환 행렬
Vertex Shader   → 각 정점의 클립 공간 위치 계산
  ↓ 보간(Interpolation)
Fragment Shader → 각 픽셀의 최종 색상 결정
  ↓
화면 출력
```

---

## 2. URP 기본 셰이더 구조

```hlsl
Shader "Custom/BasicUnlit"
{
    Properties
    {
        _BaseColor ("Color", Color) = (1,1,1,1)
        _BaseMap ("Texture", 2D) = "white" {}
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }

        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            CBUFFER_START(UnityPerMaterial)
                float4 _BaseColor;
                float4 _BaseMap_ST;
            CBUFFER_END

            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);

            struct Attributes  // CPU → Vertex Shader 입력
            {
                float4 positionOS : POSITION;
                float2 uv         : TEXCOORD0;
            };

            struct Varyings   // Vertex → Fragment 출력
            {
                float4 positionHCS : SV_POSITION;
                float2 uv          : TEXCOORD0;
            };

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = TRANSFORM_TEX(IN.uv, _BaseMap);
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                half4 tex = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);
                return tex * _BaseColor;
            }

            ENDHLSL
        }
    }
}
```

---

## 3. 좌표 공간

| 공간 | 약어 | 설명 |
|------|------|------|
| Object Space | OS | 메쉬 로컬 원점 기준 |
| World Space | WS | 씬 월드 원점 기준 |
| View Space | VS | 카메라 기준 |
| Clip Space | HCS | GPU 클리핑용, w로 나누면 NDC |

```hlsl
// URP 변환 함수
float3 worldPos = TransformObjectToWorld(IN.positionOS.xyz);
float4 clipPos  = TransformWorldToHClip(worldPos);

// 노멀 변환 (역전치 행렬 사용)
float3 worldNormal = TransformObjectToWorldNormal(IN.normalOS);
```

---

## 4. 라이팅 — Lambert 디퓨즈

```hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

half4 frag(Varyings IN) : SV_Target
{
    // 메인 라이트 방향
    Light mainLight = GetMainLight();
    float3 lightDir = mainLight.direction;

    // Lambert: 법선과 빛의 내적
    float NdotL = saturate(dot(IN.worldNormal, lightDir));

    half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv) * _BaseColor;
    return half4(color.rgb * NdotL * mainLight.color, color.a);
}
```

---

## 5. 투명 셰이더 — 의료 시각화 응용

```hlsl
// 반투명: SubShader Tags에 "Queue"="Transparent" 추가
Tags { "RenderType"="Transparent" "Queue"="Transparent" }

// Blend 설정 (Pass 안에)
Blend SrcAlpha OneMinusSrcAlpha
ZWrite Off

half4 frag(Varyings IN) : SV_Target
{
    half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv) * _BaseColor;

    // 뷰 방향에 따른 림(Rim) 투명도 — 臟器 단면 강조에 활용
    float3 viewDir = normalize(GetWorldSpaceViewDir(IN.worldPos));
    float rim = 1.0 - saturate(dot(viewDir, IN.worldNormal));
    color.a *= rim * _AlphaStrength;

    return color;
}
```

---

## 6. Shader Graph vs 수동 HLSL

| | Shader Graph | 수동 HLSL |
|--|-------------|----------|
| 진입 장벽 | 낮음 (노드 기반) | 높음 |
| 유연성 | 제한적 | 무제한 |
| 성능 튜닝 | 어려움 | 직접 최적화 가능 |
| 재사용 | Sub-Graph | Include 파일 |
| 적합 상황 | 프로토타입, 아티스트 협업 | 특수 효과, 성능 임계 |

```hlsl
// 수동 HLSL Custom Function 노드로 Shader Graph와 혼용 가능
void CustomRim_float(float3 Normal, float3 ViewDir, float Power, out float Rim)
{
    Rim = pow(1.0 - saturate(dot(Normal, ViewDir)), Power);
}
```

---

## 7. 최적화 포인트

- `half` 정밀도(16-bit)를 가능한 한 사용한다. 모바일/VR GPU에서 `float`(32-bit) 대비 빠름.
- 분기(`if`)는 GPU에서 비싸다. `step()`, `saturate()`, `lerp()`로 대체.
- 텍스처 샘플링은 최소화하고, 여러 채널을 하나의 텍스처에 패킹(R=AO, G=Roughness, B=Metallic).
- `discard`(알파 클리핑)는 Early-Z를 깬다. 반드시 필요할 때만 사용.

---

[← 목차](./README.md)
