# XR Interaction Toolkit (XRI)

Unity의 공식 XR 인터랙션 프레임워크. VR/AR 환경에서 손 추적, 오브젝트 집기, UI 조작 등을 표준화된 방식으로 구현한다.

---

## 1. 아키텍처 개요

```
XR Origin (카메라 리그)
├── Camera Offset
│   ├── Main Camera
│   ├── Left Hand Controller  → XRController + XRDirectInteractor
│   └── Right Hand Controller → XRController + XRRayInteractor
│
Interactable Objects
└── XRGrabInteractable, XRSimpleInteractable, ...
```

**핵심 인터페이스**

| 인터페이스 | 역할 |
|-----------|------|
| `IXRInteractor` | 상호작용을 시도하는 주체 (손, 레이) |
| `IXRInteractable` | 상호작용 대상 오브젝트 |
| `IXRSelectInteractable` | 집기/선택 가능 |
| `IXRHoverInteractable` | 호버 감지 가능 |

---

## 2. Interactor 종류

### XRDirectInteractor — 직접 접촉

손이 오브젝트에 직접 닿았을 때 집는 방식.

```csharp
// 별도 코드 없이 컴포넌트만 추가하면 작동
// 필요시 이벤트 구독
var direct = GetComponent<XRDirectInteractor>();
direct.selectEntered.AddListener(OnGrabbed);
direct.selectExited.AddListener(OnReleased);

private void OnGrabbed(SelectEnterEventArgs args)
{
    Debug.Log($"집었다: {args.interactableObject}");
}
```

### XRRayInteractor — 레이캐스트 방식

멀리 있는 오브젝트를 레이로 가리켜 집거나 UI를 조작할 때 사용.

```csharp
var ray = GetComponent<XRRayInteractor>();

// 현재 레이가 가리키는 위치 가져오기
if (ray.TryGetCurrentRaycastHit(out RaycastHit hit))
{
    Debug.Log($"레이 히트: {hit.collider.name} @ {hit.point}");
}

// UI 포인터 모드 (GraphicRaycaster와 연동)
ray.useForUIInteraction = true;
```

### XRSocketInteractor — 소켓 배치

특정 위치에 오브젝트를 고정하는 소켓 (인벤토리 슬롯, 장비 착용 등).

```csharp
var socket = GetComponent<XRSocketInteractor>();

// 특정 태그만 허용
socket.socketActive = true;

// 코드로 오브젝트 강제 소켓에 끼우기
// → StartManualInteraction(interactable) 사용
```

---

## 3. XRGrabInteractable

오브젝트를 손으로 집을 수 있게 만드는 컴포넌트.

```csharp
public class MedicalTool : XRGrabInteractable
{
    protected override void OnSelectEntered(SelectEnterEventArgs args)
    {
        base.OnSelectEntered(args);
        // 집었을 때: 하이라이트, 사운드 등
        _highlightRenderer.enabled = true;
    }

    protected override void OnSelectExited(SelectExitEventArgs args)
    {
        base.OnSelectExited(args);
        // 놓았을 때
        _highlightRenderer.enabled = false;
    }

    protected override void OnHoverEntered(HoverEnterEventArgs args)
    {
        base.OnHoverEntered(args);
        // 손이 근처에 왔을 때
    }
}
```

### Attach Transform

오브젝트를 집었을 때 손에 붙는 위치/회전을 지정.

```csharp
// Inspector에서 attachTransform 지정 → 수술 도구의 올바른 그립 위치 정의
// 또는 코드로
grabInteractable.attachTransform = _gripPoint;
```

---

## 4. Interaction Layer Mask

어떤 Interactor가 어떤 Interactable과 상호작용할 수 있는지 레이어로 제어.

```csharp
// 특정 오브젝트를 오른손 레이로만 집을 수 있게 설정
grabInteractable.interactionLayers = InteractionLayerMask.GetMask("RightRay");

// 양손 직접 접촉만 허용
grabInteractable.interactionLayers = InteractionLayerMask.GetMask("DirectLeft", "DirectRight");
```

---

## 5. 손 추적 (Hand Tracking)

OpenXR 손 추적과 XRI를 연동하면 컨트롤러 없이 손 동작으로 인터랙션 가능.

```csharp
// XRHandSubsystem으로 관절 데이터 접근
using UnityEngine.XR.Hands;

public class HandPoseReader : MonoBehaviour
{
    private XRHandSubsystem _handSubsystem;

    void Start()
    {
        var subsystems = new List<XRHandSubsystem>();
        SubsystemManager.GetSubsystems(subsystems);
        if (subsystems.Count > 0) _handSubsystem = subsystems[0];
    }

    void Update()
    {
        if (_handSubsystem == null) return;

        XRHand hand = _handSubsystem.rightHand;
        if (hand.GetJoint(XRHandJointID.IndexTip, out XRHandJoint joint))
        {
            if (joint.TryGetPose(out Pose pose))
            {
                Debug.Log($"검지 끝 위치: {pose.position}");
            }
        }
    }
}
```

---

## 6. XR Interaction Manager

씬 내 모든 Interactor ↔ Interactable 연결을 중재하는 싱글턴.

```csharp
// 수동으로 Interactable 등록 / 해제
var manager = FindObjectOfType<XRInteractionManager>();
manager.RegisterInteractable(myInteractable);
manager.UnregisterInteractable(myInteractable);
```

> 씬당 1개만 배치. 없으면 XRI가 자동 생성하지만, 명시적으로 배치하는 것이 권장.

---

## 7. 주의사항

- `XRGrabInteractable`은 내부적으로 Rigidbody를 조작한다. 집는 동안 Kinematic 전환이 자동으로 이루어진다.
- VR 내 UI는 `World Space Canvas` + `TrackedDeviceGraphicRaycaster`를 사용한다.
- 성능: `XRRayInteractor`의 레이캐스트는 매 프레임 발생한다. 불필요한 레이어는 Layer Mask로 제외.

---

[← 목차](./README.md)
