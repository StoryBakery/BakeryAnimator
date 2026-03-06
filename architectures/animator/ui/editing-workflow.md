---
title: Editing Workflow
---

편집 단위 우선순위와 각 편집기 역할을 정의합니다.

## 편집 우선순위

- 1순위 편집 단위는 `Section`입니다.
  - 트림, 분할, 이동, 겹침 우선순위 조정은 `Timeline`에서 처리합니다.
- 2순위 편집 단위는 `Channel`/`Keyframe`입니다.
  - 보간/핸들/곡선 형태 조정은 `GraphEditor`에서 처리합니다.
- `Track`은 섹션 집합을 관리하는 컨테이너로 취급합니다.
  - 빈번한 값 편집은 트랙 레벨이 아니라 섹션/채널 레벨에서 수행합니다.

## 기본 편집 트리

`BindingGroup > Binding > Track > Section > Channel > Keyframe`

## 편집기별 역할

- `Timeline`
  - 섹션 배치/오버랩/구간 정리
  - 구간 레벨 스냅과 시간 정렬
- `GraphEditor`
  - 채널 보간 모드 조정
  - 키프레임/핸들 정밀 편집
  - Blender `Graph Editor`와 같은 역할을 담당합니다.
- `Inspector`
  - 실행 규칙 편집
  - `BlendType`, `RowIndex`, `OverlapPriority`, `EaseIn`, `EaseOut`, `CompletionMode`

## 프리뷰 원칙

- 프리뷰 평가는 런타임과 동일한 평가 경로를 사용합니다.
- 편집 미리보기와 실제 재생 결과의 불일치를 최소화합니다.

`GraphEditor` 세부 동작은 [Graph Editor](./graph-editor.md)를 따릅니다.
