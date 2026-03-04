---
title: EventTrack
---

이벤트 키를 시간축에 배치하는 트랙입니다.
재생 시 현재 프레임의 이벤트를 엔드포인트로 디스패치합니다.
기본 모드에서는 선택 기능이며, `Marker` 기반 워크플로우로 대체할 수 있습니다.

## Attributes

### TrackName

`string`

이벤트 트랙 이름입니다.

### Scope

`"Binding" | "BindingGroup" | "Global"?`

이벤트의 적용 범위입니다.
