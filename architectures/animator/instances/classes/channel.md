## 개요

Inherits:
[BaseBakeryObject](./base-bakery-object.md)

특정 속성(Property) 하나의 시간 흐름에 따른 변화를 정의합니다.
**Value Timeline** 역할을 수행합니다.

> **Note:** [Interpolation Strategy](../runtime/interpolation.md)에 따라, 복합 타입(`Vector3`, `CFrame` 등)은 내부적으로 여러 개의 **스칼라 채널(Scalar Channel)**로 분해되어 관리될 수 있습니다. (예: `Position.X`, `Position.Y`, `Position.Z`)

## Attributes

#### PropertyName
`string`

조작할 속성 이름입니다. (예: "Position")

#### ValueType
`string`

값의 타입입니다. (예: "Vector3", "CFrame")
