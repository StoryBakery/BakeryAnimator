## 개요

**Inherits:** [BaseBakeryObject](./base-bakery-object.md)

특정 인스턴스(Part, Model) 하나에 대응되는 트랙입니다.
**Object Identity** 역할을 수행합니다.

## Properties

#### TrackName
`string`

매핑 키 이름입니다. 형제 `Track` 간에 유일해야 합니다.

#### InstanceRef
`ObjectValue`

에디터 상에서 참조 중인 실제 인스턴스입니다. (런타임에는 무시될 수 있음)
