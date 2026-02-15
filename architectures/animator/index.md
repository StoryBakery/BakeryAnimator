## 개요

**Vide** 프레임워크를 기반으로 구축된 확장 가능한 에디터 아키텍처입니다.

## Core Systems

### Plugin System (Extensions)
UI 컴포넌트와 로직이 모듈화되어 있어 외부 플러그인 주입이 가능합니다.

*   **Custom Widget:** 특정 커스텀 타입(ColorSequence 등)을 위한 전용 프로퍼티 에디터.
*   **Tool Panels:** IK Solver, 자동 생성기 등을 사이드패널에 추가 가능.

### Viewport & Preview
*   **Realtime Applier:** 편집 중인 값을 즉시 인스턴스에 적용.
*   **Custom Preview Logic:** 단순 속성 변경 외에, 특수 효과나 CFrame 연산을 포함한 미리보기 로직 지원.

## Editor UX Strategy

"내부는 정밀하게, 겉은 단순하게"를 지향하는 UX 전략입니다.

### Scalar Grouping (Drill-Down)
복합 타입(`Vector3`, `CFrame`)은 내부적으로 스칼라 채널(`X`, `Y`, `Z`)로 분리되어 있지만, 사용자에겐 **하나의 그룹**으로 먼저 제시됩니다.

*   **Default View:** `Position` 트랙 하나만 보입니다.
*   **Expand:** 화살표를 눌러 `Position.X`, `Position.Y`, `Position.Z`를 개별적으로 보고 편집할 수 있습니다.
*   **Selection:** 그룹 선택 시 하위 채널 전체가 선택됩니다.

### Auto Keying & Batch Edit
*   **Auto Keying:** 사용자가 뷰포트에서 객체를 움직이거나 속성을 수정하면, 해당 속성의 **모든 하위 채널**에 자동으로 키프레임이 생성됩니다.
    *   예: `Part`를 움직이면 `Position.X`, `Position.Y`, `Position.Z` 모두에 키가 찍힘.
*   **Batch Interpolation:** 그룹(상위 트랙)을 선택하고 보간 모드를 변경하면, 하위 모든 채널의 보간 모드가 일괄 변경됩니다.

## Editor Metadata

애니메이션 파일 내부에 에디터 전용 컨텍스트 정보를 저장합니다.

#### LinkedRig
`ObjectValue`

작업 기준이 된 원본 Rig 모델을 참조합니다. (Blender의 Target Object 개념)
