---
title: Adapter UI Extension
---

어댑터가 UI를 확장하는 경계와 원칙을 정의합니다.

## 확장 지점

- 어댑터는 [AdapterContract](../../adapter/adapter-contract.md)의 UI 훅으로 표시를 보정합니다.
  - `DescribeBindingUi`
  - `DescribePropertyUi`
- 어댑터는 시각화 훅으로 커스텀 오버레이를 추가할 수 있습니다.
  - `BuildVisualizationPrimitives`

## 경계 규칙

- 어댑터 확장은 UI 표시/상호작용을 확장할 수 있습니다.
- 어댑터는 코어 편집 파이프라인을 대체하지 않습니다.
- 코어는 어댑터 훅 미구현 상황에서도 기본 UX를 유지해야 합니다.

## 표시 정책

- 어댑터가 제공하는 표시 이름/그룹/숨김 정책은 Outliner, Inspector, Timeline에 공통 반영합니다.
- 어댑터 진단 정보는 디버그 패널에서만 노출하고, 기본 패널은 간결하게 유지합니다.
