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
|- Metadata: Folder
|  |- ModeMetadata: Folder
|  |  |- ...ModeId: Folder
```

`ModeMetadata`의 내부 스키마는 클래스 문서에서 고정하지 않습니다.
각 모드(`DefaultRoblox` 포함)가 자신의 문서에서 정의합니다.
