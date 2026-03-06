---
title: Animator Instances
---

애니메이터는 애니메이션 파일의 영속 데이터와 영속 편집 메타데이터를
`Instance` 계층을 진실의 원천으로 두고 관리합니다.
감지 가능한 저장형 값은 `:GetAttributeChangedSignal`과 `:GetAttribute`,
필요 시 `ObjectValue`를 사용해 `Observe` 패턴으로 읽습니다.

세션 전용 UI 상태까지 전부 인스턴스에 저장하지는 않습니다.
선택 상태, 임시 툴 상태, 프리뷰 캐시 같은 비영속 상태는 `EditorShell` 메모리에서 관리합니다.

## 이유

- 인스턴스를 기준으로 하면 `ChangeHistoryService`를 통해 로블록스 스튜디오 수정
  모드에서 `Ctrl + Z`와 `Ctrl + Shift + Z`를 쉽게 구현할 수 있습니다.
- 제3자 에디터를 만들거나 다른 플러그인에서 값을 수정해도, 인스턴스가 진실의 원천이므로 데이터 일관성을 유지할 수 있습니다.

## 변환 경계

1. `AnimationFile` 인스턴스 계층은 스튜디오에서 편집하고 저장하는 영속 원본입니다.
1. [AnimationData](../../runtime/animation-data/index.md)는 인스턴스 계층이나 Blender 동기화 결과를 런타임 평가용으로 정규화한 데이터입니다.
1. [LevelSequence](../../runtime/animation-track/level-sequence.md)와 [AnimationTrack](../../runtime/animation-track/index.md)은 재생 시간, 파라미터, 가중치 같은 런타임 상태를 관리합니다.

즉, 인스턴스 계층은 저장용이고, 런타임 실행기는 상태용입니다.

## 권장 계층

현재 시스템의 인스턴스 계층은 다음 구조를 권장합니다.

```text
AnimationFile
|- Marker (global)
|- BindingGroup? (optional)
|  |- Binding
|     |- Marker (local)
|     |- PropertyTrack
|     |  |- Section
|     |     |- Channel
|     |        |- Keyframe
|     |- EventTrack
|     |  |- Section
|     |     |- EventKey
|     |- AudioTrack
|     |  |- Section
|     |     |- Channel
|     |        |- Keyframe
|     |- ConstraintTrack
|        |- Section

AnimationFile
|- Marker (global)
|- Binding
|  |- Marker (local)
|  |- PropertyTrack
|  |  |- Section
|  |     |- Channel
|  |        |- Keyframe
|  |- EventTrack
|  |  |- Section
|  |     |- EventKey
|  |- AudioTrack
|  |  |- Section
|  |     |- Channel
|  |        |- Keyframe
|  |- ConstraintTrack
|     |- Section
```

`BindingGroup`은 선택 계층이며, 그룹 없이 `Binding`을 루트에 둘 수 있습니다.
`Marker`는 전역(`AnimationFile`)과 로컬(`Binding`) 두 계층을 지원합니다.

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
- [AudioTrack](./classes/audio-track.md)
- [ConstraintTrack](./classes/constraint-track.md)

## 파라미터 정책

파라미터는 Roblox 방식에 맞춰 `AnimationTrack` 런타임 상태로만 관리합니다.
즉, 인스턴스 계층에는 파라미터 기본값/오버라이드 데이터를 직렬화하지 않습니다.

## Marker 와 Event/Audio 정책

- `Marker`는 애니메이션 전체 타임라인 기준점입니다.
- `Binding` 하위 `Marker`는 해당 바인딩 트랙 전용 기준점입니다.
- 같은 이름 충돌 시 로컬 `Marker`가 전역 `Marker`보다 우선합니다.
- `EventTrack`은 실행 트리거용 선택 트랙입니다.
- `ParticleEmitter:Emit(num)` 같은 함수 실행형 동작은 `EventTrack.EndpointId`로 처리합니다.
- `AudioTrack`은 사운드 재생/피치/볼륨 자동화를 위한 선택 트랙입니다.
- 기본 Roblox 어댑터에서는 `Marker`를 신호로 매핑해 이벤트를 처리할 수 있습니다.
