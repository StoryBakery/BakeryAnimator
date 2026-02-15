# 개요

애니메이터는 애니메이션 파일의 데이터를 `Instance` 가 진실의 원천이도록 잡습니다.
그리고 모든 데이터와 ui 의 데이터 또한, `:GetAttributeChangedSignal` 이나 `:GetAttribute` 를 통해 잡아냅니다.
이 때 `Observe` 패턴을 사용합니다


이유는
- 인스턴스 로 하면 `ChangeHistoryService` 를 통해 로블록스 스튜디오 수정 모드에서, `Ctrl + Z` 나 `Ctrl + Shift + Z` 를 쉽게 구현할 수 있음
- 제 3자 에디터를 만들어서, 다른 플러그인에서 수정해도, 인스턴스 가 진실의 원천이기에 데이터 