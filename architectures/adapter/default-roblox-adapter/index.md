---
title: DefaultRobloxAdapter
---

`DefaultRobloxAdapter`는 Roblox 기본 규칙(`Property`, `Attribute`, 경로 재해결)을
사용하는 기본 어댑터 구현입니다.

이 어댑터는 [Runtime](../../runtime/index.md) 문서에서 정의한
기본 런타임 템플릿과 가장 자연스럽게 맞물리도록 설계합니다.
다만 플러그인이 그 런타임 구현을 자동 삽입하는 것을 전제로 하지는 않습니다.
프로젝트는 같은 계약을 따르는 자체 런타임을 둘 수 있습니다.

## 기본 예시

```lua
local DefaultRobloxAdapter = {}

DefaultRobloxAdapter.Manifest = {
    Id = "DefaultRoblox",
    DisplayName = "Default Roblox Adapter",
    Version = "1.0.0",
    Priority = 0,
}

function DefaultRobloxAdapter.CanHandle(entryPoint, _context)
    return entryPoint:IsA("Model") or entryPoint:IsA("Folder")
end

function DefaultRobloxAdapter.DetectBindings(entryPoint, context)
    -- ReflectionService 기반 기본 바인딩 감지
    return {}, {}
end

function DefaultRobloxAdapter.DetectProperties(binding, context)
    -- Roblox 기본 Property + Attribute 규칙
    return {}, {}
end

function DefaultRobloxAdapter.DescribeBindingUi(binding, _context)
    return {
        DisplayName = binding.TrackName,
    }
end

function DefaultRobloxAdapter.DescribePropertyUi(_binding, channel, _context)
    return {
        DisplayName = channel.Name or channel.Key,
    }
end

function DefaultRobloxAdapter.BuildVisualizationPrimitives(_bindings, _context)
    -- 기본 본/선택 비주얼은 코어가 렌더링합니다.
    -- DefaultRoblox는 필요할 때만 추가 primitive를 반환합니다.
    return {}, {}
end

function DefaultRobloxAdapter.ResolveEventEndpoint(endpointId, _context)
    local endpoints = {
        ["Fx.EmitParticles"] = function(eventCall, context)
            local payload = eventCall.Payload or {}
            local targetBindingId = payload.TargetBindingId or eventCall.BindingId
            local bindingIndexById = context.BindingIndexById or {}
            if type(targetBindingId) ~= "string" then
                return
            end

            local binding = bindingIndexById[targetBindingId]
            if binding == nil then
                return
            end

            local emitterName = payload.EmitterName
            if type(emitterName) ~= "string" then
                return
            end

            local emitter = binding.Instance:FindFirstChild(emitterName, true)
            if emitter == nil or not emitter:IsA("ParticleEmitter") then
                return
            end

            local count = math.max(0, math.floor(payload.Count or 0))
            if count > 0 then
                emitter:Emit(count)
            end
        end,
        ["Fx.ClearParticles"] = function(eventCall, context)
            local payload = eventCall.Payload or {}
            local targetBindingId = payload.TargetBindingId or eventCall.BindingId
            local bindingIndexById = context.BindingIndexById or {}
            if type(targetBindingId) ~= "string" then
                return
            end

            local binding = bindingIndexById[targetBindingId]
            if binding == nil then
                return
            end

            local emitterName = payload.EmitterName
            if type(emitterName) ~= "string" then
                return
            end

            local emitter = binding.Instance:FindFirstChild(emitterName, true)
            if emitter ~= nil and emitter:IsA("ParticleEmitter") then
                emitter:Clear()
            end
        end,
    }

    return endpoints[endpointId], {}
end

function DefaultRobloxAdapter.ResolveAudioClip(clip, _context)
    -- AudioAssetId -> Roblox SoundId 해석, AttachBindingId -> Instance 연결
    return {
        AudioAssetId = clip.AudioAssetId,
    }, {}
end

return DefaultRobloxAdapter
```

## 시각화 처리

- 기본 `Bone`/선택 강조 비주얼은 코어 `Visualization` 계층이 담당합니다.
- `DefaultRobloxAdapter`는 필요 시 `BuildVisualizationPrimitives`에서
  프로젝트 특화 오버레이 primitive를 추가합니다.

## 함수 실행형 Event 처리

`DefaultRobloxAdapter`는 함수 실행형 이벤트를 `EventTrack -> EventKey`로 처리합니다.
키프레임 곡선으로는 실행하지 않습니다.

### 인식 입력

- `EventKey.EndpointId`: 실행할 함수 엔드포인트 키
- `EventKey.Payload`: JSON 문자열 payload
- `EventKey.ExecutionMode`: `Trigger` 또는 `Repeater`
- `EventKey.FireWhenForwards`, `EventKey.FireWhenBackwards`: 재생 방향별 발화 조건

### 실행 허용 목록

기본 어댑터는 allowlist 엔드포인트만 실행합니다.

- `Fx.EmitParticles`
- `Fx.ClearParticles`

allowlist에 없는 `EndpointId`는 실행하지 않고 진단만 누적합니다.

### 타깃 해석 규칙

1. `Payload.TargetBindingId`가 있으면 해당 바인딩을 우선 사용합니다.
1. 없으면 현재 이벤트가 속한 `BindingId`를 사용합니다.
1. `EmitterName` 기준으로 바인딩 인스턴스 하위에서 `ParticleEmitter`를 탐색합니다.
1. 타입 불일치나 탐색 실패 시 실행을 스킵하고 진단을 반환합니다.

### Payload 예시

```lua
-- Fx.EmitParticles
{
    TargetBindingId = "Muzzle",
    EmitterName = "MuzzleFlash",
    Count = 12,
}
```

## AdapterMetadata 저장 위치

`DefaultRoblox` 어댑터의 메타데이터는 바인딩/바인딩그룹
인스턴스 자체 `Attribute`에 저장합니다.
키 접두사는 `METADATA_DEFAULT_ROBLOX_`를 사용합니다.

## 저장 규칙

- 스칼라 값(`string`, `number`, `boolean`)은 `Attribute`로 저장합니다.
- `StringValue`, `NumberValue`, `BoolValue` 같은 `ValueBase` 저장은 사용하지 않습니다.
- 여러 속성을 JSON 하나로 합치지 않고, 속성별 `Attribute`로 분리합니다.
- `Instance` 참조만 예외적으로 동일 키 이름의 `ObjectValue`를 사용합니다.

## 권장 키

- `METADATA_DEFAULT_ROBLOX_TARGET_PATH` (`string`): 재해결 가능한 기준 경로
- `METADATA_DEFAULT_ROBLOX_INCLUDE_DESCENDANTS` (`boolean`): 하위 탐색 여부
- `METADATA_DEFAULT_ROBLOX_FILTER_TAG` (`string?`): 태그 기반 필터
- `METADATA_DEFAULT_ROBLOX_TARGET_REF` (`ObjectValue`): 현재 해석된 캐시 참조

## 해석 우선순위

1. `METADATA_DEFAULT_ROBLOX_TARGET_REF`가 유효하면 즉시 사용합니다.
1. `METADATA_DEFAULT_ROBLOX_TARGET_REF`가 없거나 무효면
   `METADATA_DEFAULT_ROBLOX_TARGET_PATH`로 재해결합니다.
1. 재해결 결과를 `METADATA_DEFAULT_ROBLOX_TARGET_REF`에 다시 캐시합니다.
