## 개요

Easing 에 대해 다루는 설계도입니다.

## 내부 계산

`MathUtil` 라이브러리의 `Easing` 함수와 `LinearInterpolation`을 사용합니다. 
로블록스 엔진의 `TweenService`에 의존하지 않습니다, 느리기 때문.

`MathUtil.easingsByStyleAndDirection[style][direction]` 으로 스타일(`"Cubic"`, `"Elastic"` 등)과 방향(`"In"`, `"Out"` 등)
에 맞는 함수를 가져옵니다.

## EasingStyle

#### Linear
`@default`

- InterpolationLineColor: "#637264ff"

#### Sine

- InterpolationLineColor: "#3aff3dff"

#### Back

- InterpolationLineColor: "#6C5CE7"

#### Quad

- InterpolationLineColor: "#00B894"

#### Quart

- InterpolationLineColor: "#FAB1A0"

#### Quint

- InterpolationLineColor: "#FF7675"

#### Bounce

- InterpolationLineColor: "#FDCB6E"

#### Elastic

- InterpolationLineColor: "#FD79A8"

#### Exponential

- InterpolationLineColor: "#D63031"

#### Circular

- InterpolationLineColor: "#74B9FF"

#### Cubic

- InterpolationLineColor: "#00CEC9"


## EasingDirection

보간의 적용 방향을 정의합니다.

#### In

#### Out

#### InOut

#### OutIn



