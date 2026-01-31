# Editor Architecture

**Vide** 프레임워크를 기반으로 구축된 확장 가능한 에디터 아키텍처입니다.

## Core Systems

### Plugin System (Extensions)
UI 컴포넌트와 로직이 모듈화되어 있어 외부 플러그인 주입이 가능합니다.

*   **Custom Widget:** 특정 커스텀 타입(ColorSequence 등)을 위한 전용 프로퍼티 에디터.
*   **Tool Panels:** IK Solver, 자동 생성기 등을 사이드패널에 추가 가능.

### Viewport & Preview
*   **Realtime Applier:** 편집 중인 값을 즉시 인스턴스에 적용.
*   **Custom Preview Logic:** 단순 속성 변경 외에, 특수 효과나 CFrame 연산을 포함한 미리보기 로직 지원.

## Editor Metadata

애니메이션 파일 내부에 에디터 전용 컨텍스트 정보를 저장합니다.

#### LinkedRig
`ObjectValue`

작업 기준이 된 원본 Rig 모델을 참조합니다. (Blender의 Target Object 개념)
