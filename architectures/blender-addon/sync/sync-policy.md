---
title: Blender Sync Policy
---

블렌더 애드온과 로블록스 스튜디오 간 동기화의 권한/초기화/충돌 처리 정책입니다.

## 범위

- 동기화 방향과 권한 소스 정책을 정의합니다.
- 동기화 해석 모드의 선택 정책을 정의합니다.
- 초기 연결 시 기준 소스 선택 정책을 정의합니다.
- 소스 간 동시 수정 충돌 처리 정책을 정의합니다.
- `Studio -> Blender` 제한 범위를 정의합니다.

## 동기화 해석 모드

- 기본 모드는 `SceneSampled`입니다.
- 고급 워크플로우에서는 `ActionDriven`을 선택할 수 있습니다.
- 모드별 해석 기준은 [Sync Mode](./sync-mode.md)에서 관리합니다.

## Scene 기준 정책

- `SourceSceneName` 기본값은 `"Scene"`입니다.
- `scene_scope = "NamedScene"`이면 `SourceSceneName`을 기준 Scene으로 사용합니다.
- `scene_scope = "ActiveScene"`이면 활성 Scene 이름을 기준 Scene으로 사용합니다.
- 기준 Scene 해석에 실패하면 `"Scene"`으로 폴백하고 진단을 남깁니다.

## 동기화 방향

### 기본 방향

- 기본 동기화 방향은 `Blender -> Studio`입니다.
- `Studio -> Blender`는 제한 모드로만 지원합니다.
- 항목별 권한 소스와 라운드트립 등급은 [Bidirectional Sync](./bidirectional-sync.md)에서 고정합니다.

### 초기 동기화 정책

- 초기 연결 시 `InitialSyncSource`를 결정합니다.
  - `Auto`: 값 존재 여부를 기준으로 소스를 자동 선택합니다.
  - `Blender`: 블렌더 값을 기준으로 스튜디오 값을 덮어씁니다.
  - `Studio`: 스튜디오 값을 기준으로 블렌더 값을 덮어씁니다.
- 기본값은 `Auto`입니다.
- `Auto` 결정 규칙은 다음 순서를 따릅니다.
  1. Studio 쪽에 사전 정의된 값이 있으면 `Studio`를 선택합니다.
  1. Studio 사전 정의가 없고, 블렌더 쪽 값 개수가 더 많으면 `Blender`를 선택합니다.
  1. 양쪽 모두 값이 없으면 `Studio`를 선택합니다.
  1. 그 외(동률/판정 불가)는 `Studio`를 선택합니다.
- 빈 값(`nil`/누락 필드) 채움은 선택된 초기 소스를 따릅니다.
  - `FillEmptyFromBlender = true`는 `Blender`가 선택된 경우에만 적용합니다.

### Studio -> Blender 제한

- `Studio -> Blender`는 다음 항목만 안정적으로 반영합니다.
  - Transform 채널(위치/회전/스케일)
  - 마커
- 다음 항목은 기본적으로 반영하지 않거나 축소 반영합니다.
  - Blender 전용 `F-Curve Modifiers`
  - Blender 전용 제약/리그 보조 데이터
  - Studio 런타임 전용 동작 훅

### 항목별 기본 권한

- 커브/키프레임/정밀 보간 저작은 기본적으로 `Blender`를 우선합니다.
- 바인딩 해석, 어댑터 메타데이터, 함수 실행 엔드포인트 해석은 기본적으로 `Studio`를 우선합니다.
- 마커와 단순 오디오 배치는 공유 편집을 허용하되, 같은 항목 충돌은 활성 권한 소스를 따릅니다.

## 충돌 규약

- 자동 병합은 기본적으로 수행하지 않습니다.
- 동일 세션 내 충돌은 `Revision` 기준 마지막 쓰기를 우선 적용합니다.
- 서로 다른 소스(`Blender`, `Studio`)에서 같은 대상/같은 시간/같은 채널을 동시에 수정하면
  활성 권한 소스를 우선합니다.
  - 기본 권한 소스는 초기 동기화에서 선택된 소스입니다.
- 충돌 항목은 삭제하지 않고 진단으로 누적합니다.

## 관련 문서

- [Sync Mode](./sync-mode.md): Scene 평가값 샘플 기준/Action 기준 해석 모드
- [Bidirectional Sync](./bidirectional-sync.md): 항목별 권한 소스와 라운드트립 범위
- [Transform Sync](./transform-sync.md): 시간축, 패킷, 좌표계, Motor6D 변환
