---
title: Animator Instances
---

애니메이터는 애니메이션 파일의 데이터를 `Instance`를 진실의 원천으로 둡니다.
모든 데이터와 UI 상태는 `:GetAttributeChangedSignal`과 `:GetAttribute`로 감지하며,
`Observe` 패턴을 사용합니다.

## 이유

- 인스턴스를 기준으로 하면 `ChangeHistoryService`를 통해 로블록스 스튜디오 수정
  모드에서 `Ctrl + Z`와 `Ctrl + Shift + Z`를 쉽게 구현할 수 있습니다.
- 제3자 에디터를 만들거나 다른 플러그인에서 값을 수정해도, 인스턴스가 진실의 원천이므로 데이터 일관성을 유지할 수 있습니다.

## 권장 계층

현재 시스템의 인스턴스 계층은 다음 구조를 권장합니다.

```text
AnimationFile
|- Marker
|- BindingGroup? (optional)
|  |- Binding
|     |- PropertyTrack
|     |  |- Section
|     |     |- Channel
|     |        |- Keyframe
|     |- EventTrack
|     |  |- Section
|     |     |- EventKey
|     |- ConstraintTrack
|        |- Section

AnimationFile
|- Marker
|- Binding
|  |- PropertyTrack
|  |  |- Section
|  |     |- Channel
|  |        |- Keyframe
|  |- EventTrack
|  |  |- Section
|  |     |- EventKey
|  |- ConstraintTrack
|     |- Section
```

`BindingGroup`은 선택 계층이며, 그룹 없이 `Binding`을 루트에 둘 수 있습니다.
`Marker`는 블렌더 호환을 위해 파일 레벨 기준으로 관리합니다.

## 클래스 문서

- [AnimationFile](./classes/animation-file.md)
- [BindingGroup](./classes/binding-group.md)
- [Binding](./classes/binding.md)
- [Marker](./classes/marker.md)
- [PropertyTrack](./classes/property-track.md)
- [Section](./classes/section.md)
- [Channel](./classes/channel.md)
- [Keyframe](./classes/keyframe.md)
- [EventTrack](./classes/event-track.md)
- [EventKey](./classes/event-key.md)
- [ConstraintTrack](./classes/constraint-track.md)

## 파라미터 정책

파라미터는 Roblox 방식에 맞춰 `AnimationTrack` 런타임 상태로만 관리합니다.
즉, 인스턴스 계층에는 파라미터 기본값/오버라이드 데이터를 직렬화하지 않습니다.

## Marker 와 Event 정책

- `Marker`는 애니메이션 전체 타임라인 기준점입니다.
- `EventTrack`은 실행 트리거용 선택 트랙입니다.
- Roblox 기본 모드에서는 `Marker`를 신호로 매핑해 이벤트를 처리할 수 있습니다.
