---
title: BindingGroupAnimationData
---

단일 `bindingGroupKey`에 대응되는 애니메이션 데이터 묶음입니다.
상위 컨테이너는 `LevelSequenceData`입니다.

- 특정 바인딩 그룹에 속한 트랙 데이터를 모읍니다.
- 바인딩 그룹 단위와 그 하위 범위에 적용되는 클립들을 함께 보관합니다.
- 하나의 파일이 여러 바인딩 그룹을 동시에 제어할 때 기준점 역할을 합니다.

## 구조

```lua
LevelSequenceData
|- BindingGroupAnimationDatasByKey
|  |- Root: BindingGroupAnimationData
|  |- Hero: BindingGroupAnimationData
|  |- Enemy: BindingGroupAnimationData

BindingGroupAnimationData
|- TracksByBindingId
|  |- 8d3f...: TrackData
|  |- f2b1...: TrackData
|- Clips
|  |- BindingGroup Level Clip
|  |- Binding Level Clip
|  |- Property Level Clip

TrackData
|- TrackName
|- BindingId
|- PropertyTracksByKey
|  |- Position: PropertyTrackData
|  |- CFrame: PropertyTrackData
|- EventTracksByKey
|  |- Footstep: EventTrackData
|- ConstraintTracksByKey
|  |- AimConstraint: ConstraintTrackData

PropertyTrackData
|- Sections
|  |- SectionData
|  |- SectionData

SectionData
|- SectionId
|- StartTime
|- EndTime
|- BlendType
|- Channels
|  |- X: ChannelData
|  |- Y: ChannelData
|  |- Z: ChannelData

EventTrackData
|- Sections
|  |- EventSectionData

ConstraintTrackData
|- Sections
|  |- ConstraintSectionData
```

## 포함하는 데이터

- **Track Data**: 바인딩 그룹 내부의 각 바인딩별 애니메이션 곡선 데이터입니다.
  런타임에서는 `BindingId`를 1순위 키로 사용하고, `trackName`은 보조 키로 사용합니다.
- **PropertyTrack Data**: 속성 단위 트랙 컨테이너입니다.
- **Section Data**: 속성 트랙 내부의 시간 구간 단위입니다.
  같은 속성에서 여러 섹션을 구간/겹침 규칙으로 조합할 수 있습니다.
- **Channel Data**: 키프레임 목록을 가지는 곡선 단위입니다.
  기본 키 보간 모드는 `Bezier`를 권장합니다.
- **EventTrack Data**: 구간 내 이벤트 키를 디렉터/엔드포인트로 디스패치하는 트랙 데이터입니다.
- **ConstraintTrack Data**: 바인딩 간 제약(Parent, LookAt 등)을 시간 구간 단위로 적용하는 트랙 데이터입니다.
- **BindingGroup Level Clip**: 바인딩 그룹 전체에 적용되는 하위 클립입니다.
- **Binding Level Clip**: 바인딩 그룹 내부의 특정 바인딩을 대상으로 하는 하위 클립입니다.
- **Property Level Clip**: 특정 Property 변화만을 위한 세밀한 하위 클립입니다.

`EasingData`는 섹션 경계 블렌드(`EaseIn`/`EaseOut`) 전용 데이터입니다.
키프레임 구간 보간은 `KeyframeData.InterpolationMode`를 사용합니다.

```lua
type EasingData = {
    DurationSeconds: number?,
    Function: "Linear" | "EaseIn" | "EaseOut" | "EaseInOut" | "Custom",
}

type BezierHandleData = {
    Time: number,
    Value: number,
    HandleType: "Auto" | "Free" | "Vector" | "Aligned",
}

type KeyframeData = {
    Time: number,
    Value: any,
    InterpolationMode: "Constant" | "Linear" | "Bezier",
    LeftHandle: BezierHandleData?,
    RightHandle: BezierHandleData?,
}

type ChannelData = {
    DefaultValue: any?,
    Keys: {KeyframeData},
}

type SectionData = {
    SectionId: string,
    StartTime: number,
    EndTime: number?,
    PreRollFrames: number?,
    PostRollFrames: number?,
    RowIndex: number?,
    OverlapPriority: number?,
    EaseIn: EasingData?,
    EaseOut: EasingData?,
    CompletionMode: "KeepState" | "RestoreState" | "ProjectDefault",
    BlendType: "Absolute" | "Additive" | "Relative",
    Channels: {[string]: ChannelData},
}

type PropertyTrackData = {
    PropertyKey: string,
    Sections: {SectionData},
}

type EventKeyData = {
    Time: number,
    EndpointId: string,
    Payload: table?,
}

type EventSectionData = {
    SectionId: string,
    StartTime: number,
    EndTime: number?,
    Events: {EventKeyData},
}

type EventTrackData = {
    TrackName: string,
    Sections: {EventSectionData},
}

type ConstraintSectionData = {
    SectionId: string,
    StartTime: number,
    EndTime: number?,
    ConstraintType: "Parent" | "Position" | "Rotation" | "Scale" | "LookAt",
    SourceBindingId: string,
    TargetBindingId: string,
    WeightChannel: ChannelData?,
}

type ConstraintTrackData = {
    TrackName: string,
    Sections: {ConstraintSectionData},
}

type TrackData = {
    TrackName: string,
    BindingId: string,
    PropertyTracksByKey: {[string]: PropertyTrackData},
    EventTracksByKey: {[string]: EventTrackData}?,
    ConstraintTracksByKey: {[string]: ConstraintTrackData}?,
}
```

## 포함하지 않는 데이터

`BindingGroupAnimationData`는 정적 데이터만 다룹니다.
현재 시간, 재생 상태, Weight, BlendMode, 파라미터 기본값/오버라이드 같은 런타임 상태는 가지지 않으며,
이런 값은 `AnimationTrack`과 `LevelSequence`가 관리합니다.

## 바인딩 예시

- **Single BindingGroup**: `LevelSequenceData.BindingGroupAnimationDatasByKey["Root"]`
- **Multi BindingGroup**: `LevelSequenceData.BindingGroupAnimationDatasByKey["Hero"]`,
  `LevelSequenceData.BindingGroupAnimationDatasByKey["Enemy"]`

그룹 없는 바인딩 구성은 암묵적 `"Root"` 그룹으로 정규화되어 저장됩니다.

즉, `LevelSequenceData`가 파일 전체 컨테이너라면,
`BindingGroupAnimationData`는 그 내부에서 바인딩 그룹 하나를 위한 서브 컨테이너입니다.
