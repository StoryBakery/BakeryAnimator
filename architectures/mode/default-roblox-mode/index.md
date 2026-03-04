---
title: DefaultRobloxMode
---

`DefaultRobloxMode`는 Roblox 기본 규칙(`Property`, `Attribute`, 경로 재해결)을
사용하는 기본 모드 구현입니다.

## 기본 예시

```lua
local DefaultRobloxMode = {}

DefaultRobloxMode.Manifest = {
    Id = "DefaultRoblox",
    DisplayName = "Default Roblox Mode",
    Version = "1.0.0",
    Priority = 0,
}

function DefaultRobloxMode.CanHandle(entryPoint, _context)
    return entryPoint:IsA("Model") or entryPoint:IsA("Folder")
end

function DefaultRobloxMode.DetectBindings(entryPoint, context)
    -- ReflectionService 기반 기본 바인딩 감지
    return {}, {}
end

function DefaultRobloxMode.DetectProperties(binding, context)
    -- Roblox 기본 Property + Attribute 규칙
    return {}, {}
end

function DefaultRobloxMode.DescribeBindingUi(binding, _context)
    return {
        DisplayName = binding.TrackName,
    }
end

function DefaultRobloxMode.DescribePropertyUi(_binding, channel, _context)
    return {
        DisplayName = channel.Name or channel.Key,
    }
end

return DefaultRobloxMode
```

## ModeMetadata 저장 위치

`DefaultRoblox` 모드의 메타데이터는 바인딩/바인딩그룹의
`Metadata/ModeMetadata/DefaultRoblox` 아래에 저장합니다.

## 저장 규칙

- 스칼라 값(`string`, `number`, `boolean`)은 `Folder`의 `Attribute`로 저장합니다.
- `StringValue`, `NumberValue`, `BoolValue` 같은 `ValueBase` 저장은 사용하지 않습니다.
- `Instance` 참조만 예외적으로 `ObjectValue`를 사용합니다.

## 권장 키

- `TargetPath` (`string`): 재해결 가능한 기준 경로
- `IncludeDescendants` (`boolean`): 하위 탐색 여부
- `FilterTag` (`string?`): 태그 기반 필터
- `TargetRef` (`ObjectValue`): 현재 해석된 캐시 참조

## 해석 우선순위

1. `TargetRef`가 유효하면 즉시 사용합니다.
1. `TargetRef`가 없거나 무효면 `TargetPath`로 재해결합니다.
1. 재해결 결과를 `TargetRef`에 다시 캐시합니다.
