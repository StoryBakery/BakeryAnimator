---
title: LevelSequence
---

**여러 바인딩 그룹/바인딩**의 애니메이션을 통합 제어하는 **최상위 시퀀서**입니다.

Unreal Engine의 `Level Sequence`나 Blender의 `Action Editor`와 유사한
**타임라인 디렉터** 역할을 수행합니다.

내부적으로 여러 `AnimationTrack`을 **조합**하여 관리합니다.

1. **Multi-Binding Scope**: 하나의 시퀀스 파일이 여러 대상(예: 주인공, 적, 카메라,
   소품)을 바인딩 그룹 또는 루트 바인딩 형태로 제어할 수 있습니다.
1. **Synchronization**: 여러 캐릭터의 애니메이션을 하나의 타임라인에서 동기화하여 재생합니다.
1. **Track Management**: 내부적으로 여러 트랙 타입(`Property`, `Event`,
   `Constraint`, `Subsequence`, `CameraCut`)을 스케줄링합니다.

```lua
LevelSequence
|- BindingGroupsByKey?: {[string]: Instance}
| |- Character1: workspace.Model1
| |- Character2: workspace.Model2
| |- Character1.Weapon: workspace.Model1.Weapon
|- RootBindingsById?: {[string]: Instance}
| |- CameraBinding: workspace.CurrentCamera
| |- EffectBinding: workspace.VfxAnchor
|- TrackCollectionsByBindingKey: {[string]: TrackCollection}
| |- Character1: TrackCollection
| |- CameraBinding: TrackCollection
|- SubsequenceInstancesByKey: {[string]: LevelSequence}
| |- ShotA: LevelSequence
| |- ShotB: LevelSequence
|- HierarchicalBiasBySequenceKey: {[string]: number}
|- BindingTagsToBindingIds: {[string]: {string}}
|- SpawnablesByBindingId: {[string]: SpawnableState}
|- MasterClock: number
|- ClockSource: "Tick" | "Platform" | "Audio" | "Timecode"
|- DisplayRate: number
|- TickResolution: number
|- TimeController: TimeController?
|- SyncGroupsByName: {[string]: SyncGroup}
```

## Constructors

### fromData

`(data: LevelSequenceData, targets: LevelSequenceTargets, params: LevelSequenceParams?) -> (LevelSequence)`

```lua
LevelSequenceTargets = {
    BindingGroupsByKey: {[string]: Instance}?,
    RootBindingsById: {[string]: Instance}?,
    BindingTagsToBindingIds: {[string]: {string}}?,
}

LevelSequenceParams = {
    MaxDriftSeconds: number?,
    ClockSource: "Tick" | "Platform" | "Audio" | "Timecode"?,
    DisplayRate: number?,
    TickResolution: number?,
    UseDeterministicClock: boolean?,
}

TrackCollection = {
    PropertyTracks: {AnimationTrack}?,
    EventTracks: {EventTrack}?,
    ConstraintTracks: {ConstraintTrack}?,
    SubsequenceTracks: {SubsequenceTrack}?,
    CameraCutTracks: {CameraCutTrack}?,
}

SpawnableState = {
    BindingId: string,
    IsSpawned: boolean,
    SpawnOwner: string?,
}
```

### fromFile

`(file: AnimationFile, targets: LevelSequenceTargets, params: LevelSequenceParams?) -> (LevelSequence)`

## Properties

### BindingGroupsByKey

`{[string]: Instance}?`

바인딩 그룹 키와 실제 `Instance`의 매핑입니다.
필수 값이 아니며, 그룹 없이 루트 바인딩만으로 구성할 수 있습니다.

### RootBindingsById

`{[string]: Instance}?`

그룹 없이 루트 레벨에 직접 바인딩되는 대상 맵입니다.

### TrackCollectionsByBindingKey

`{[string]: TrackCollection}`

바인딩 키(그룹 키 또는 루트 바인딩 ID)별 트랙 컬렉션입니다.
속성 트랙 외에도 이벤트/제약/서브시퀀스/카메라 컷 트랙을 함께 관리합니다.

### SubsequenceInstancesByKey

`{[string]: LevelSequence}`

샷/서브시퀀스 인스턴스 맵입니다.
부모 시퀀스는 하위 시퀀스를 계층적으로 평가합니다.

### HierarchicalBiasBySequenceKey

`{[string]: number}`

동일 시점에 여러 계층 결과가 충돌할 때 적용할 우선순위입니다.
값이 높을수록 최종 평가에서 나중에 적용됩니다.

### BindingTagsToBindingIds

`{[string]: {string}}`

동적 바인딩을 위한 태그 인덱스입니다.
런타임에서 태그를 기준으로 바인딩 대상을 재해결할 수 있습니다.

### SpawnablesByBindingId

`{[string]: SpawnableState}`

시퀀스가 소유하는 생성형(Spawnable) 바인딩의 런타임 상태입니다.

### MasterClock

`number`

시퀀스 전체가 공유하는 단일 시간축(초)입니다.

### ClockSource

`"Tick" | "Platform" | "Audio" | "Timecode"`

시퀀스 시간이 어떤 기준 시계를 따를지 정의합니다.

### DisplayRate

`number`

UI/편집에서 기준이 되는 표시 FPS입니다.

### TickResolution

`number`

내부 평가 단위(틱/초)입니다.
`DisplayRate`와 분리해 고정밀 평가를 지원합니다.

### TimeController

`TimeController?`

외부 시간 입력과 연동하기 위한 커스텀 시간 컨트롤러입니다.
미지정 시 `ClockSource` 기본 컨트롤러를 사용합니다.

### MaxDriftSeconds

`number`
`@default 1/120`

트랙 간 시간 차이가 이 값을 초과하면 즉시 재동기화합니다.

### IsPlaying

`boolean`

시퀀스 재생 여부입니다.

## Synchronization Contract

### Single Clock

모든 하위 트랙은 `MasterClock`을 기준으로 샘플링합니다.
각 트랙이 자체 `deltaTime`으로 독립 진행하지 않습니다.

### Marker Lock

`LevelSequenceData.Clips[].SyncMarkers`에 동일한 이름의 마커가 존재하면,
같은 `SyncGroup` 안에서 동일 마커 시점으로 정렬합니다.

### Drift Correction

트랙별 시간 편차가 `MaxDriftSeconds`를 넘으면
해당 프레임에 강제 보정(즉시 스냅)합니다.

### Section Window

각 `AnimationTrack`은 현재 `MasterClock` 시점에 활성인 `Section`만 평가합니다.
같은 `PropertyTrack` 안에서 여러 `Section`이 겹치면 섹션 블렌드 규칙으로 합성합니다.
상세 규약은 [Section Evaluator](./section-evaluator.md)를 따릅니다.

### Hierarchical Bias

부모/자식 시퀀스가 같은 바인딩/속성에 동시에 값을 쓰면
`HierarchicalBiasBySequenceKey`를 기준으로 적용 순서를 결정합니다.

### Dynamic Binding

태그 기반 바인딩이 활성화되면 `BindingTagsToBindingIds`를 통해
실제 대상을 재조회한 뒤 해당 프레임부터 평가 대상에 반영합니다.

### Spawn Lifecycle

Spawnable 바인딩은 시퀀스 평가 구간에서 생성/파괴될 수 있으며,
상태는 `SpawnablesByBindingId`에 저장합니다.

### Event Dispatch

`EventTrack`은 현재 프레임의 이벤트 키를 수집해
디렉터/엔드포인트에 순서대로 디스패치합니다.

### Constraint Solve Order

`ConstraintTrack`은 `PropertyTrack` 평가 이후, 최종 적용 직전에 처리합니다.
여러 제약이 겹치면 제약 우선순위와 섹션 블렌드 규칙으로 합성합니다.

## Methods

### Play

`(fadeTime: number?) -> ()`

하위 트랙들을 동일한 `MasterClock` 기준으로 재생 시작합니다.

### Stop

`(fadeTime: number?) -> ()`

하위 트랙들을 정지합니다.

### SetTimePosition

`(time: number) -> ()`

`MasterClock`을 설정하고 모든 하위 트랙 시간을 동일하게 맞춥니다.

### SetTimeController

`(controller: TimeController?) -> ()`

커스텀 시간 컨트롤러를 주입하거나 해제합니다.

### RebindBindingTag

`(tag: string, bindingIds: {string}) -> ()`

태그 기반 바인딩 매핑을 런타임에 갱신합니다.

### SetHierarchicalBias

`(sequenceKey: string, bias: number) -> ()`

특정 하위 시퀀스의 계층 우선순위를 갱신합니다.

### Update

`(deltaTime: number) -> ()`

`MasterClock`을 진행하고 동기화/드리프트 보정을 수행합니다.
