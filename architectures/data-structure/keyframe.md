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

#### InterpolationMode
[`InterpolationMode`](./enums.md#interpolationmode)

키프레임 간의 보간 모드입니다.
*   `"Constant"`, `"Linear"`, `"Bezier"` 중 하나입니다.
*   `string`, `boolean` 등 불연속 타입은 `"Constant"`만 허용됩니다.

#### EasingStyle
[`EasingStyle`](./enums.md#easingstyle)

`InterpolationMode`가 `"Constant"`나 `"Linear"`가 아닐 때(예: Tween 보간) 사용되는 스타일입니다.
(`Bezier` 모드일 경우 핸들 값에 의해 곡선이 결정되므로 무시될 수 있습니다.)

#### EasingDirection
[`EasingDirection`](./enums.md#easingdirection)

`InterpolationMode`가 `"Constant"`나 `"Linear"`가 아닐 때 사용되는 방향입니다.

---

### Bezier Properties

`InterpolationMode`가 `"Bezier"`일 때 활성화되는 속성들입니다.
[`BezierInterpolator`](https://github.com/StoryBakery/BezierInterpolator) 로직을 따릅니다.

#### LeftHandleType
[`HandleType`](./enums.md#handletype)

좌측(들어오는) 핸들의 타입입니다. (`Automatic`, `Free` 등)

#### LeftHandleValue
`any`

좌측 핸들의 값입니다. (`HandleIn`)
타입에 따라 자동 계산될 수 있습니다.

#### RightHandleType
[`HandleType`](./enums.md#handletype)

우측(나가는) 핸들의 타입입니다. (`Automatic`, `Free` 등)

#### RightHandleValue
`any`

우측 핸들의 값입니다. (`HandleOut`)
타입에 따라 자동 계산될 수 있습니다.
