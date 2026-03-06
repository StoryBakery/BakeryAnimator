---
title: AdapterFactory
---

`AdapterFactory`는 어댑터 모듈을 런타임 표준 형태로 정규화합니다.

## 입력과 출력

```lua
(rawAdapter: RawAdapterModule, context: AdapterContext) -> (AdapterDefinition)
```

- 입력: 테이블 기반 어댑터 또는 생성자 기반 어댑터
- 출력: 항상 `AdapterDefinition`

## 정규화 규칙

1. `rawAdapter.new`가 있으면 `rawAdapter.new(context)`를 호출합니다.
1. 그렇지 않으면 `rawAdapter`를 `AdapterDefinition`으로 사용합니다.
1. `Manifest.Priority` 미지정 시 기본값 `0`을 채웁니다.
1. optional 훅은 `nil`을 유지하고, 런타임에서 존재 여부로 분기합니다.

## 예시

```lua
local function normalize(rawAdapter: RawAdapterModule, context: AdapterContext): AdapterDefinition
    local adapterDefinition: AdapterDefinition

    if rawAdapter.new ~= nil then
        adapterDefinition = rawAdapter.new(context)
    else
        adapterDefinition = rawAdapter
    end

    if adapterDefinition.Manifest.Priority == nil then
        adapterDefinition.Manifest.Priority = 0
    end

    return adapterDefinition
end
```
