---
title: Layout
---

단일 `EditorShell` 안에서 사용하는 기본 화면 구성을 정의합니다.

## 레이아웃 영역

- `TopBar`: 파일, 편집, 재생, 스냅, 프로필 전환
- `Outliner`: `BindingGroup/Binding/Track` 트리 탐색
- `Timeline`: 섹션 중심 편집(트림, 분할, 이동, 오버랩)
- `GraphEditor`: 채널/키프레임 커브 편집
- `Inspector`: 선택 항목 상세 속성
- `Viewport`: 선택/변형/시각화 오버레이
- `Panels`: IK, 자동 키, 어댑터 툴, 디버그

## 패널 규칙

- 패널은 도킹/접기/분리 가능한 상태를 지원합니다.
- 같은 기능을 중복 패널로 제공하지 않습니다.
- 고급 옵션은 기본 접힘 상태를 유지합니다.

## 프로필별 기본 구성

- `Animate`: `Timeline` + `GraphEditor` 비중을 높입니다.
- `Rig`: `Viewport` + `Inspector` 비중을 높입니다.
- `Sequencer`: `Outliner` + `Timeline` 비중을 높입니다.

`GraphEditor` 세부 기준은 [Graph Editor](./graph-editor.md)를 따릅니다.
