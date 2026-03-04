---
title: AnimationSequenceData
---

`AnimationSequenceData`는 캐릭터/본 기반 애니메이션 평가에 필요한 concrete 타입입니다.
`AnimationAssetData` 공통 인터페이스를 확장합니다.

## Properties

### DataType

`"AnimationSequence"`

타입 분기 식별자입니다.

### Name

`string`

애니메이션 시퀀스 이름입니다.

### Duration

`number`

시퀀스 길이(초)입니다.

### FrameRate

`number`

기본 FPS입니다.

### Tags

`{string}?`

시퀀스 태그 목록입니다.

### SkeletonId

`string`

리타게팅 기준 스켈레톤 식별자입니다.

### BoneTracksByName

`{[string]: BoneTrackData}`

본 이름별 트랙 데이터입니다.

### CurveTracksByKey

`{[string]: CurveTrackData}`

보조 곡선(예: 얼굴/게임플레이 파라미터) 데이터입니다.

### Notifies

`{AnimationNotifyData}`

애니메이션 노티파이 목록입니다.

### SyncMarkers

`{SyncMarkerData}`

동기화 마커 목록입니다.

### RootMotionSettings

`RootMotionSettingsData?`

루트 모션 추출/적용 설정입니다.

### CompressionSettings

`CompressionSettingsData?`

곡선/키프레임 압축 정책입니다.

### RetargetSettings

`RetargetSettingsData?`

리타게팅 정책입니다.

## Types

```lua
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

type BoneTrackData = {
    BoneName: string,
    TranslationChannels: {[string]: ChannelData},
    RotationChannels: {[string]: ChannelData},
    ScaleChannels: {[string]: ChannelData},
}

type CurveTrackData = {
    CurveKey: string,
    Channel: ChannelData,
}

type AnimationNotifyData = {
    Name: string,
    Time: number,
    Payload: table?,
}

type SyncMarkerData = {
    Name: string,
    Time: number,
}

type RootMotionSettingsData = {
    Enabled: boolean,
    Space: "Component" | "World",
}

type CompressionSettingsData = {
    Codec: "None" | "ACL" | "Custom",
    ErrorTolerance: number?,
    ReduceKeys: boolean?,
}

type RetargetSettingsData = {
    RetargetProfileId: string?,
    RetargetScaleMode: "None" | "Uniform" | "PerBone"?,
}
```

## Methods

### GetBoneTrack

`(boneName: string) -> (BoneTrackData?)`

본 이름 기준으로 트랙 데이터를 반환합니다.

### GetCurveTrack

`(curveKey: string) -> (CurveTrackData?)`

곡선 키 기준으로 곡선 트랙을 반환합니다.

### GetNotifiesInRange

`(startTime: number, endTime: number) -> ({AnimationNotifyData})`

시간 범위에 포함되는 노티파이를 반환합니다.

### SamplePoseAtTime

`(time: number) -> (table)`

지정 시간의 본 포즈 샘플을 반환합니다.
