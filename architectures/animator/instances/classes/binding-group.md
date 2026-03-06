---
title: BindingGroup
---

여러 `Binding`을 묶는 선택 계층입니다.
멀티 캐릭터/멀티 오브젝트 시퀀스에서 대상 범위를 분리할 때 사용합니다.

## Attributes

### BindingGroupKey

`string`

`LevelSequenceData.BindingGroupAnimationDatasByKey`와 매핑되는 그룹 키입니다.

### DisplayName

`string?`

에디터 표시용 이름입니다.

## Children

```lua
BindingGroup
|- BindingChildren: Folder
|  |- ...Binding
|- ...METADATA_<ADAPTER>_<PROP>: ObjectValue (optional, instance refs only)
```

어댑터 메타데이터의 기본 저장소는 `BindingGroup` 자체 `Attribute`입니다.
키 규칙은 `METADATA_<ADAPTER>_<PROP>`를 사용합니다.
`Instance` 참조만 예외적으로 동일 키 이름의 `ObjectValue`를 사용할 수 있습니다.
