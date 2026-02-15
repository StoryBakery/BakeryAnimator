## 개요

**Inherits:** [BaseBakeryObject](./base-bakery-object.md)

애니메이션 전체를 총괄하는 루트 컨테이너입니다.
주로 `ModuleScript` 래퍼나 `Folder`로 존재합니다.

## Attributes

#### Version
`string`

파일 포맷의 시멘틱 버전(Semantic Version)입니다. (예: "1.0.0")
마이그레이션과 호환성 체크의 기준이 됩니다.

#### FrameRate
`number`

초당 프레임 수(FPS)입니다.

#### Looped
`boolean`

반복 재생 여부입니다.

#### Priority
`number`

애니메이션 우선순위입니다.
