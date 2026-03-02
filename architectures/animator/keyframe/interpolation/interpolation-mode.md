---
title: Interpolation Mode
---

`InterpolationMode`에 따른 계산 방식입니다.

## 모드 종류

### Bezier

`@default`

베지어 곡선 보간을 수행합니다. (핸들 속성 필요)

[BezierInterpolator](https://github.com/StoryBakery/BezierInterpolator)를 사용합니다.
4개의 점(시작점, 시작핸들, 끝핸들, 끝점)을 사용해 곡선을 계산합니다.

$$ V(t) = (1-t)^3 P_0 + 3(1-t)^2 t P_1 + 3(1-t) t^2 P_2 + t^3 P_3 $$

- $P_0$: 시작 키프레임 값
- $P_1$: 시작 키프레임의 `RightHandleValue` (HandleOut)
- $P_2$: 끝 키프레임의 `LeftHandleValue` (HandleIn)
- $P_3$: 끝 키프레임 값

> **Note**: 핸들 값은 "상대 좌표"가 아닌 **"절대 좌표"**(Value와 동일한 스케일)로
> 저장하는 것이 데이터 관리에 더 용이합니다.

### Constant

다음 키프레임에 도달하기 전까지 현재 값을 유지합니다.

- InterpolationLineColor: "#4DA3FF"

### Easing

EasingStyle 과 EasingDirection 을 사용해 값을 보간합니다.

- InterpolationLineColor: [easing](./easing/index.md) 참고

## Custom Interpolation Line

베지어 키프레임이 아닌 키프레임 이후에는 다음 프레임까지 특정한 색상의 얇은 선이 이어지게 됩니다.
