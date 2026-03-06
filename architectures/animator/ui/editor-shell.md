---
title: Editor Shell
---

에디터 코어 구조와 시퀀서 책임 경계를 정의합니다.

## Single EditorShell

- `EditorShell` 인스턴스는 하나만 유지합니다.
- 공통 기능은 `EditorShell`이 담당합니다.
  - Undo/Redo
  - 선택 상태
  - 타임라인 재생 제어
  - 저장/로드
- 기능 분기는 별도 에디터를 늘리지 않고 `EditorProfile`로 처리합니다.

## EditorProfile

- 프로필은 레이아웃/기본 패널/기본 툴 세트를 정의합니다.
- 프로필 전환 시 데이터 모델은 유지하고 표시와 도구 구성만 변경합니다.
- `Animate` 프로필은 `Timeline`과 `GraphEditor`를 함께 제공해
  키 타이밍 편집과 커브 편집을 분리합니다.
- 기본 프로필:
  - `Animate`
  - `Rig`
  - `Sequencer`

## 단축키 정책

- 기본 단축키 맵은 Blender 기준을 우선합니다.
- Roblox 스튜디오 고유 단축키와 충돌하는 경우, Roblox 단축키를 우선 유지합니다.
- 충돌한 단축키는 대체 조합으로 재배치하고, 키맵 문서에 명시합니다.

## 시퀀서 책임 분리

- `LevelSequence`는 전체 타임라인 디렉터입니다.
  - 단일 `MasterClock` 기준으로 하위 트랙을 동기화합니다.
  - 바인딩 재해결, 마커 동기화, 계층 시퀀스 우선순위를 관리합니다.
- `AnimationTrack`은 바인딩 단위 실행기입니다.
  - 활성 `Section`을 평가하고 최종 속성 값을 계산합니다.
  - `Property`, `Event`, `Audio`, `Constraint` 트랙 적용을 수행합니다.

상세 규약:

- [LevelSequence](../../runtime/animation-track/level-sequence.md)
- [AnimationTrack](../../runtime/animation-track/index.md)
- [Section Evaluator](../../runtime/animation-track/section-evaluator.md)
