---
title: AnimationData
---

`AnimationData`는 런타임 애니메이션 자산의 공통 진입 타입입니다.

Unreal Engine 구조에 맞춰 `LevelSequenceData`와 `AnimationSequenceData`를 분리하고,
두 타입은 공통 인터페이스(`AnimationAssetData`)만 공유합니다.

공통 레이어 기준과 충돌 우선순위는 [Architecture Main](../../main.md)을 따릅니다.

## 설계 방향

- `LevelSequenceData`: 바인딩/트랙/섹션 기반 시퀀서 데이터
  - 전역 마커(`SequenceMarkers`)와 트랙 로컬 마커(`TrackData.Markers`)를 함께 지원합니다.
  - `PropertyTrack`, `EventTrack`, `AudioTrack`, `ConstraintTrack`을 함께 지원합니다.
- `AnimationSequenceData`: 본/곡선 기반 애니메이션 시퀀스 데이터
- `AnimationData`: 두 concrete 타입의 유니온
- `AnimationAssetData`: 두 concrete 타입이 공유하는 최소 공통 인터페이스

## 계층 관계

1. [Animator Instances](../../animator/instances/instances.md)는 편집 저장용 원본입니다.
1. `AnimationData`는 그 원본을 런타임 평가용으로 정규화한 정적 데이터입니다.
1. [LevelSequence](../animation-track/level-sequence.md)와 [AnimationTrack](../animation-track/index.md)은 이 정적 데이터를 읽어 런타임 상태를 붙입니다.

따라서 `AnimationData`는 편집기 내부 저장 구조를 그대로 복제하는 계층이 아니라,
런타임 평가에 맞게 압축되고 정규화된 계층입니다.

## 공통 인터페이스

### AnimationAssetData

```lua
{
    DataType: "LevelSequence" | "AnimationSequence",
    Name: string,
    Duration: number,
    FrameRate: number,
    Tags: {string}?,
}
```

- `DataType`은 concrete 타입 분기 키입니다.
- 공통 인터페이스는 최소 필드만 가지며, 상세 필드는 각 concrete 타입이 확장합니다.

## Concrete Types

- [LevelSequenceData](./level-sequence-data.md)
- [AnimationSequenceData](./animation-sequence-data.md)
- [BindingGroupAnimationData](./binding-group-animation-data.md)

```lua
type AnimationData = LevelSequenceData | AnimationSequenceData
```

## Constructors

### fromTable

`(data: table, params: AnimationDataParams?) -> (AnimationData)`

`data.DataType`을 기준으로 concrete 타입을 생성합니다.

### fromInstance

`(animationFile: AnimationFile, params: AnimationDataParams?) -> (AnimationData)`

인스턴스 메타(`DataType`)를 기준으로 concrete 타입을 생성합니다.

## Methods

### Type Guards

#### IsLevelSequenceData

`(data: AnimationData) -> (boolean)`

`data.DataType == "LevelSequence"` 여부를 반환합니다.

#### IsAnimationSequenceData

`(data: AnimationData) -> (boolean)`

`data.DataType == "AnimationSequence"` 여부를 반환합니다.

## UE 정렬 메모

- Unreal도 `LevelSequence` 계열(`UMovieSceneSequence`)과
  `AnimationSequence` 계열(`UAnimationAsset`)을 분리합니다.
- `BindingGroupAnimationData`는 공통 인터페이스가 아니라
  `LevelSequenceData` 내부 서브 컨테이너로 유지합니다.
- 따라서 단일 거대 데이터보다
  **분리된 concrete 타입 + 공통 최소 인터페이스**가 더 UE 구조에 가깝습니다.
