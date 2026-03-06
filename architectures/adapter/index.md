---
title: Adapter System
---

사용자가 인스턴스 감지 방식, 속성 감지 방식, 런타임 동작 방식을 선택하거나 교체할 수 있도록 하는 시스템입니다.

## 목표

- 기본 Roblox 의존 어댑터를 제공합니다.
- 기본 어댑터와 제3자 어댑터를 같은 위치, 같은 계약으로 로드합니다.
- `Property`, `Attribute`, `ValueObject`, 스크립트 상태 기반 감지를 모두 지원합니다.
- 런타임 전용 커스텀 동작도 어댑터 훅으로 확장합니다.

공통 레이어 기준과 충돌 우선순위는 [Architecture Main](../main.md)을 따릅니다.

## 핵심 원칙

### Default Is Just Another Adapter

기본 어댑터는 엔진 내부 특례가 아니라 `AdapterDefinition`을 구현한 일반 `ModuleScript`입니다.
코어는 어댑터 ID를 선택하고 인터페이스만 호출합니다.

### Same Registry, Same Loading Path

기본 어댑터와 제3자 어댑터는 동일한 `AdapterRegistry`에 등록합니다.
별도 하드코드 분기 없이 동일한 우선순위 규칙으로 선택합니다.

### Engine Dependency Out Of Core

ReflectionService, Roblox 기본 `Property` 규칙은 `DefaultRobloxAdapter` 내부에 둡니다.
코어 런타임은 어댑터가 반환한 바인딩/채널 정보만 사용합니다.

### Optional BindingGroup

`BindingGroup`은 선택 계층입니다.
특수 케이스에서는 그룹 없이 `Binding`을 루트에 직접 둘 수 있어야 합니다.

## 폴더 구조

```text
AdapterCore/
|- AdapterContract.luau
|- AdapterFactory.luau
|- AdapterValidator.luau

Adapters/
|- DefaultRoblox.adapter.luau
|- ProjectGameplay.adapter.luau
|- ThirdPartyCombat.adapter.luau
```

기본 어댑터와 제3자 어댑터를 동일 폴더와 동일 로딩 파이프라인에서 처리합니다.

상세 설계:

- [AdapterContract](./adapter-contract.md)
- [AdapterFactory](./adapter-factory.md)
- [AdapterValidator](./adapter-validator.md)
- [DefaultRobloxAdapter](./default-roblox-adapter/index.md)

## 계약 단일 원본

- 어댑터 타입 계약 원문은 [AdapterContract](./adapter-contract.md)에서만 유지합니다.
- 어댑터 정규화 규칙 원문은 [AdapterFactory](./adapter-factory.md)에서만 유지합니다.
- 어댑터 검증 규칙 원문은 [AdapterValidator](./adapter-validator.md)에서만 유지합니다.

`AdapterContext.BindingIndexById`는 런타임에서 `BindingId -> Binding` 재조회에 사용합니다.

## 실행 파이프라인

1. `AdapterRegistry`가 `Adapters/` 아래 모듈을 모두 로드합니다.
1. 각 모듈을 `AdapterFactory`로 정규화합니다.
1. `AdapterValidator`로 검증 후 통과한 어댑터만 등록합니다.
1. 사용자가 명시한 어댑터 ID가 있으면 우선 사용합니다.
1. 명시가 없으면 `CanHandle` + `Manifest.Priority`로 어댑터를 선택합니다.
1. 선택된 어댑터의 `DetectBindings`, `DetectProperties`로 트랙 매핑을 구성합니다.
1. 에디터에서는 `DescribeBindingUi`, `DescribePropertyUi` 결과를 반영해 표시를 보정합니다.
1. 뷰포트가 있으면 `BuildVisualizationPrimitives`로 어댑터 비주얼을 합성합니다.
1. 태그 기반 바인딩이 있으면 `ResolveBindingTag`로 런타임 대상 재해결을 수행합니다.
1. 이벤트 트랙이 있으면 `ResolveEventEndpoint`로 엔드포인트를 연결하고 `AdapterEventCall`로 호출합니다.
1. 오디오 트랙이 있으면 `ResolveAudioClip`으로 재생 대상을 해석합니다.
1. `Observe`를 구독해 변경 발생 시 해당 바인딩만 부분 재평가합니다.
1. `BuildRuntimeBehaviors`가 있으면 런타임 훅을 연결합니다.

## Blender 동기화 연계

- Blender 애드온은 원시 채널 데이터와 변경분만 전송합니다.
- 스튜디오 측 속성/값 해석은 어댑터가 담당합니다.
  - `DetectBindings`
  - `DetectProperties`
  - `AdapterPropertyChannel.Read` / `AdapterPropertyChannel.Write`
- 코어는 전송/저장/평가 파이프라인을 담당하고,
  최종 속성 적용 의미는 어댑터 해석 결과를 따릅니다.

## Visualization 확장 정책

- 코어는 기본 제공 비주얼(`Bone`, 선택 상태 표시)을 항상 렌더링합니다.
- 어댑터는 `BuildVisualizationPrimitives`로 커스텀 비주얼을 추가할 수 있습니다.
- 어댑터 비주얼은 기본 비주얼 위에 오버레이로 합성합니다.
- 시각화 primitive는 가상 데이터로 취급하며, 저장형 `Instance`를 직접 계약에 넣지 않습니다.
- 어댑터 비주얼 생성 실패 시 `AdapterDiagnostic`만 누적하고 코어 비주얼은 유지합니다.

## UI 표시 확장 정책

- 어댑터는 `DescribeBindingUi`, `DescribePropertyUi`로 표시 텍스트/정렬/숨김을 보정할 수 있습니다.
- UI 훅은 편집기 표시만 바꾸며, 감지/평가/적용 로직은 바꾸지 않습니다.
- UI 훅이 없으면 코어 기본 표시 규칙을 사용합니다.

## 인스턴스 계층 연계

저장형 인스턴스 계층 원문은 [Animator Instances](../animator/instances/instances.md)에서만 유지합니다.
어댑터 시스템은 그 계층을 다시 정의하지 않고, 다음 규약만 기대합니다.

- `BindingGroup`은 선택 계층입니다.
- `Binding`은 그룹 하위 또는 루트에 직접 올 수 있습니다.
- 전역 마커는 루트 파일에, 로컬 마커는 `Binding`에 둡니다.
- `PropertyTrack`, `EventTrack`, `AudioTrack`, `ConstraintTrack`은 모두 `Section` 단위 평가를 따릅니다.
- 어댑터 메타데이터는 별도 숨김 트리를 만들지 않고 기존 `Binding`/`BindingGroup`에 저장합니다.

### Section 계층 규약

- `Section`은 `PropertyTrack` 내부의 시간 구간 단위입니다.
- 각 `Section`은 시작/종료 시간과 블렌드 규칙을 가지며, 그 아래에 `Channel`과 `Keyframe`을 포함합니다.
- 같은 `PropertyTrack` 안에서 `Section`이 겹치면 섹션 블렌드 정책으로 합성합니다.
- `EventTrack`, `AudioTrack`, `ConstraintTrack`도 동일하게 `Section` 단위로 평가합니다.
- `Marker`는 `Section` 바깥의 동기화 기준점이며, 전역(`LevelSequence`)과 로컬(`Binding`) 두 계층을 가집니다.

## 어댑터 선택 규칙

1. 사용자 명시 선택값 (`AdapterId`)이 있으면 해당 어댑터를 강제 사용합니다.
1. 없으면 엔트리 포인트의 `AnimatorAdapterId` `Attribute`를 확인합니다.
1. 그래도 없으면 `CanHandle=true` 어댑터 중 최고 우선순위를 선택합니다.
1. 후보가 없으면 `DefaultRoblox`로 폴백합니다.

## 스크립트 상태 기반 감지

`ScriptState`는 스튜디오 인스펙터에서 직접 보이지 않을 수 있으므로,
어댑터가 다음 훅을 제공하도록 권장합니다.

- `Observe`: 런타임 상태 변경 감지
- `SimulateEditorState`: 에디터에서 가능한 범위의 미리보기 상태 계산

이 계약을 사용하면 `Attribute`/`ValueObject`에 의존하지 않는 규칙도
동일한 파이프라인에서 다룰 수 있습니다.

## AdapterMetadata 저장 정책

- 어댑터 메타데이터는 기본적으로 소유 인스턴스(`Binding`, `BindingGroup`)의 `Attribute`에 저장합니다.
- 키 형식은 `METADATA_<ADAPTER>_<PROP>`를 사용합니다.
  - 예: `METADATA_DEFAULT_ROBLOX_TARGET_PATH`
- 어댑터 ID는 대문자 `SNAKE_CASE`로 정규화해 키에 사용합니다.
  - 예: `DefaultRoblox` -> `DEFAULT_ROBLOX`
- `Instance` 참조가 필요한 경우에만 예외적으로 `ObjectValue`를 사용합니다.
  - 예: `METADATA_DEFAULT_ROBLOX_TARGET_REF` (`ObjectValue`)
- 여러 속성을 한 JSON `Attribute` 하나에 몰아넣는 방식은 권장하지 않습니다.
  키 단위 변경 감지/부분 업데이트/진단 가독성이 떨어지기 때문입니다.

## 기본 어댑터 상세

기본 어댑터 구현 예시와 `AdapterMetadata` 저장 규칙은
[DefaultRobloxAdapter](./default-roblox-adapter/index.md) 문서에서 관리합니다.

## 권장 정책

- `BindingId`를 1순위 식별자로 사용하고 `TrackName`은 표시용 보조 키로 유지합니다.
- `BindingGroupId`는 선택값으로 취급하고, 없는 경우 루트 바인딩으로 처리합니다.
- 어댑터 구현은 `throw` 대신 `AdapterDiagnostic`을 반환합니다.
- 어댑터 모듈은 테이블 직접 반환 또는 `new()` 생성자 반환 중 하나로 통일합니다.
- 기본 어댑터는 이벤트 엔드포인트 allowlist만 실행하고 임의 코드 문자열 실행은 금지합니다.
- 오디오를 지원하는 어댑터는 `ResolveAudioClip`을 구현하고, 미지원이면 진단과 함께 스킵합니다.
- 느린 어댑터는 `DetectProperties` 결과를 캐시하고 부분 무효화 전략을 사용합니다.
