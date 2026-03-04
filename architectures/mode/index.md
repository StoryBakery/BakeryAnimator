---
title: Mode System
---

사용자가 인스턴스 감지 방식, 속성 감지 방식, 런타임 동작 방식을 선택하거나 교체할 수 있도록 하는 시스템입니다.

## 목표

- 기본 Roblox 의존 모드를 제공합니다.
- 기본 모드와 제3자 모드를 같은 위치, 같은 계약으로 로드합니다.
- `Property`, `Attribute`, `ValueObject`, 스크립트 상태 기반 감지를 모두 지원합니다.
- 런타임 전용 커스텀 동작도 모드 훅으로 확장합니다.

## 레이어 기준

- 에디터 UI/UX는 Blender 기준을 따릅니다.
- 데이터 계층은 Unreal Engine 시퀀스 계층을 따릅니다.
- 모드 시스템을 포함한 런타임 동작은 Roblox 실행 제약을 우선합니다.

충돌 시 우선순위:

1. Roblox 실행 가능성
1. 데이터 구조 일관성
1. 에디터 UX 편의성

## 핵심 원칙

### Default Is Just Another Mode

기본 모드는 엔진 내부 특례가 아니라 `ModeDefinition`을 구현한 일반 `ModuleScript`입니다.
코어는 모드 ID를 선택하고 인터페이스만 호출합니다.

### Same Registry, Same Loading Path

기본 모드와 제3자 모드는 동일한 `ModeRegistry`에 등록합니다.
별도 하드코드 분기 없이 동일한 우선순위 규칙으로 선택합니다.

### Engine Dependency Out Of Core

ReflectionService, Roblox 기본 `Property` 규칙은 `DefaultRobloxMode` 내부에 둡니다.
코어 런타임은 모드가 반환한 바인딩/채널 정보만 사용합니다.

### Optional BindingGroup

`BindingGroup`은 선택 계층입니다.
특수 케이스에서는 그룹 없이 `Binding`을 루트에 직접 둘 수 있어야 합니다.

## 폴더 구조

```text
ModeCore/
|- ModeContract.luau
|- ModeFactory.luau
|- ModeValidator.luau

Modes/
|- DefaultRoblox.mode.luau
|- ProjectGameplay.mode.luau
|- ThirdPartyCombat.mode.luau
```

기본 모드와 제3자 모드를 동일 폴더와 동일 로딩 파이프라인에서 처리합니다.

상세 설계:

- [ModeContract](./mode-contract.md)
- [ModeFactory](./mode-factory.md)
- [ModeValidator](./mode-validator.md)
- [DefaultRobloxMode](./default-roblox-mode/index.md)

## Mode Contract

```lua
type ModeContext = {
    IsEditor: boolean,
    IsRuntime: boolean,
    Services: {[string]: any},
    Features: {[string]: boolean},
}

type ModeDiagnostic = {
    Level: "Info" | "Warning" | "Error",
    Code: string,
    Message: string,
    BindingId: string?,
    PropertyKey: string?,
}

type ModeBinding = {
    BindingId: string,
    TrackName: string,
    Instance: Instance,
    BindingGroupId: string?,
    ParentBindingId: string?,
    Metadata: {[string]: any}?,
}

type ModePropertyChannel = {
    Key: string,
    Name: string?,
    ValueType: string,
    Read: (binding: ModeBinding, context: ModeContext) -> (any),
    Write: (binding: ModeBinding, value: any, context: ModeContext) -> (),
    Observe: ((binding: ModeBinding, onChanged: () -> (), context: ModeContext) -> (() -> ()))?,
}

type ModeBindingUi = {
    DisplayName: string?,
    Description: string?,
    Group: string?,
    Order: number?,
    Hidden: boolean?,
}

type ModePropertyUi = {
    DisplayName: string?,
    Description: string?,
    Group: string?,
    Unit: string?,
    Order: number?,
    Hidden: boolean?,
    ReadOnly: boolean?,
}

type ModeEventEndpoint = (payload: table?) -> ()

type ModeDefinition = {
    Manifest: {
        Id: string,
        DisplayName: string,
        Version: string,
        Priority: number?,
    },
    CanHandle: (entryPoint: Instance, context: ModeContext) -> (boolean),
    DetectBindings: (entryPoint: Instance, context: ModeContext) -> ({ModeBinding}, {ModeDiagnostic}),
    DetectProperties: (binding: ModeBinding, context: ModeContext) -> ({ModePropertyChannel}, {ModeDiagnostic}),
    DescribeBindingUi: ((binding: ModeBinding, context: ModeContext) -> (ModeBindingUi?))?,
    DescribePropertyUi: ((binding: ModeBinding, channel: ModePropertyChannel, context: ModeContext) -> (ModePropertyUi?))?,
    BuildRuntimeBehaviors: ((binding: ModeBinding, context: ModeContext) -> ({any}, {ModeDiagnostic}))?,
    SimulateEditorState: ((binding: ModeBinding, context: ModeContext) -> (table?))?,
    ResolveBindingTag: ((tag: string, context: ModeContext) -> ({ModeBinding}, {ModeDiagnostic}))?,
    ResolveEventEndpoint: ((endpointId: string, context: ModeContext) -> (ModeEventEndpoint?, {ModeDiagnostic}))?,
}

type ModeConstructor = {
    new: (context: ModeContext) -> (ModeDefinition),
}

type RawModeModule = ModeDefinition | ModeConstructor
```

## ModeCore Modules

### ModeContract

`ModeDefinition`/`ModeConstructor`/`RawModeModule` 타입 계약을 단일 모듈에서 제공합니다.
모드 구현 모듈과 레지스트리가 동일 타입 소스를 참조하도록 고정합니다.

### ModeFactory

모드 로드 시 `RawModeModule`을 `ModeDefinition`으로 정규화합니다.

```lua
-- ModeFactory.normalize(rawMode, context)
if rawMode.new ~= nil then
    return rawMode.new(context)
end

return rawMode
```

`ModeFactory`에서 기본값(`Manifest.Priority`, optional 훅 기본 처리)을 채워
런타임 분기 비용을 줄입니다.

### ModeValidator

등록 전 정적 계약을 검증합니다.

- 필수 필드: `Manifest.Id`, `Manifest.DisplayName`, `Manifest.Version`
- 필수 함수: `CanHandle`, `DetectBindings`, `DetectProperties`
- 선택 함수 반환 형식: diagnostics 배열 규약 준수
- 중복 ID 충돌 검사

```lua
type ValidationResult = {
    IsValid: boolean,
    Diagnostics: {ModeDiagnostic},
}
```

## 실행 파이프라인

1. `ModeRegistry`가 `Modes/` 아래 모듈을 모두 로드합니다.
1. 각 모듈을 `ModeFactory`로 정규화합니다.
1. `ModeValidator`로 검증 후 통과한 모드만 등록합니다.
1. 사용자가 명시한 모드 ID가 있으면 우선 사용합니다.
1. 명시가 없으면 `CanHandle` + `Manifest.Priority`로 모드를 선택합니다.
1. 선택된 모드의 `DetectBindings`, `DetectProperties`로 트랙 매핑을 구성합니다.
1. 에디터에서는 `DescribeBindingUi`, `DescribePropertyUi` 결과를 반영해 표시를 보정합니다.
1. 태그 기반 바인딩이 있으면 `ResolveBindingTag`로 런타임 대상 재해결을 수행합니다.
1. 이벤트 트랙이 있으면 `ResolveEventEndpoint`로 엔드포인트를 연결합니다.
1. `Observe`를 구독해 변경 발생 시 해당 바인딩만 부분 재평가합니다.
1. `BuildRuntimeBehaviors`가 있으면 런타임 훅을 연결합니다.

## UI 표시 확장 정책

- 모드는 `DescribeBindingUi`, `DescribePropertyUi`로 표시 텍스트/정렬/숨김을 보정할 수 있습니다.
- UI 훅은 편집기 표시만 바꾸며, 감지/평가/적용 로직은 바꾸지 않습니다.
- UI 훅이 없으면 코어 기본 표시 규칙을 사용합니다.

## 계층 구조

기본 계층은 다음과 같습니다.

```text
LevelSequence
|- BindingGroup? (optional)
|  |- Binding
|     |- PropertyTrack
|        |- Section
|           |- Channel
|              |- Keyframe
|     |- EventTrack
|        |- Section
|           |- EventKey
|     |- ConstraintTrack
|        |- Section
|           |- ConstraintKey
```

`BindingGroup`이 없는 경우:

```text
LevelSequence
|- Binding
|  |- PropertyTrack
|     |- Section
|        |- Channel
|           |- Keyframe
|  |- EventTrack
|     |- Section
|        |- EventKey
|  |- ConstraintTrack
|     |- Section
|        |- ConstraintKey
```

### Section 계층 규약

- `Section`은 `PropertyTrack` 내부의 시간 구간 단위입니다.
- 각 `Section`은 시작/종료 시간과 블렌드 규칙을 가지며, 그 아래에 `Channel`과 `Keyframe`을 포함합니다.
- 같은 `PropertyTrack` 안에서 `Section`이 겹치면 섹션 블렌드 정책으로 합성합니다.
- `EventTrack`과 `ConstraintTrack`도 동일하게 `Section` 단위로 평가합니다.

## 모드 선택 규칙

1. 사용자 명시 선택값 (`ModeId`)이 있으면 해당 모드를 강제 사용합니다.
1. 없으면 엔트리 포인트의 `AnimatorModeId` `Attribute`를 확인합니다.
1. 그래도 없으면 `CanHandle=true` 모드 중 최고 우선순위를 선택합니다.
1. 후보가 없으면 `DefaultRoblox`로 폴백합니다.

## 스크립트 상태 기반 감지

`ScriptState`는 스튜디오 인스펙터에서 직접 보이지 않을 수 있으므로,
모드가 다음 훅을 제공하도록 권장합니다.

- `Observe`: 런타임 상태 변경 감지
- `SimulateEditorState`: 에디터에서 가능한 범위의 미리보기 상태 계산

이 계약을 사용하면 `Attribute`/`ValueObject`에 의존하지 않는 규칙도
동일한 파이프라인에서 다룰 수 있습니다.

## 기본 모드 상세

기본 모드 구현 예시와 `ModeMetadata` 저장 규칙은
[DefaultRobloxMode](./default-roblox-mode/index.md) 문서에서 관리합니다.

## 권장 정책

- `BindingId`를 1순위 식별자로 사용하고 `TrackName`은 표시용 보조 키로 유지합니다.
- `BindingGroupId`는 선택값으로 취급하고, 없는 경우 루트 바인딩으로 처리합니다.
- 모드 구현은 `throw` 대신 `ModeDiagnostic`을 반환합니다.
- 모드 모듈은 테이블 직접 반환 또는 `new()` 생성자 반환 중 하나로 통일합니다.
- 느린 모드는 `DetectProperties` 결과를 캐시하고 부분 무효화 전략을 사용합니다.
