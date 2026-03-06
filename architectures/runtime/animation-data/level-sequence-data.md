---
title: LevelSequenceData
---

`LevelSequenceData`는 시퀀서 평가에 필요한 데이터를 담는 concrete 타입입니다.
`AnimationAssetData` 공통 인터페이스를 확장합니다.

## Properties

### DataType

`"LevelSequence"`

타입 분기 식별자입니다.

### Name

`string`

시퀀스 이름입니다.

### Duration

`number`

시퀀스 길이(초)입니다.

### FrameRate

`number`

시퀀스 기본 FPS입니다.

### Tags

`{string}?`

시퀀스 태그 목록입니다.

### BindingGroupAnimationDatasByKey

`{[string]: BindingGroupAnimationData}`

바인딩 그룹 키별 트랙 데이터 맵입니다.
그룹 없는 데이터는 `"Root"`로 정규화합니다.

### Clips

`{LevelSequenceClip}`

시퀀서 클립 목록입니다.

### SequenceMarkers

`{MarkerData}?`

시퀀스 전체 타임라인에서 공유되는 전역 마커 목록입니다.

### SequencerSettings

`SequencerSettingsData`

시간 평가 설정입니다.
Blender 동기화 시 `DisplayRate`는 Scene FPS를 사용하고,
`TickResolution`은 더 높은 정밀도 단위를 사용합니다.

### RecordingMetadata

`RecordingMetadataData?`

테이크/레코딩 메타데이터입니다.
Blender 동기화 시 `SourceSceneName`으로 추출 기준 Scene을 기록할 수 있습니다.

### BindingTagsToBindingIds

`{[string]: {string}}?`

동적 바인딩용 태그 인덱스입니다.

## Types

```lua
type SequencerSettingsData = {
    DisplayRate: number,
    TickResolution: number,
    ClockSource: "Tick" | "Platform" | "Audio" | "Timecode",
    DefaultCompletionMode: "KeepState" | "RestoreState" | "ProjectDefault",
}

type RecordingMetadataData = {
    Slate: string?,
    TakeNumber: number?,
    SourceSceneName: string?,
    SourceTimecode: string?,
    RecorderVersion: string?,
}

type EaseCurveData = {
    DurationSeconds: number?,
    Function: "Linear" | "EaseIn" | "EaseOut" | "EaseInOut" | "Custom",
}

type MarkerData = {
    Name: string,
    Time: number,
    Value: string?,
}

type SectionEvalData = {
    StartTime: number,
    EndTime: number?,
    PreRollFrames: number?,
    PostRollFrames: number?,
    RowIndex: number?,
    OverlapPriority: number?,
    EaseIn: EaseCurveData?,
    EaseOut: EaseCurveData?,
    CompletionMode: "KeepState" | "RestoreState" | "ProjectDefault",
    BlendType: "Absolute" | "Additive" | "Relative",
}

type LevelSequenceClip = {
    ClipType: "Animation" | "Audio" | "Event" | "Constraint" | "Subsequence" | "CameraCut",
    Scope: "Global" | "BindingGroup" | "Binding" | "Property",
    StartTime: number,
    Duration: number,
    Speed: number?,
    Weight: number?,
    Offset: number?,
    Section: SectionEvalData?,
    BindingIds: {string}?,
    BindingTags: {string}?,
    AudioAssetId: string?,
    AttachBindingId: string?,
    StartOffsetSeconds: number?,
    Looping: boolean?,
    EventEndpointId: string?,
    ConstraintId: string?,
    SubsequenceKey: string?,
    Payload: table?,
}
```

## Methods

### GetBindingGroupData

`(bindingGroupKey: string?) -> (BindingGroupAnimationData?)`

바인딩 그룹 데이터를 반환합니다.

### IsSingleBindingGroup

`() -> (boolean)`

바인딩 그룹 개수가 1개인지 반환합니다.

### FindTrackData

`(bindingGroupKey: string, trackName: string) -> (TrackData?)`

트랙 이름 기준으로 트랙 데이터를 반환합니다.

### FindTrackDataByBindingId

`(bindingGroupKey: string, bindingId: string) -> (TrackData?)`

`BindingId` 기준으로 트랙 데이터를 반환합니다.

### FindSubsequenceClip

`(sequenceKey: string) -> (LevelSequenceClip?)`

`ClipType = "Subsequence"`인 클립을 반환합니다.

### FindAudioClipsInRange

`(startTime: number, endTime: number) -> ({LevelSequenceClip})`

`ClipType = "Audio"`인 클립 중 시간 구간에 겹치는 항목을 반환합니다.

### GetSequenceMarkers

`() -> ({MarkerData})`

시퀀스 전역 마커를 시간 오름차순으로 반환합니다.

### FindSequenceMarkersByName

`(name: string) -> ({MarkerData})`

이름이 같은 시퀀스 전역 마커 목록을 반환합니다.

### FindBindingIdsByTag

`(tag: string) -> ({string})`

태그 기반 바인딩 대상 ID 목록을 반환합니다.
