# Interpolation Strategy

애니메이션의 품질과 에디터의 사용성을 결정짓는 **보간(Interpolation) 전략** 상세 설계입니다.
블렌더(Blender)의 F-Curve 시스템과 로블록스 엔진의 특성을 고려하여, **스칼라 분해(Scalar Decomposition)** 방식을 채택합니다.

## 1. Core Philosophy: Scalar Decomposition

블렌더와 같은 정밀한 그래프 편집기(Graph Editor)를 구현하기 위해, 모든 복합 데이터 타입(`Vector3`, `CFrame`, `Color3`)은 **단일 숫자(Scalar, `number`) 채널**들로 분해되어 관리됩니다.

### 왜 분해하는가?
*   **정밀한 제어:** `Vector3`의 X축은 Linear로, Y축은 Bezier로 움직여야 할 때가 많습니다. 통짜 데이터로는 이를 제어할 수 없습니다.
*   **베지어 수학:** 베지어 곡선 공식은 기본적으로 1차원 값에 대해 작동합니다.
*   **블렌더 호환성:** 블렌더는 Transform 데이터를 `Location X, Y, Z`, `Euler Rotation X, Y, Z` 등 개별 F-Curve로 관리합니다.

---

## 2. Type Decomposition Rules

각 로블록스 타입이 어떤 스칼라 채널들로 분해되는지 정의합니다.

### 2.1. Basic Types
*   **`number`**: 그대로 1개의 채널 (`Value`).
*   **`bool`**: 1개의 채널 (`Value`). (`Constant` 보간만 허용)

### 2.2. Vector Types
*   **`Vector2`**: `X`, `Y` (2 Channels)
*   **`Vector3`**: `X`, `Y`, `Z` (3 Channels)
*   **`UDim2`**: `X.Scale`, `X.Offset`, `Y.Scale`, `Y.Offset` (4 Channels)

### 2.3. Color Types
*   **`Color3`**: `R`, `G`, `B` (3 Channels)
    *   색상값은 0~1 범위 내에서 보간되거나, 0~255 범위로 에디터에 표시될 수 있습니다.

### 2.4. Rotation & CFrame (Critical)
`CFrame`은 위치(Position)와 회전(Rotation)으로 나뉩니다.

*   **Position**: `X`, `Y`, `Z` (Vector3와 동일)
*   **Rotation**: **Euler Angles (`X`, `Y`, `Z`)** 사용.
    *   **Rationale:** 사용자는 쿼터니언(Quaternion, 4D)을 직관적으로 이해하거나 그래프로 수정하기 어렵습니다. 따라서 에디팅 단계에서는 **오일러 각도(Euler Angles)**를 사용합니다.
    *   **Gimbal Lock:** 오일러 각도의 한계인 짐벌 락은 존재하지만, 그래프 에디터의 직관성을 위해 업계 표준(Blender, Maya 등)도 기본적으로 오일러를 사용합니다.
    *   **Runtime:** 재생 시에는 Euler Angles(`CFrame.fromEulerAnglesYXZ` 등)를 통해 최종 CFrame을 합성합니다.

---

## 3. Interpolation Logic per Mode

`InterpolationMode`에 따른 계산 방식입니다.

### 3.1. Constant (Step)
다음 키프레임에 도달하기 전까지 현재 값을 유지합니다.
$$ V(t) = P_0 $$

### 3.2. Linear
두 키프레임 사이를 선형으로 연결합니다.
$$ V(t) = P_0 + (P_1 - P_0) \times t $$

### 3.3. Bezier (Cubic)
[BezierInterpolator](https://github.com/StoryBakery/BezierInterpolator)를 사용합니다.
4개의 점(시작점, 시작핸들, 끝핸들, 끝점)을 사용해 곡선을 계산합니다.

$$ V(t) = (1-t)^3 P_0 + 3(1-t)^2 t P_1 + 3(1-t) t^2 P_2 + t^3 P_3 $$

*   $P_0$: 시작 키프레임 값
*   $P_1$: 시작 키프레임의 `RightHandleValue` (HandleOut)
*   $P_2$: 끝 키프레임의 `LeftHandleValue` (HandleIn)
*   $P_3$: 끝 키프레임 값

> **Note:** 핸들 값은 "상대 좌표"가 아닌 **"절대 좌표"**(Value와 동일한 스케일)로 저장하는 것이 데이터 관리에 더 용이합니다.

---

## 4. Easing Support (MathUtil)
`Constant`나 `Linear` 모드가 아닌데 `Bezier` 핸들이 없는 경우(또는 명시적으로 Easing을 원하는 경우), **`MathUtil`** 라이브러리의 Easing 함수와 `LinearInterpolation`을 사용합니다. 로블록스 엔진의 `TweenService`에 의존하지 않습니다.

*   **Logic:**
    1.  `MathUtil.easingsByStyleAndDirection`에서 스타일(`"Cubic"`, `"Elastic"` 등)과 방향(`"In"`, `"Out"` 등)에 맞는 함수를 가져옵니다.
    2.  진행도 `t`를 변형시킵니다: `alpha = easingFunction(t)`
    3.  `LinearInterpolation`으로 값을 보간합니다.

$$ V(t) = LinearInterpolation(P_0, P_1, alpha) $$

*   **Reference:**
    *   [MathUtil](https://github.com/StoryBakery/MathUtil)
    *   [LinearInterpolation](https://github.com/StoryBakery/LinearInterpolation)
