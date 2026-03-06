---
title: Section Evaluator
---

`Section Evaluator`는 `PropertyTrack` 내부의 활성 `Section`들을 평가하고
최종 속성 값을 계산하는 런타임 모듈입니다.

## 역할

- 현재 시간에서 활성 `Section` 후보를 수집합니다.
- 각 섹션의 `Channel`을 샘플링해 구간 값을 계산합니다.
- 섹션 경계 `EaseIn`/`EaseOut`과 블렌드 타입을 반영해 합성합니다.
- 최종 값을 `Applier` 단계로 전달합니다.

## 평가 단위

### PropertyTrack

속성 하나(`Position`, `Transparency` 등)에 대한 상위 컨테이너입니다.

### Section

시간 구간 단위입니다.
`StartTime`, `EndTime`, `PreRollFrames`, `PostRollFrames`, `BlendType`을 가집니다.

### Channel

실제 키프레임 곡선입니다.
스칼라 분해(`X`, `Y`, `Z`)된 채널을 개별 평가합니다.

## 처리 순서

1. 활성 섹션을 수집합니다.
1. `RowIndex`, `OverlapPriority` 기준으로 섹션 순서를 정렬합니다.
1. 각 섹션의 채널 값을 샘플링합니다.
1. 섹션 경계 `EaseIn`/`EaseOut` 가중치를 계산합니다.
1. `BlendType`(`Absolute`/`Additive`/`Relative`) 규칙으로 합성합니다.
1. 속성 최종 값을 반환합니다.

## 활성 섹션 판정

### 기본 구간

`StartTime <= time <= EndTime`이면 활성입니다.
`EndTime`이 없으면 무한 구간으로 처리합니다.

### 롤 구간

`PreRollFrames`, `PostRollFrames`가 지정된 경우
해당 프레임 범위를 평가 후보에 포함할 수 있습니다.

## 키프레임 샘플링

`Channel` 샘플링은 키 사이 구간 보간으로 계산합니다.

- `Constant`: 이전 키 값을 유지합니다.
- `Linear`: 두 키를 선형 보간합니다.
- `Bezier`: 블렌더 방식의 핸들 기반 보간을 사용합니다.

`Bezier`를 기본 보간으로 사용하는 경우, 키프레임에 핸들 정보가 없으면
자동 핸들 정책(`Auto` 또는 `AutoClamped`)으로 계산합니다.

## 섹션 경계 블렌드

`EaseIn`/`EaseOut`은 **섹션 진입/이탈 가중치** 계산용입니다.
키프레임 보간 모드(`Bezier` 등)와 역할이 다릅니다.

- `EaseIn`: 섹션 시작 근처에서 가중치를 0에서 1로 올립니다.
- `EaseOut`: 섹션 종료 근처에서 가중치를 1에서 0으로 내립니다.

최종 섹션 가중치:

```lua
sectionWeight = easeInWeight * easeOutWeight
```

## 섹션 합성 규칙

### Absolute

절대값 트랙입니다.
여러 섹션이 겹치면 가중치 정규화 후 혼합합니다.

### Additive

기준값 위에 델타를 더합니다.

### Relative

기준 포즈/기준값 대비 상대값을 적용합니다.

## 의사코드

```lua
local function evaluatePropertyTrack(propertyTrack, time, referenceValue)
    local sections = collectActiveSections(propertyTrack.Sections, time)
    sortSections(sections) -- RowIndex, OverlapPriority

    local absoluteSamples = {}
    local additiveDelta = 0
    local relativeDelta = 0

    for _, section in sections do
        local sampleValue = sampleSectionChannels(section, time) -- Constant/Linear/Bezier
        local weight = computeSectionWeight(section, time) -- EaseIn/EaseOut

        if section.BlendType == "Absolute" then
            table.insert(absoluteSamples, {Value = sampleValue, Weight = weight})
        elseif section.BlendType == "Additive" then
            additiveDelta += sampleValue * weight
        else -- Relative
            relativeDelta += (sampleValue - referenceValue) * weight
        end
    end

    local absoluteValue = blendAbsoluteSamples(absoluteSamples, referenceValue)
    return absoluteValue + additiveDelta + relativeDelta
end
```

## 출력 계약

- 입력: `PropertyTrackData`, `time`, `referenceValue`
- 출력: 최종 속성 값(`any`)

평가 실패(유효 키 없음, 타입 불일치)는 진단을 남기고
`referenceValue` 또는 채널 기본값으로 폴백합니다.
