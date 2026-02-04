# AnimationSequence

**여러 캐릭터(Model)**와 **객체(Object)**의 애니메이션을 통합 제어하는 **최상위 시퀀서(Top-level Sequencer)**입니다.
Unreal Engine의 `Level Sequence`나 Blender의 `Action Editor`와 유사한 **타임라인 디렉터(Director)** 역할을 수행합니다.

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

## Constructors

#### fromData
(data: AnimationData, models: {[string]: Model}) -> (AnimationSequence)

**통합 애니메이션 데이터(`AnimationData`)**와 대상 모델들을 매핑하여 인스턴스를 생성합니다.

*   `data`: `Models` 및 `Clips` 정보를 포함한 애니메이션 데이터.
*   `models`: 데이터 내의 Model 이름(Key)과 실체 인스턴스(Value)를 매핑한 테이블.
    *   예: `{ ["Hero"] = workspace.Dummy, ["Camera"] = workspace.CurrentCamera }`

## Methods

#### Play
(fadeTime: number?, speed: number?) -> ()
시퀀스 전체를 재생합니다. 내부의 모든 트랙이 동기화되어 시작됩니다.

#### Stop
(fadeTime: number?) -> ()
시퀀스 전체를 정지합니다.

#### SetTimePosition
(time: number) -> ()
시퀀스의 타임라인을 특정 시점으로 이동시킵니다. (스크러빙)
