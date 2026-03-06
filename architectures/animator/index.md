---
title: Animator Architecture
---

애니메이터 플러그인을 어떻게 설계할지 다루는 문서입니다

## 인게임 애니메이터

인게임에서도 애니메이터로 사용가능하게 지원합니다.

공통 레이어 기준과 충돌 우선순위는 [Architecture Main](../main.md)을 따릅니다.

## Core Systems

### Editor Extension Surface

UI 컴포넌트와 편집 도구는 모듈화되어 있어 외부 편집기 플러그인 주입이 가능합니다.
이 계층은 `Adapter`와 다르게, 편집기 표시와 도구 확장을 담당합니다.

- **Custom Widget**: 특정 커스텀 타입(ColorSequence 등)을 위한 전용 프로퍼티 에디터.
- **Tool Panels**: IK Solver, 자동 생성기 등을 사이드패널에 추가 가능.

### Viewport & Preview

- **Realtime Applier**: 편집 중인 값을 즉시 인스턴스에 적용.
- **Custom Preview Logic**: 단순 속성 변경 외에, 특수 효과나 CFrame 연산을 포함한 미리보기 로직 지원.

### Visualization

뷰포트 표시 계층은 `visualization` 문서에서 관리합니다.
기본 제공 비주얼은 코어가 렌더링하고, 어댑터는 커스텀 비주얼을 오버레이로 확장합니다.

- [Visualization](./visualization/index.md)
- [본](./visualization/bone.md)

## Editor UX Strategy

"내부는 정밀하게, 겉은 단순하게"를 지향하는 UX 전략입니다.

### Scalar Grouping (Drill-Down)

복합 타입(`Vector3`, `CFrame`)은 내부적으로 스칼라 채널(`X`, `Y`, `Z`)로 분리되어 있지만,
사용자에겐 **하나의 그룹**으로 먼저 제시됩니다.

- **Default View**: `Position` 트랙 하나만 보입니다.
- **Expand**: 화살표를 눌러 `Position.X`, `Position.Y`, `Position.Z`를
  개별적으로 보고 편집할 수 있습니다.
- **Selection**: 그룹 선택 시 하위 채널 전체가 선택됩니다.

### Auto Keying & Batch Edit

- **Auto Keying**:
  사용자가 뷰포트에서 객체를 움직이거나 속성을 수정하면,
  해당 속성의 **모든 하위 채널**에 자동으로 키프레임이 생성됩니다.
  - 예: `Part`를 움직이면 `Position.X`, `Position.Y`, `Position.Z` 모두에 키가 찍힘.
- **Batch Interpolation**:
  그룹(상위 트랙)을 선택하고 보간 모드를 변경하면, 하위 모든 채널의 보간 모드가 일괄 변경됩니다.
