## 개요

`InterpolationMode`에 따른 계산 방식입니다.

## 모드

#### Constant
다음 키프레임에 도달하기 전까지 현재 값을 유지합니다.
$$ V(t) = P_0 $$

#### Linear
두 키프레임 사이를 선형으로 연결합니다.
$$ V(t) = P_0 + (P_1 - P_0) \times t $$

#### Bezier
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