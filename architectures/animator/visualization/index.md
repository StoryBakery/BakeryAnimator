---
title: Visualization
---

`Visualization`은 애니메이터의 뷰포트 표시 계층입니다.
애니메이션 데이터와 실행 로직과 분리해, 표시 방식만 독립적으로 확장합니다.
이 계층의 진실 원천은 `Instance` 트리가 아니라 플러그인 메모리의 가상 primitive 데이터입니다.

## 목표

- 기본 제공 비주얼(`Bone`, 선택 강조, 기준선)을 안정적으로 제공합니다.
- 어댑터가 프로젝트별 커스텀 비주얼을 추가할 수 있게 합니다.
- 표시 계층 변경이 `AnimationData`/`AnimationTrack` 계약을 깨지 않게 유지합니다.
- 표시 데이터와 저장 데이터를 분리해 인스턴스 수 증가를 억제합니다.

## 비영속 원칙

- 시각화 데이터는 플러그인 내부 가상 데이터(`AdapterVisualPrimitive`)로 관리합니다.
- 뷰포트 표시를 위해 백엔드가 임시 객체를 만들 수는 있지만, 저장/동기화 대상은 아닙니다.
- 시각화 상태를 수정해도 `AnimationFile` 계층은 변경하지 않습니다.
- 편집 저장이 필요한 값만 메타데이터 `Attribute`로 직렬화합니다.

## 렌더 파이프라인

1. 코어가 기본 비주얼 primitive를 생성합니다.
1. 어댑터가 `BuildVisualizationPrimitives`로 커스텀 primitive를 추가합니다.
1. 코어가 primitive 목록을 병합하고 diff를 계산합니다.
1. 렌더 백엔드가 변경분만 적용합니다.

primitive 계약은 [AdapterContract](../../adapter/adapter-contract.md)의
`AdapterVisualPrimitive`를 기준으로 합니다.

## 기본 제공 비주얼

- `Bone` 표시
- 선택/비선택 상태 강조
- 편집 상태(`normal`, `on-edit`, `selected`)별 표시 변형

상세는 [본](./bone.md)을 참조합니다.

## 어댑터 확장

어댑터는 `AdapterContract`의 시각화 훅으로 커스텀 비주얼 primitive를 반환할 수 있습니다.

- 기본 비주얼은 코어가 생성합니다.
- 어댑터 비주얼은 기본 비주얼 위에 오버레이로 합성합니다.
- 어댑터 비주얼 실패 시 진단만 누적하고 기본 비주얼은 유지합니다.

관련 계약은 [AdapterContract](../../adapter/adapter-contract.md)를 참조합니다.

## 성능 원칙

- primitive 생성/삭제를 매 프레임 반복하지 않고 diff 적용을 우선합니다.
- 화면 밖 요소는 저주기 업데이트 또는 컬링합니다.
- 시각화 메타데이터는 `Attribute` 기반 네임스페이스 키를 우선 사용합니다.
- 동기화가 필요 없는 표시 상태는 메모리 캐시로만 유지합니다.
