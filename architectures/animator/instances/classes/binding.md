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

## PreferredChildren

- `Marker` (local)
- `PropertyTrack`
- `EventTrack`
- `AudioTrack`
- `ConstraintTrack`

## Children

```lua
Binding
|- ...Marker
|- ...PropertyTrack
|- ...EventTrack
|- ...AudioTrack
|- ...ConstraintTrack
|- BindingChildren: Folder
|  |- ...Binding
|- ...METADATA_<ADAPTER>_<PROP>: ObjectValue (optional, instance refs only)
```

어댑터 메타데이터의 기본 저장소는 `Binding` 자체 `Attribute`입니다.
키 규칙은 `METADATA_<ADAPTER>_<PROP>`를 사용합니다.
`Instance` 참조만 예외적으로 동일 키 이름의 `ObjectValue`를 사용할 수 있습니다.
