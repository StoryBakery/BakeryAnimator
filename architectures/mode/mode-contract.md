---
title: ModeContract
---

`ModeContract`는 모드 시스템의 단일 타입 계약 모듈입니다.

## 역할

- `ModeDefinition`의 필수/선택 항목을 고정합니다.
- 생성자 기반 모드(`new`)와 테이블 기반 모드를 같은 타입으로 수용합니다.
- 레지스트리, 팩토리, 검증기가 같은 타입을 참조하도록 강제합니다.

## API

### ModeContext

모드 선택/감지/실행 전 과정에 전달되는 공통 실행 문맥입니다.
에디터/런타임 여부와 서비스 접근, 기능 플래그를 담습니다.

```lua
{
    IsEditor: boolean,
    IsRuntime: boolean,
    Services: {[string]: any},
    Features: {[string]: boolean},
}
```

### ModeDiagnostic

모드가 검출한 상태를 구조화해 반환하는 진단 데이터입니다.
예외를 던지는 대신 `Info`/`Warning`/`Error`로 누적 보고할 때 사용합니다.

```lua
{
    Level: "Info" | "Warning" | "Error",
    Code: string,
    Message: string,
    BindingId: string?,
    PropertyKey: string?,
}
```

### ModeBinding

감지된 대상 `Instance`를 트랙 바인딩 단위로 표현하는 데이터입니다.
`BindingId`는 안정 식별자, `TrackName`은 표시/보조 식별자 용도입니다.

```lua
{
    BindingId: string,
    TrackName: string,
    Instance: Instance,
    BindingGroupId: string?,
    ParentBindingId: string?,
    Metadata: {[string]: any}?,
}
```

### ModePropertyChannel

바인딩 하나에서 읽고 쓸 수 있는 속성 채널 계약입니다.
`Read`/`Write`는 필수이며, 변경 감지가 필요할 때만 `Observe`를 구현합니다.

```lua
{
    Key: string,
    Name: string?,
    ValueType: string,
    Read: (binding: ModeBinding, context: ModeContext) -> (any),
    Write: (binding: ModeBinding, value: any, context: ModeContext) -> (),
    Observe: ((binding: ModeBinding, onChanged: () -> (), context: ModeContext) -> (() -> ()))?,
}
```

### ModeDefinition

모드 구현체가 반드시 따라야 하는 핵심 계약입니다.
`CanHandle`로 모드 적용 가능 여부를 판단하고, `DetectBindings`/`DetectProperties`로
실제 바인딩 및 채널 구성을 제공합니다.

```lua
{
    Manifest: {
        Id: string,
        DisplayName: string,
        Version: string,
        Priority: number?,
    },
    CanHandle: (entryPoint: Instance, context: ModeContext) -> (boolean),
    DetectBindings: (entryPoint: Instance, context: ModeContext) -> ({ModeBinding}, {ModeDiagnostic}),
    DetectProperties: (binding: ModeBinding, context: ModeContext) -> ({ModePropertyChannel}, {ModeDiagnostic}),
    BuildRuntimeBehaviors: ((binding: ModeBinding, context: ModeContext) -> ({any}, {ModeDiagnostic}))?,
    SimulateEditorState: ((binding: ModeBinding, context: ModeContext) -> (table?))?,
}
```

### ModeConstructor

생성자 기반 모드 모듈의 형태입니다.
초기화 문맥을 받아 `ModeDefinition`을 생성합니다.

```lua
{
    new: (context: ModeContext) -> (ModeDefinition),
}
```

### RawModeModule

레지스트리가 로드할 수 있는 모듈 입력 형태입니다.
직접 구현한 `ModeDefinition` 또는 생성자 기반 `ModeConstructor`를 허용합니다.

`ModeDefinition | ModeConstructor`
