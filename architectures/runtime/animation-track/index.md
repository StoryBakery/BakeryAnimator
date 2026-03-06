---
title: AnimationTrack
---

`LevelSequenceData`를 실제 로블록스 바인딩 그룹 대상에 바인딩하여 재생하는 **런타임 실행기**입니다.
재생 시간/가중치 같은 상태와 함께, 트랙 인스턴스별 파라미터 상태를 관리합니다.

단일 `BindingGroup` 또는 그룹 없는 루트 바인딩 범위에 사용됩니다.

## 관계

- `LevelSequence`는 여러 바인딩 그룹/루트 바인딩을 조율하는 상위 디렉터입니다.
- `AnimationTrack`은 그 내부에서 단일 바인딩 그룹 범위를 평가하는 실행기입니다.
- 멀티 캐릭터/카메라/오디오를 한 타임라인에서 맞추려면 `LevelSequence`가 필요하고,
  개별 그룹 재생만 필요하면 `AnimationTrack` 단독 사용도 가능합니다.

상세 설계:

- [Section Evaluator](./section-evaluator.md)

내부 파이프라인은 다음 순서로 동작합니다.

1. **Player**: 시간/재생 상태를 갱신합니다.
1. **Parameter Resolver**: `GetParameterDefaults()`와 런타임 오버라이드를 병합합니다.
1. **Evaluator**: 현재 시간에서 `PropertyTrack`의 활성 `Section`들을 평가/합성해 포즈를 계산합니다.
1. **Constraint Solver**: 제약 트랙 결과를 합성하여 최종 포즈를 보정합니다.
1. **Applier**: 계산된 포즈를 실제 `Instance`에 적용합니다.
1. **Marker Evaluator**: 전역/로컬 마커를 수집해 동기화 기준을 계산합니다.
1. **Audio Scheduler**: 활성 `AudioTrack` 섹션을 평가해 재생/정지/볼륨/피치를 갱신합니다.
1. **Event Dispatcher**: 해당 프레임의 이벤트 키를 엔드포인트로 디스패치합니다.
   `ExecutionMode`, 재생 방향 플래그로 발화 조건을 판정합니다.

## Supported Track Types

- `PropertyTrack`: 속성 값 샘플링과 섹션 블렌딩.
- `EventTrack`: 노티파이/게임플레이 이벤트 디스패치.
- `AudioTrack`: 오디오 클립 재생과 볼륨/피치 곡선 평가.
- `ConstraintTrack`: 바인딩 간 제약(Parent, LookAt 등) 적용.
- `Marker`: 시퀀스 전역/트랙 로컬 동기화 기준점.

## Parameter Runtime Model

- 파라미터는 `AnimationTrack` 인스턴스 상태입니다.
- 같은 데이터 자산을 여러 트랙에서 재생해도 파라미터 값은 서로 공유하지 않습니다.
- 파라미터 기본값은 `AnimationTrack:GetParameterDefaults()`로 조회합니다.
- 파라미터 기본값은 애니메이션 데이터 파일에 직렬화하지 않습니다.
- `GetParameter`는 오버라이드 값을 우선하고, 없으면 기본값을 조회합니다.
- 기본값도 없으면 `nil`을 반환합니다.

파라미터 기본값 출처:

- 기본 경로에서는 Roblox `AnimationTrack:GetParameterDefaults()` 결과를 사용합니다.
- 해당 API를 제공하지 않는 런타임/어댑터에서는 빈 맵 `{}`을 기본값으로 사용합니다.
- 파라미터 기본값은 인스턴스/데이터 파일에 직렬화하지 않습니다.

## Constructors

### fromData

```lua
(data: LevelSequenceData, targetBindingGroup: Instance, params: AnimationTrackParams?) -> (AnimationTrack)
```

단일 바인딩 그룹(`targetBindingGroup`)을 대상으로 하는 애니메이션 트랙을 생성합니다.
시퀀스 데이터(`LevelSequenceData`)에서 `targetBindingGroup`에 대응되는 바인딩 그룹 데이터를 바인딩해 사용합니다.

- `data`: 시퀀스 애니메이션 데이터.
- `targetBindingGroup`: 애니메이션을 적용할 실제 대상 `Instance`입니다.
  (예: `Model`, `Folder`)

```lua
AnimationTrackParams = {
    Speed: number?,
    Weight: number?,
    Priority: number?,
    PoseMode: AnimationPoseMode?,
    InitialParameters: {[string]: any}?,
}
```

### fromFile

```lua
(file: AnimationFile, targetBindingGroup: Instance, params: AnimationTrackParams?) -> (AnimationTrack)
```

## Properties

### IsDestroyed

`boolean`

트랙 인스턴스의 파괴 여부입니다.

### Maid

`Maid`

트랙 인스턴스의 수명 관리를 담당하는 Maid 인스턴스입니다.

### Data

`LevelSequenceData`

재생 중인 원본 애니메이션 데이터입니다.

### SequenceMarkers

`{MarkerData}?`

시퀀스 전역 마커 목록입니다.

### TrackMarkersByBindingId

`{[string]: {TrackMarkerData}}?`

바인딩 ID별 트랙 로컬 마커 목록입니다.

### AudioTracksByBindingId

`{[string]: {[string]: AudioTrackData}}?`

바인딩 ID별 오디오 트랙 맵입니다.

### TimePosition

`number`

현재 재생 시간 (초) 입니다.

### Speed

`number`

재생 속도 배율입니다. 기본값은 `1.0`입니다.

### Weight

`number`

애니메이션의 영향력(가중치)입니다. $[0.0, 1.0]$ 사이의 값을 가집니다.
여러 트랙이 섞일 때 이 값이 클수록 더 강하게 표현됩니다.

### BlendMode

`AnimationBlendMode`
`@default WeightedSet`

이 트랙이 **레이어 내부에서** 다른 트랙들과 합성되는 방식입니다.

- `WeightedSet`: 가중치(Weight) 비율에 따라 값을 적용합니다.
  - 단일 트랙일 경우 Weight만큼 하위 레이어를 덮어씁니다. (Alpha Blend)
  - 같은 우선순위에 여러 트랙이 있을 경우, 가중치 비율대로 섞입니다. (BlendSpace)
- `Add`: 하위 레이어 결과값 위에 현재 값을 더합니다. (반동, 흔들림 등)
- `Multiply`: 하위 레이어 결과값에 현재 값을 곱합니다.

### PoseMode

`AnimationPoseMode`
`@default Absolute`

포즈를 어떤 기준으로 평가할지 정의합니다.

- `Absolute`: 키프레임 값을 절대값으로 평가합니다.
- `Relative`: `ReferencePose` 대비 변화량(Delta)으로 평가합니다.

`Relative`는 블렌드 함수가 아니라 평가 단계 동작입니다.

### IsPlaying

`boolean`

현재 재생 중인지 여부입니다.

### Priority

`number`
`@default 0`

**레이어 내부에서의** 연산 우선순위입니다.
`ValueByPriority` 노드의 우선순위로 매핑됩니다. 높은 우선순위의 트랙이 나중에 연산되어 결과를 덮어쓰거나 합성합니다.

### Looped

`boolean`
`@default false`

`Data`의 설정을 덮어쓰고 반복 여부를 실시간으로 제어합니다.

### ReferencePose

`table?`

`PoseMode`가 `Relative`일 때 기준이 되는 포즈입니다.
미지정 시 트랙 시작 시점 포즈를 기준으로 사용합니다.

### ParametersByKey

`{[string]: any}`

현재 트랙 인스턴스의 파라미터 오버라이드 값입니다.

## Methods

### Destroy

() -> ()

트랙을 파괴하고 관련 리소스를 정리합니다.

### Play

(fadeTime: number?, weight: number?, speed: number?) -> ()

애니메이션 재생을 시작하거나 재개합니다.

- `fadeTime`: 지정한 시간(초) 동안 Weight를 0에서 목표값까지 부드럽게 올립니다. (Fade In)
- `weight`: 재생 시 목표 Weight입니다.
- `speed`: 재생 속도입니다.

### Stop

(fadeTime: number?) -> ()

애니메이션을 정지합니다.

- `fadeTime`: 지정한 시간(초) 동안 Weight를 현재 값에서 0으로 부드럽게 내린 후 정지합니다. (Fade Out)

### AdjustWeight

(targetWeight: number, fadeTime: number?) -> ()

실행 중에 트랙의 가중치를 부드럽게 변경합니다. 다른 동작과 섞이거나 서서히 사라지게 할 때 사용합니다.

### SetBlendMode

`(mode: AnimationBlendMode) -> ()`

합성 모드를 변경합니다.

### SetPoseMode

`(mode: AnimationPoseMode) -> ()`

포즈 평가 기준을 변경합니다.

### SetReferencePose

`(pose: table?) -> ()`

`Relative` 평가 기준 포즈를 설정합니다.

### SetTimePosition

(time: number) -> ()

현재 진행 시간을 설정하고 즉시 포즈를 업데이트합니다.

### SetParameter

`(key: string, value: any) -> ()`

파라미터 오버라이드 값을 설정합니다.

### GetParameter

`(key: string) -> (any?)`

현재 파라미터 값을 조회합니다.
오버라이드가 있으면 해당 값을 반환하고, 없으면 기본값을 반환합니다.

### GetParameterDefaults

`() -> ({[string]: any})`

트랙이 참조하는 기본 파라미터 맵을 반환합니다.
Roblox `AnimationTrack:GetParameterDefaults()`와 동일 의미입니다.

### GetSequenceMarkers

`() -> ({MarkerData})`

시퀀스 전역 마커를 반환합니다.

### GetTrackMarkers

`(bindingId: string) -> ({TrackMarkerData})`

지정 바인딩의 로컬 마커를 반환합니다.

### GetAudioTracks

`(bindingId: string) -> ({[string]: AudioTrackData})`

지정 바인딩의 오디오 트랙 맵을 반환합니다.

### GetMarkersInRange

`(startTime: number, endTime: number, bindingId: string?) -> ({MarkerData})`

구간에 포함되는 마커를 반환합니다.
`bindingId`가 있으면 로컬+전역 병합, 없으면 전역만 반환합니다.

### ResetParameter

`(key: string) -> ()`

지정한 키의 오버라이드를 제거하고 기본값으로 복귀시킵니다.

### ResetParameters

`() -> ()`

모든 파라미터 오버라이드를 제거합니다.
