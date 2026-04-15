# 4. 데이터 직렬화 및 네트워크 최적화 (Delta Encoding)

---

## Binary Serialization

데이터를 0과 1의 이진 형태로 저장하는 방식.

| | JSON / XML | Binary |
|--|-----------|--------|
| **용량** | 크다 (텍스트 기반) | 작다 |
| **파싱 속도** | 느리다 | 빠르다 |
| **가독성** | 높다 | 낮다 (디버깅 어려움) |
| **적합한 상황** | 설정 파일, 외부 API | 실시간 네트워크, 저장 파일 |

```csharp
// BinaryWriter로 Vector3 직렬화
void WriteTransform(BinaryWriter writer, Transform t)
{
    writer.Write(t.position.x);
    writer.Write(t.position.y);
    writer.Write(t.position.z);
}

// JSON 대비 절반 이하의 바이트 사용
```

---

## Delta Encoding

전체 데이터가 아닌 **이전 상태와의 차잇값(Delta)**만 기록하는 방식.

### 왜 쓰는가?

- 네트워크 패킷 크기를 대폭 줄인다.
- 대부분의 프레임에서 변화량은 전체보다 훨씬 작다.

```csharp
struct TransformSnapshot
{
    public Vector3 Position;
    public Quaternion Rotation;
}

byte[] EncodeDelta(TransformSnapshot prev, TransformSnapshot curr)
{
    var delta = new TransformSnapshot
    {
        Position = curr.Position - prev.Position,
        Rotation = Quaternion.Inverse(prev.Rotation) * curr.Rotation
    };

    using var ms = new MemoryStream();
    using var bw = new BinaryWriter(ms);

    // 위치: 변화량이 작으면 short(2byte)로 압축
    bw.Write((short)(delta.Position.x * 1000));
    bw.Write((short)(delta.Position.y * 1000));
    bw.Write((short)(delta.Position.z * 1000));

    return ms.ToArray();
}
```

---

## Keyframe (I-Frame)

Delta 방식의 단점: 오차 누적 + 중간 탐색 불가.

**해결:** 일정 주기마다 **전체 상태(Keyframe)**를 삽입한다.

```
Frame: 1    2    3    4    5    6    7    8    9    10   11 ...
       [KF] [Δ]  [Δ]  [Δ]  [Δ]  [KF] [Δ]  [Δ]  [Δ]  [Δ] [KF]
```

- KF에서 중간 탐색 가능, 오차 초기화.
- 재생 시 가장 가까운 KF → 이후 델타를 순서대로 적용.

---

## 실무 적용 — TwinWorld

### 누적 오차 해결

```csharp
private int _frameCount = 0;
private const int KeyframeInterval = 30; // 30프레임마다 KF 삽입

void SendNetworkUpdate()
{
    _frameCount++;

    if (_frameCount % KeyframeInterval == 0)
    {
        SendKeyframe(_currentState); // 전체 상태 전송
    }
    else
    {
        SendDelta(_prevState, _currentState); // 차잇값만 전송
    }

    _prevState = _currentState;
}
```

### 회전값 압축 (바이트 최적화)

```csharp
// Quaternion을 그대로 보내면 float 4개 = 16바이트
// 정밀도를 줄여서 short 3개 = 6바이트로 압축
void WriteCompressedRotation(BinaryWriter bw, Quaternion q)
{
    // 가장 큰 성분을 제외하고 나머지 3개만 저장 (Smallest Three 기법)
    bw.Write((short)(q.x * 32767));
    bw.Write((short)(q.y * 32767));
    bw.Write((short)(q.z * 32767));
    // w는 수신 측에서 x²+y²+z²+w²=1 로 복원
}
```

---

[← 목차](./README.md)
