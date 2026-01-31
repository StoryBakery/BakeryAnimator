# Keyframe

**Inherits:** [Node](./node.md)

특정 프레임에서의 값과 보간 정보를 담는 최소 단위입니다.
**Data Point** 역할을 수행합니다.

## Properties

#### Frame
`number`

프레임 위치입니다.

#### Value
`any`

해당 프레임의 값입니다.

#### EasingStyle
[`EasingStyle`](./enums.md#easingstyle)

보간 스타일입니다.
기본 `Enum.EasingStyle` 대신 커스텀 문자열 Enum을 사용합니다.

#### EasingDirection
[`EasingDirection`](./enums.md#easingdirection)

보간 방향입니다.
기본 `Enum.EasingDirection` 대신 커스텀 문자열 Enum을 사용합니다.
