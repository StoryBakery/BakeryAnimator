---
title: BindingGroupAnimationData
---

단일 `bindingGroupKey`에 대응되는 애니메이션 데이터 묶음입니다.

- 특정 바인딩 그룹에 속한 트랙 데이터를 모읍니다.
- 바인딩 그룹 단위와 그 하위 범위에 적용되는 클립들을 함께 보관합니다.
- 하나의 파일이 여러 바인딩 그룹을 동시에 제어할 때 기준점 역할을 합니다.

## 구조

```lua
AnimationData
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
```

## 포함하는 데이터

- **Track Data**: 바인딩 그룹 내부의 각 바인딩별 애니메이션 곡선 데이터입니다.
  런타임에서는 `BindingId`를 1순위 키로 사용하고, `trackName`은 보조 키로 사용합니다.
- **PropertyTrack Data**: 속성 단위 트랙 컨테이너입니다.
- **Section Data**: 속성 트랙 내부의 시간 구간 단위입니다.
  같은 속성에서 여러 섹션을 구간/겹침 규칙으로 조합할 수 있습니다.
- **BindingGroup Level Clip**: 바인딩 그룹 전체에 적용되는 하위 클립입니다.
- **Binding Level Clip**: 바인딩 그룹 내부의 특정 바인딩을 대상으로 하는 하위 클립입니다.
- **Property Level Clip**: 특정 Property 변화만을 위한 세밀한 하위 클립입니다.

```lua
type SectionData = {
    SectionId: string,
    StartTime: number,
    EndTime: number?,
    BlendType: "Absolute" | "Additive" | "Relative",
    Channels: {[string]: table},
}

type PropertyTrackData = {
    PropertyKey: string,
    Sections: {SectionData},
}

type TrackData = {
    TrackName: string,
    BindingId: string,
    PropertyTracksByKey: {[string]: PropertyTrackData},
}
```

## 포함하지 않는 데이터

`BindingGroupAnimationData`는 정적 데이터만 다룹니다.
현재 시간, 재생 상태, Weight, BlendMode 같은 런타임 상태는 가지지 않으며,
이런 값은 `AnimationTrack`과 `LevelSequence`가 관리합니다.

## 바인딩 예시

- **Single BindingGroup**: `AnimationData.BindingGroupAnimationDatasByKey["Root"]`
- **Multi BindingGroup**: `AnimationData.BindingGroupAnimationDatasByKey["Hero"]`,
  `AnimationData.BindingGroupAnimationDatasByKey["Enemy"]`

그룹 없는 바인딩 구성은 암묵적 `"Root"` 그룹으로 정규화되어 저장됩니다.

즉, `AnimationData` 가 파일 전체 컨테이너라면,
`BindingGroupAnimationData`는 그 내부에서 바인딩 그룹 하나를 위한 서브 컨테이너입니다.
