---
title: AdapterContract
---

`AdapterContract`는 어댑터 시스템의 단일 타입 계약 모듈입니다.

## 역할

- `AdapterDefinition`의 필수/선택 항목을 고정합니다.
- 생성자 기반 어댑터(`new`)와 테이블 기반 어댑터를 같은 타입으로 수용합니다.
- 레지스트리, 팩토리, 검증기가 같은 타입을 참조하도록 강제합니다.
- 어댑터별 UI 표시 보정 계약(`DescribeBindingUi`, `DescribePropertyUi`)을 고정합니다.
- 오디오 클립 해석 계약(`ResolveAudioClip`)을 고정합니다.
- 뷰포트 커스텀 비주얼 계약(`BuildVisualizationPrimitives`)을 고정합니다.

## API

### AdapterContext

어댑터 선택/감지/실행 전 과정에 전달되는 공통 실행 문맥입니다.
에디터/런타임 여부와 서비스 접근, 기능 플래그를 담습니다.
`BindingIndexById`는 런타임에서 `BindingId -> AdapterBinding` 재조회에 사용합니다.

```lua
{
    IsEditor: boolean,
    IsRuntime: boolean,
    Services: {[string]: any},
    Features: {[string]: boolean},
    BindingIndexById: {[string]: any}?,
}
```

### AdapterDiagnostic

어댑터가 검출한 상태를 구조화해 반환하는 진단 데이터입니다.
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

### AdapterBinding

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

### AdapterPropertyChannel

바인딩 하나에서 읽고 쓸 수 있는 속성 채널 계약입니다.
`Read`/`Write`는 필수이며, 변경 감지가 필요할 때만 `Observe`를 구현합니다.

```lua
{
    Key: string,
    Name: string?,
    ValueType: string,
    Read: (binding: AdapterBinding, context: AdapterContext) -> (any),
    Write: (binding: AdapterBinding, value: any, context: AdapterContext) -> (),
    Observe: ((binding: AdapterBinding, onChanged: () -> (), context: AdapterContext) -> (() -> ()))?,
}
```

### AdapterBindingUi

바인딩 트리/아웃라이너 표시를 어댑터가 보정할 때 사용하는 UI 메타데이터입니다.
표시 이름, 정렬, 숨김 여부 같은 편집기 표시 정보만 다루며 실행 로직은 바꾸지 않습니다.

```lua
{
    DisplayName: string?,
    Description: string?,
    Group: string?,
    Order: number?,
    Hidden: boolean?,
}
```

### AdapterPropertyUi

프로퍼티 패널 표시를 어댑터가 보정할 때 사용하는 UI 메타데이터입니다.
라벨, 단위, 읽기 전용 여부 같은 편집기 표시 정보를 제공합니다.

```lua
{
    DisplayName: string?,
    Description: string?,
    Group: string?,
    Unit: string?,
    Order: number?,
    Hidden: boolean?,
    ReadOnly: boolean?,
}
```

### AdapterVisualPrimitive

어댑터가 뷰포트에 추가할 시각화 primitive 단위입니다.
코어는 이 primitive 목록을 기본 비주얼 위에 합성합니다.
`AdapterVisualPrimitive`는 가상 데이터 계약이며,
저장/동기화 대상 `Instance` 트리를 직접 나타내지 않습니다.

```lua
{
    PrimitiveId: string,
    PrimitiveType: "Line" | "Sphere" | "Cone" | "Box" | "Mesh",
    BindingId: string?,
    Transform: CFrame?,
    Size: Vector3?,
    Color: Color3?,
    Transparency: number?,
    Thickness: number?,
    AlwaysOnTop: boolean?,
    ZIndex: number?,
    Metadata: {[string]: any}?,
}
```

### AdapterEventEndpoint

이벤트 트랙이 호출할 엔드포인트 함수 타입입니다.
`EventTrack`의 엔드포인트 ID를 런타임 함수로 해석할 때 사용합니다.

`(eventCall: AdapterEventCall, context: AdapterContext) -> ()`

### AdapterEventCall

이벤트 디스패처가 엔드포인트에 전달하는 호출 데이터입니다.
실행 시점, 방향, 페이로드를 함께 전달합니다.

```lua
{
    EndpointId: string,
    BindingId: string?,
    Payload: table?,
    TimePosition: number,
    Direction: "Forward" | "Backward",
    IsScrubbing: boolean?,
}
```

### AdapterAudioClip

오디오 트랙 섹션에서 런타임으로 전달되는 오디오 재생 요청 데이터입니다.
어댑터는 이 정보를 바탕으로 에셋 해석/붙이기(Attach)/메타 보정을 수행할 수 있습니다.

```lua
{
    AudioAssetId: string,
    BindingId: string?,
    AttachBindingId: string?,
    StartOffsetSeconds: number?,
    Metadata: {[string]: any}?,
}
```

### AdapterResolvedAudioClip

어댑터가 해석한 오디오 재생 대상입니다.
코어 런타임은 이 구조를 사용해 실제 오디오 재생을 실행합니다.
`ResolveAudioClip`이 `nil`을 반환하면 해당 섹션 오디오는 스킵하고 진단만 누적합니다.

```lua
{
    AudioAssetId: string,
    SoundInstance: Instance?,
    AttachInstance: Instance?,
    Metadata: {[string]: any}?,
}
```

### AdapterDefinition

어댑터 구현체가 반드시 따라야 하는 핵심 계약입니다.
`CanHandle`로 어댑터 적용 가능 여부를 판단하고, `DetectBindings`/`DetectProperties`로
실제 바인딩 및 채널 구성을 제공합니다.

```lua
{
    Manifest: {
        Id: string,
        DisplayName: string,
        Version: string,
        Priority: number?,
    },
    CanHandle: (entryPoint: Instance, context: AdapterContext) -> (boolean),
    DetectBindings: (entryPoint: Instance, context: AdapterContext) -> ({AdapterBinding}, {AdapterDiagnostic}),
    DetectProperties: (binding: AdapterBinding, context: AdapterContext) -> ({AdapterPropertyChannel}, {AdapterDiagnostic}),
    DescribeBindingUi: ((binding: AdapterBinding, context: AdapterContext) -> (AdapterBindingUi?))?,
    DescribePropertyUi: ((binding: AdapterBinding, channel: AdapterPropertyChannel, context: AdapterContext) -> (AdapterPropertyUi?))?,
    BuildVisualizationPrimitives: ((bindings: {AdapterBinding}, context: AdapterContext) -> ({AdapterVisualPrimitive}, {AdapterDiagnostic}))?,
    BuildRuntimeBehaviors: ((binding: AdapterBinding, context: AdapterContext) -> ({any}, {AdapterDiagnostic}))?,
    SimulateEditorState: ((binding: AdapterBinding, context: AdapterContext) -> (table?))?,
    ResolveBindingTag: ((tag: string, context: AdapterContext) -> ({AdapterBinding}, {AdapterDiagnostic}))?,
    ResolveEventEndpoint: ((endpointId: string, context: AdapterContext) -> (AdapterEventEndpoint?, {AdapterDiagnostic}))?,
    ResolveAudioClip: ((clip: AdapterAudioClip, context: AdapterContext) -> (AdapterResolvedAudioClip?, {AdapterDiagnostic}))?,
}
```

### AdapterConstructor

생성자 기반 어댑터 모듈의 형태입니다.
초기화 문맥을 받아 `AdapterDefinition`을 생성합니다.

```lua
{
    new: (context: AdapterContext) -> (AdapterDefinition),
}
```

### RawAdapterModule

레지스트리가 로드할 수 있는 모듈 입력 형태입니다.
직접 구현한 `AdapterDefinition` 또는 생성자 기반 `AdapterConstructor`를 허용합니다.

`AdapterDefinition | AdapterConstructor`
