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
1. **Track Management**: 내부적으로 여러 개의 `AnimationTrack`을 생성하고 스케줄링합니다.

```lua
LevelSequence
|- BindingGroupsByKey?: {[string]: Instance}
| |- Character1: workspace.Model1
| |- Character2: workspace.Model2
| |- Character1.Weapon: workspace.Model1.Weapon
|- RootBindingsById?: {[string]: Instance}
| |- CameraBinding: workspace.CurrentCamera
| |- EffectBinding: workspace.VfxAnchor
|- AnimationTrackListsByKey: {[string]: {AnimationTrack}}
| |- Character1: {AnimationTrack}
| |- CameraBinding: {AnimationTrack}
|- MasterClock: number
|- SyncGroupsByName: {[string]: SyncGroup}
```

## Constructors

### fromData

`(data: AnimationData, targets: LevelSequenceTargets, params: LevelSequenceParams?) -> (LevelSequence)`

```lua
LevelSequenceTargets = {
    BindingGroupsByKey: {[string]: Instance}?,
    RootBindingsById: {[string]: Instance}?,
}

LevelSequenceParams = {
    MaxDriftSeconds: number?,
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

### AnimationTrackListsByKey

`{[string]: {AnimationTrack}}`

바인딩 키(그룹 키 또는 루트 바인딩 ID)별 `AnimationTrack` 목록입니다.

### MasterClock

`number`

시퀀스 전체가 공유하는 단일 시간축(초)입니다.

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

`AnimationData.Clips[].SyncMarkers`에 동일한 이름의 마커가 존재하면,
같은 `SyncGroup` 안에서 동일 마커 시점으로 정렬합니다.

### Drift Correction

트랙별 시간 편차가 `MaxDriftSeconds`를 넘으면
해당 프레임에 강제 보정(즉시 스냅)합니다.

### Section Window

각 `AnimationTrack`은 현재 `MasterClock` 시점에 활성인 `Section`만 평가합니다.
같은 `PropertyTrack` 안에서 여러 `Section`이 겹치면 섹션 블렌드 규칙으로 합성합니다.

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

### Update

`(deltaTime: number) -> ()`

`MasterClock`을 진행하고 동기화/드리프트 보정을 수행합니다.
