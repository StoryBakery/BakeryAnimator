# AnimationSequence

## 개요

**여러 캐릭터(Model)**와 **객체(Object)**의 애니메이션을 통합 제어하는 **최상위 시퀀서(Top-level Sequencer)**입니다.
Unreal Engine의 `Level Sequence`나 Blender의 `Action Editor`와 유사한 **타임라인 디렉터(Director)** 역할을 수행합니다.

[AnimationTrack](./animation-track.md) 와 다르게 개별 모델의 애니메이션을 관리하는 것이 아닌
여러 모델의 애니메이션 트랙을 관리할 수 있습니다.

또한 동일 모델에서 2 개 이상의 AnimationTrack 을 실행시킬 수 있습니다


기본적인 구조
```lua
AnimationSequence
ㄴ ModelsByKey: {[string]: Model}
    ㄴ Character1: workspace.Model1
    ㄴ Character2: workspace.Model2
    ㄴ Character1.Weapon: workspace.Model1.Weapon
ㄴ AnimationTrackListsByKey: { [string]: {AnimationTrack} }
    ㄴ Character1: {AnimationTrack}
    ㄴ Character2: {AnimationTrack}
    ㄴ Character1.Weapon: {AnimationTrack}
```

## Constructors

#### fromData
`(data: AnimationData, modelsByKey: {[string]: Model}) -> (AnimationSequence)`

#### fromFile
`(file: AnimationFile, modelsByKey: {[string]: Model}) -> (AnimationSequence)`


## Properties

#### Name
`string`

이름입니다, AnimationData 의 이름과 같습니다 

#### ModelsByKey
`{[string]: Model}`
`@readonly`

#### IsPlaying
`boolean`
`@readonly`

#### TimePosition
`number`
`@readonly`

#### Speed
`number`
`@readonly`

## Roles

1.  **Multi-Model Binding**: 하나의 시퀀스 파일이 여러 모델(예: 주인공, 적, 카메라, 소품)을 제어할 수 있도록 바인딩합니다.
2.  **Synchronization**: 여러 캐릭터의 애니메이션을 하나의 타임라인에서 완벽하게 동기화하여 재생합니다.
3.  **Track Management**: 내부적으로 여러 개의 `AnimationTrack`을 생성하고 스케줄링합니다.

## Structure

```lua
AnimationSequence
 ├── Actors: { ["Hero"] = ModelA, ["Monster"] = ModelB }
 ├── Tracks
 │    ├── [Hero] WalkTrack (0s ~ 5s)
 │    ├── [Hero] AttackTrack (5s ~ 6s)
 │    └── [Monster] HitTrack (5.1s ~ 6s)
 └── Speed / TimePosition / IsPlaying (Global Control)
```

## Methods

#### UpdateModelsByKey
`(callback: (modelsByKey: {[string]: Model}) -> ({[string]: Model}))->()`

#### Play
`(fadeTime: number?, speed: number?) -> ()`

시퀀스 전체를 재생합니다. 내부의 모든 트랙이 동기화되어 시작됩니다.

#### Stop
`(fadeTime: number?) -> ()`

시퀀스 전체를 정지합니다.

#### SetTimePosition
`(time: number) -> ()`

시퀀스의 타임라인을 특정 시점으로 이동시킵니다.


#### Set
`(time: number) -> ()`

시퀀스의 타임라인을 특정 시점으로 이동시킵니다.
