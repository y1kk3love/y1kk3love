# 2. GPU 연산 최적화: Compute Shader

---

## Compute Shader란?

그래픽 파이프라인과 별개로 **GPU의 병렬 연산 능력**을 일반 목적(GPGPU)으로 활용하는 프로그램.  
수천 개의 스레드가 동시에 실행되므로, 독립적으로 반복되는 대규모 연산에 적합하다.

**적합한 사례:**
- 텍스처 클리닝 / 프로시저럴 텍스처 생성
- 군집 AI 알고리즘 (Boids)
- 파티클 시뮬레이션
- 충돌 전처리, 공간 분할

---

## CPU ↔ GPU 병목 (Bottleneck)

| 방향 | 설명 | 위험도 |
|------|------|--------|
| **Write (CPU → GPU)** | CPU에서 GPU로 데이터를 전송 | 낮음 |
| **Read / Readback (GPU → CPU)** | GPU 연산 결과를 CPU로 가져올 때 메인 스레드 **Stall** 발생 | 매우 높음 |

Readback은 GPU 연산이 끝날 때까지 CPU가 대기해야 하므로 **프레임 드랍의 직접 원인**이 된다.

---

## 핵심 전략

### 1. Readback 자체를 제거

GPU 결과를 CPU로 가져오지 않고 **RenderTexture에 바로 기록** → 셰이더에서 직접 참조.

```csharp
// Compute Shader에서 RenderTexture에 직접 쓰기
[SerializeField] private ComputeShader _cs;
[SerializeField] private RenderTexture _rt;

void Start()
{
    _rt = new RenderTexture(512, 512, 0, RenderTextureFormat.ARGBFloat);
    _rt.enableRandomWrite = true;
    _rt.Create();

    int kernel = _cs.FindKernel("CSMain");
    _cs.SetTexture(kernel, "Result", _rt);
    _cs.Dispatch(kernel, 512 / 8, 512 / 8, 1);  // (512/8)² 개의 스레드 그룹
}
```

```hlsl
// Compute Shader (CSMain.compute)
#pragma kernel CSMain
RWTexture2D<float4> Result;

[numthreads(8, 8, 1)]
void CSMain(uint3 id : SV_DispatchThreadID)
{
    float2 uv = float2(id.xy) / 512.0;
    Result[id.xy] = float4(uv, 0, 1);
}
```

### 2. Readback이 필요할 때 — AsyncGPUReadback

메인 스레드를 막지 않고 비동기로 GPU 데이터를 읽어온다.

```csharp
void RequestReadback()
{
    AsyncGPUReadback.Request(_rt, 0, TextureFormat.RGBAFloat, OnReadbackComplete);
}

void OnReadbackComplete(AsyncGPUReadbackRequest request)
{
    if (request.hasError) return;

    var data = request.GetData<Color32>();
    // 결과 처리 — 메인 스레드 Stall 없음
}
```

---

## ComputeBuffer — 구조체 데이터 처리

```csharp
struct BoidData { public Vector3 position; public Vector3 velocity; }

ComputeBuffer _buffer;

void Init(int count)
{
    _buffer = new ComputeBuffer(count, sizeof(float) * 6); // Vector3 * 2
    _cs.SetBuffer(kernel, "Boids", _buffer);
    _cs.Dispatch(kernel, count / 64, 1, 1);
}

void OnDestroy() => _buffer?.Release(); // 반드시 해제
```

---

[← 목차](./README.md)
