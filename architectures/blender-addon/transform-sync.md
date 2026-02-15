## 개요

블렌더 애드온과 로블록스 플러그인 간의 동기화 및 데이터 전송 설계입니다.


## 기능

### 실시간-동기화
*   **Bone Transformation:** Blender Bone (Local) ↔ Roblox Motor6D/Bone Transform
*   **Object Property:** Scale(Size), Location/Rotation(CFrame)
*   **Websocket:** 실시간 프리뷰를 위한 WebSocket 통신 지원

### 데이터-포맷
*   **Raw Format:** `.sbbtra` (Story Bakery Blender To Roblox Animation)
*   **Optimization:** FBX/glTF 변환 생략, Blender API 직접 추출.
*   **Compression:** 
    *   매 프레임 캡처 (Sampling).
    *   변화 없는 트랙(Constant/Epsilon) 생략.
    *   중립 포즈 기준 Local Delta 저장.

## Addon-Interface (Blender)

*   Target Armature/Object/Camera 선택 기능.
*   실시간 레코딩 및 전송 토글.

## 좌표계

로블록스와 블렌더의 좌표계 차이를 자동 보정합니다.

*   **Roblox:** +X(Right), +Y(Up), -Z(Forward)
*   **Blender:** +X(Right), +Y(Forward), +Z(Up)
*   **Conversion:** `Roblox(x, y, z) = (Blender.x, Blender.z, -Blender.y)`

## Motor6D-조작

### Motor6D-생성-규칙

블렌더에서의 `Amature` 모디파이어가 있는 메쉬 오브젝트의 이름과 `Amature` 의 `Bone` 이름 일치할 경우 `Motor6D`로 변환합니다.
일치하지 않을 경우 `Bone` 인스턴스로 변환됩니다.


### 회전-보정

로블록스에서 `glTF` 든 `fbx` 든 `Motor6D` 가 파트와 연결될 때 초기 회전(Rest Rotation)이 (0,0,0)으로 리셋되는 특성이 있습니다.


그래서 애니메이션 에디터에서 fromFbx 로 애니메이션을 로드할 때, 

따라서 블렌더에서 애니메이션을 가져올 때 다음 과정이 필요합니다:

1.  **Neutral Pose Correction:** 블렌더 Bone의 월드 회전 축을 로블록스 Motor6D C0/C1 기준에 맞춰 보정.
2.  **Transform Calculation:** `m.Transform = (Part0.CFrame * C0):Inverse() * (Part1.CFrame * m.C1)` 공식을 역이용하여 정확한 포즈 적용.
