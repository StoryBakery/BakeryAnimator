---
title: UI
---

애니메이터 UI는 단일 에디터 코어를 기준으로 설계합니다.
여러 독립 에디터를 나누지 않고, 하나의 에디터에서 프로필/패널 구성을 바꿔 워크플로우를 분기합니다.

## 문서 구성

- [Editor Shell](./editor-shell.md)
- [레이아웃](./layout.md)
- [Graph Editor](./graph-editor.md)
- [편집 워크플로우](./editing-workflow.md)
- [어댑터 UI 확장](./adapter-ui-extension.md)
- [상태와 성능](./state-and-performance.md)

## 구현 기준

- UI 프레임워크는 `bide`를 기본으로 사용합니다.
- 데이터 진실 원천은 [Animator Instances](../instances/instances.md)입니다.
- 뷰포트 표시는 [Visualization](../visualization/index.md) 계층을 사용합니다.

## 코어 원칙

- `LevelSequence`와 `AnimationTrack`의 책임을 분리합니다.
- 편집 기본 단위는 `Track`이 아니라 `Section`으로 유지합니다.
- `Timeline`(타이밍)과 `GraphEditor`(커브)를 분리해 함께 제공합니다.
- 단축키는 Roblox 고유 단축키와 충돌하지 않는 범위에서 Blender 기준을 우선합니다.
- 프리뷰 평가는 런타임과 동일한 경로를 사용합니다.
- 어댑터 확장은 훅 기반으로 허용하되, 코어 편집 파이프라인은 유지합니다.

## 관련 문서

- [LevelSequence](../../runtime/animation-track/level-sequence.md)
- [AnimationTrack](../../runtime/animation-track/index.md)
- [Section Evaluator](../../runtime/animation-track/section-evaluator.md)
