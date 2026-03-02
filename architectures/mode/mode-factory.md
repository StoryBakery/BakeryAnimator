---
title: ModeFactory
---

`ModeFactory`는 모드 모듈을 런타임 표준 형태로 정규화합니다.

## 입력과 출력

```lua
(rawMode: RawModeModule, context: ModeContext) -> (ModeDefinition)
```

- 입력: 테이블 기반 모드 또는 생성자 기반 모드
- 출력: 항상 `ModeDefinition`

## 정규화 규칙

1. `rawMode.new`가 있으면 `rawMode.new(context)`를 호출합니다.
1. 그렇지 않으면 `rawMode`를 `ModeDefinition`으로 사용합니다.
1. `Manifest.Priority` 미지정 시 기본값 `0`을 채웁니다.
1. optional 훅은 `nil`을 유지하고, 런타임에서 존재 여부로 분기합니다.

## 예시

```lua
local function normalize(rawMode: RawModeModule, context: ModeContext): ModeDefinition
    local modeDefinition: ModeDefinition

    if rawMode.new ~= nil then
        modeDefinition = rawMode.new(context)
    else
        modeDefinition = rawMode
    end

    if modeDefinition.Manifest.Priority == nil then
        modeDefinition.Manifest.Priority = 0
    end

    return modeDefinition
end
```
