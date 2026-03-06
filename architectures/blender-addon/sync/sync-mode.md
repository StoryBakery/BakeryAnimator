---
title: Blender Sync Mode
---

블렌더 동기화에서 어떤 기준으로 애니메이션 데이터를 해석할지 정의하는 문서입니다.

이 문서의 `Sync Mode`는 `adapter-contract`에서 정의한 어댑터와 다른 개념입니다.

- `adapter-contract` 어댑터: 런타임 속성 해석/적용 책임
- `Blender Sync Mode`: 블렌더 데이터 추출/구조화 기준

양방향 편집 범위와 항목별 권한 소스는 [Bidirectional Sync](./bidirectional-sync.md)에서 다룹니다.

## 목표

- 초보 사용자에게는 Scene 평가값 샘플 중심의 단순 워크플로우를 제공합니다.
- 고급 사용자에게는 Action 단위 자산 중심 워크플로우를 제공합니다.
- 같은 `.blend`에서도 동기화 의도에 맞는 데이터 구조를 선택할 수 있게 합니다.

## Sync Mode

### SceneSampled

- 권장 대상: NLA 내부 구조를 깊게 다루지 않는 사용자
- 기본값: `SceneSampled`
- 해석 기준:
  - 현재 Scene의 평가 결과(워크스페이스에서 보이는 최종 값)를 프레임 단위로 샘플링합니다.
  - Armature/Object별 바인딩 경로에 연속된 섹션을 생성합니다.
  - 기본 추출 대상은 Transform이며, 그 외 속성 해석은 런타임 어댑터에 위임합니다.
  - 결과는 "베이크된 최종 포즈/값" 중심이며 Action/Strip 구조는 보존하지 않습니다.
- 장점:
  - 직관적이고 실패 확률이 낮습니다.
  - 미리보기와 결과 일치성이 높습니다.
- 제약:
  - Action 재사용 구조를 그대로 보존하지 못합니다.
  - NLA 편집 의도(스트립 단위 조합 정보)는 축약됩니다.

### ActionDriven

- 권장 대상: Action/NLA를 명시적으로 구성하는 고급 사용자
- 해석 기준:
  - Action 데이터를 우선 추출해 재사용 가능한 클립 단위로 동기화합니다.
  - NLA Strip은 Action 클립의 배치 정보(시작/길이/속도/오프셋/블렌딩)로 해석합니다.
  - 사용자가 연결한 트랙 배치를 우선 반영합니다.
- 장점:
  - 같은 `.blend`에서 Action 자산을 블록처럼 조합하는 워크플로우를 유지할 수 있습니다.
  - NLA 구성 변경을 클립 재배치 수준으로 관리하기 쉽습니다.
- 제약:
  - 추출/검증 규칙이 많아 초기 설정이 더 복잡합니다.
  - Strip 타입별(Transition/Meta/Sound) 매핑 정책이 필요합니다.

## Scene Scope

Scene 기준을 애니메이션 파일마다 지정할 수 있습니다.

- `ActiveScene`
  - 동기화 시점의 활성 Scene을 사용합니다.
- `NamedScene`
  - `SourceSceneName`으로 지정된 Scene을 사용합니다.
- `SourceSceneName` 기본값은 `"Scene"`입니다.

Scene 선택 우선순위는 다음과 같습니다.

1. `scene_scope = "NamedScene"`이면 `SourceSceneName`을 사용합니다.
1. `scene_scope = "ActiveScene"`이면 활성 Scene 이름을 사용합니다.
1. 위 해석이 불가능하면 `"Scene"`으로 폴백합니다.

Scene 기준은 `RecordingMetadata.SourceSceneName`에 저장합니다.

## 트랙/클립 해석 규칙

### SceneSampled

- 레벨 단위:
  - 평가 결과를 `LevelSequenceData` 기준 시간축으로 정규화합니다.
- 트랙 단위:
  - 바인딩별 `PropertyTrackData`를 생성하고, 기본적으로 연속 섹션으로 저장합니다.
- 클립 단위:
  - 필요 시 바인딩 단위 `Animation` 클립으로 구성하되, 원본 NLA Strip 구조를 보존하지 않습니다.

### ActionDriven

- 레벨 단위:
  - Action 클립 라이브러리와 배치(Strip) 정보를 분리 관리합니다.
- 트랙 단위:
  - Action 소스 채널을 보존하고, Strip 배치 정보는 섹션/클립 메타로 저장합니다.
- 클립 단위:
  - NLA `Action Strip`은 `Animation` 클립 배치로 매핑합니다.
  - `Transition`은 구간 블렌딩(`EaseIn`/`EaseOut`)으로 매핑합니다.
  - `Meta`는 내부 스트립을 유지한 채 그룹 메타로 저장하거나 평탄화합니다.
  - `Sound`는 `Audio` 클립으로 매핑합니다.

## 운영 주의점

- `SceneSampled`는 프레임 샘플링이 많아질수록 데이터가 빠르게 커집니다.
  - 채널 최적화(Constant/Epsilon 제거)를 기본 적용해야 합니다.
- `ActionDriven`는 Action 이름 충돌과 참조 누락 검증이 필요합니다.
  - 키는 이름이 아니라 안정적인 식별자(해시/UUID)를 권장합니다.
- Scene 이름 기반 동기화(`NamedScene`)는 Scene rename 시 재연결 정책이 필요합니다.
- 두 모드 모두 "최종 적용 대상 속성"은 런타임 어댑터와 계약이 맞아야 합니다.

## 속성 해석 경계

- Blender Sync Mode는 "추출/구조화"까지 담당합니다.
- 실제 속성 키 해석은 런타임 어댑터가 담당합니다.
  - [AdapterContract](../../adapter/adapter-contract.md)의 `DetectProperties`, `Write` 경로 사용

즉, 동기화 모드는 "어떤 데이터를 가져올지"를 결정하고,
런타임 어댑터는 "그 데이터를 어디에 적용할지"를 결정합니다.
