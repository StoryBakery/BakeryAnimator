---
title: AnimationData
---

`AnimationData`는 런타임 애니메이션 자산의 공통 진입 타입입니다.

Unreal Engine 구조에 맞춰 `LevelSequenceData`와 `AnimationSequenceData`를 분리하고,
두 타입은 공통 인터페이스(`AnimationAssetData`)만 공유합니다.

## 레이어 기준

`AnimationData` 계층은 다음 원칙을 따릅니다.

- 에디터 표시와 조작 경험은 Blender 기준 UX를 따릅니다.
- 저장/평가 구조는 Unreal Engine 시퀀스 계층을 따릅니다.
- 최종 동작은 Roblox 런타임 제약을 우선합니다.

충돌 시 우선순위:

1. Roblox 실행 가능성
1. 데이터 구조 일관성
1. 에디터 UX 편의성

## 설계 방향

- `LevelSequenceData`: 바인딩/트랙/섹션 기반 시퀀서 데이터
- `AnimationSequenceData`: 본/곡선 기반 애니메이션 시퀀스 데이터
- `AnimationData`: 두 concrete 타입의 유니온
- `AnimationAssetData`: 두 concrete 타입이 공유하는 최소 공통 인터페이스

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
- 따라서 단일 거대 데이터보다
  **분리된 concrete 타입 + 공통 최소 인터페이스**가 더 UE 구조에 가깝습니다.
