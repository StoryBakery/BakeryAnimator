---
title: AnimationData
---

애니메이션의 "어떻게 움직여야 하는가"에 대한 정보만을 담으며, 런타임 상태를 가지지 않습니다.

## Constructors

### fromTable

`(data: table, params: AnimationDataParams?) -> (AnimationData)`

Raw 테이블 데이터로부터 AnimationData 인스턴스를 생성합니다.

### fromInstance

`(animationFile: AnimationFile, params: AnimationDataParams) -> (AnimationData)`

Instance로부터 AnimationData 인스턴스를 생성합니다.

## Properties

### Name

`string`

애니메이션의 이름입니다.
주로 가져와진 AnimationFile의 이름과 같습니다.

### FrameRate

`number`

초당 프레임 수 (FPS) 입니다.

### Looped

`boolean`

애니메이션 반복 재생 여부입니다.

### Priority

`number`

애니메이션의 우선순위입니다.

### Tags

`{string}`

애니메이션에 할당된 식별 태그 목록입니다.

### Duration

`number`

애니메이션의 총 길이 (초 단위) 입니다.

### BindingGroupAnimationDatasByKey

`{ [string]: BindingGroupAnimationData }`

이 애니메이션에 등장하는 바인딩 그룹 데이터 맵입니다.

- **Single BindingGroup**: `BindingGroupAnimationDatasByKey["Root"]`
- **Multi BindingGroup**
  - `BindingGroupAnimationDatasByKey["Hero"]`
  - `BindingGroupAnimationDatasByKey["Enemy"]`
  등 다수 존재.

바인딩 그룹을 사용하지 않는 경우에도 런타임에서 암묵적 `"Root"` 그룹으로 정규화합니다.

### Clips

`{ AudioClip | AnimationClip }`

이 데이터 내부에 포함된 하위 클립들입니다.

- **Global Level**: 전체 시퀀스 레벨에서 실행되는 클립 (예: 카메라, 환경 효과)
- **BindingGroup Level**: 특정 바인딩 그룹 전체에 적용되는 클립 (예: 걷기 사이클 파일 통째로 삽입)
- **Property Level**: 특정 속성 변화를 위해 삽입된 클립

```lua
type AnimationNotify = {
    Name: string,
    Time: number,
    Payload: table?,
}

type SyncMarker = {
    Name: string,
    Time: number,
}

type RootMotionData = {
    Enabled: boolean,
    Space: "BindingGroup" | "World",
}

type AnimationClip = {
    Type: "Animation" | "Audio",
    Data: AnimationData | ObjectValue<AnimationData>, -- Direct or Reference
    Scope: "Global" | "BindingGroup" | "Binding" | "Property", -- Where it applies
    StartTime: number,
    Duration: number,
    Speed: number,
    Weight: number,
    Offset: number, -- Start offset within the source clip
    Notifies: {AnimationNotify}?,
    SyncMarkers: {SyncMarker}?,
    RootMotion: RootMotionData?,
    Curves: {[string]: {Time: number, Value: number}}?,
}
```

## Methods

### BindingGroup Methods

#### GetBindingGroupData

`(bindingGroupKey: string?) -> (BindingGroupAnimationData?)`

특정 바인딩 그룹의 데이터를 가져옵니다.
`bindingGroupKey`가 없으면 기본 바인딩 그룹 데이터를 반환 시도합니다.
그룹 없는 데이터는 `"Root"`를 기본으로 반환합니다.

### IsSingleBindingGroup

`()->(boolean)`

바인딩 그룹이 하나인지 여부를 반환합니다.

### Track Methods

#### FindTrackData

`(bindingGroupKey: string, trackName: string) -> (table?)`

특정 바인딩 그룹의 특정 트랙 데이터를 반환합니다.
반환 트랙 데이터 내부는 `PropertyTrack -> Section -> Channel -> Keyframe` 구조를 따릅니다.

#### FindTrackDataByBindingId

`(bindingGroupKey: string, bindingId: string) -> (table?)`

`BindingId` 기준으로 트랙 데이터를 반환합니다.
`trackName` 변경이 있어도 안정적으로 바인딩할 수 있습니다.
반환 트랙 데이터 내부는 `PropertyTrack -> Section -> Channel -> Keyframe` 구조를 따릅니다.

`GetTrackData`도 있습니다.
