## 개요

애니메이션의 품질과 에디터의 사용성을 결정짓는 **보간(Interpolation) 전략** 상세 설계입니다.
블렌더의 F-Curve 시스템과 로블록스 엔진의 특성을 고려하여, **스칼라 분해(Scalar Decomposition)** 방식을 채택합니다.

#### 왜 분해하는가?
Vector3 를 한 번에 조작하는 커브를 제공하는 것보다,
각 채널 x y z 별로 키프레임을 넣을 수도 있고, 커브를 바꿀 수 있는 것이 더 유용하기 때문입니다.
그러나 x y z 를 한 번에 기록하는 것이 편의상 기본.

*   **정밀한 제어:** `Vector3`의 X축은 Linear로, Y축은 Bezier로 움직여야 할 때가 많습니다. 통짜 데이터로는 이를 제어할 수 없습니다.
*   **베지어 수학:** 베지어 곡선 공식은 기본적으로 1차원 값에 대해 작동합니다.
*   **블렌더 호환성:** 블렌더는 Transform 데이터를 
    *   `Location X, Y, Z`
    *   `Euler Rotation X, Y, Z`
    *   `Quaternion W X Y Z`
    등 개별 F-Curve로 관리합니다.

## 타입마다의-스칼라-분해

각 로블록스 타입이 어떤 스칼라 채널들로 분해되는지 정의합니다.

#### 1개-채널
*   `number`: 그대로 1개의 채널 (`Value`).
*   `bool`: 1개의 채널 (`Value`). (`Constant` 보간만 허용)

#### Vector Types
*   `Vector2`: `X`, `Y` (2 Channels)
*   `Vector3`: `X`, `Y`, `Z` (3 Channels)
*   `UDim2`: `X.Scale`, `X.Offset`, `Y.Scale`, `Y.Offset` (4 Channels)

#### Color Types
*   `Color3`: `R`, `G`, `B` (3 Channels)
    *   색상값은 0~1 범위 내에서 보간되거나, 0~255 범위로 에디터에 표시될 수 있습니다.

#### Rotation & CFrame (Critical)
`CFrame`은 위치(Position)와 회전(Rotation)으로 나뉩니다.

*   Position: `X`, `Y`, `Z` (Vector3와 동일)
*   Rotation: Euler Angles (`X`, `Y`, `Z`) 사용 혹은 쿼터니언 사용.

---

## Interpolation Logic per Mode

`InterpolationMode`에 따른 계산 방식입니다.

#### Constant (Step)
다음 키프레임에 도달하기 전까지 현재 값을 유지합니다.
$$ V(t) = P_0 $$

#### Linear
두 키프레임 사이를 선형으로 연결합니다.
$$ V(t) = P_0 + (P_1 - P_0) \times t $$

#### Bezier (Cubic)
[BezierInterpolator](https://github.com/StoryBakery/BezierInterpolator)를 사용합니다.
4개의 점(시작점, 시작핸들, 끝핸들, 끝점)을 사용해 곡선을 계산합니다.

$$ V(t) = (1-t)^3 P_0 + 3(1-t)^2 t P_1 + 3(1-t) t^2 P_2 + t^3 P_3 $$

*   $P_0$: 시작 키프레임 값
*   $P_1$: 시작 키프레임의 `RightHandleValue` (HandleOut)
*   $P_2$: 끝 키프레임의 `LeftHandleValue` (HandleIn)
*   $P_3$: 끝 키프레임 값

> **Note:** 핸들 값은 "상대 좌표"가 아닌 **"절대 좌표"**(Value와 동일한 스케일)로 저장하는 것이 데이터 관리에 더 용이합니다.

---

## Easing-Support
`Constant`나 `Linear` 모드가 아닌데 `Bezier` 핸들이 없는 경우(또는 명시적으로 Easing을 원하는 경우), **`MathUtil`** 라이브러리의 Easing 함수와 `LinearInterpolation`을 사용합니다. 
로블록스 엔진의 `TweenService`에 의존하지 않습니다, 느리기 때문.

*   **Logic:**
    1.  `MathUtil.easingsByStyleAndDirection`에서 스타일(`"Cubic"`, `"Elastic"` 등)과 방향(`"In"`, `"Out"` 등)에 맞는 함수를 가져옵니다.
    2.  진행도 `t`를 변형시킵니다: `alpha = easingFunction(t)`
    3.  `LinearInterpolation`으로 값을 보간합니다.

$$ V(t) = LinearInterpolation(P_0, P_1, alpha) $$
