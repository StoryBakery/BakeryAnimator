---
title: Binding
---

특정 대상 인스턴스 하나를 식별하는 바인딩입니다.
런타임 `BindingId` 매핑의 기준이 됩니다.

## Attributes

### BindingId

`string`

바인딩의 고유 식별자입니다.

### TrackName

`string`

에디터 표시용 이름입니다.
형제 `Binding` 간에 유일해야 합니다.

### BindingGroupKey

`string?`

소속 바인딩 그룹 키입니다.
그룹 없이 루트 바인딩이면 `nil`입니다.

### ParentBindingId

`string?`

부모 바인딩 ID입니다.
계층 바인딩이 아닐 경우 `nil`입니다.

## Children

```lua
Binding
|- BindingChildren: Folder
|  |- ...Binding
|- Metadata: Folder
|  |- ModeMetadata: Folder
|  |  |- ...ModeId: Folder
```

`ModeMetadata`의 내부 스키마는 클래스 문서에서 고정하지 않습니다.
각 모드(`DefaultRoblox` 포함)가 자신의 문서에서 정의합니다.
