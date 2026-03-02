---
title: Animator Instances
---

애니메이터는 애니메이션 파일의 데이터를 `Instance`를 진실의 원천으로 둡니다.
모든 데이터와 UI 상태는 `:GetAttributeChangedSignal`과 `:GetAttribute`로 감지하며,
`Observe` 패턴을 사용합니다.

## 이유

- 인스턴스를 기준으로 하면 `ChangeHistoryService`를 통해 로블록스 스튜디오 수정
  모드에서 `Ctrl + Z`와 `Ctrl + Shift + Z`를 쉽게 구현할 수 있습니다.
- 제3자 에디터를 만들거나 다른 플러그인에서 값을 수정해도, 인스턴스가 진실의 원천이므로 데이터 일관성을 유지할 수 있습니다.
