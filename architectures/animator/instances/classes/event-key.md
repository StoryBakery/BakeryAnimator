---
title: EventKey
---

특정 시점에 실행될 이벤트를 표현합니다.
`EventTrack`의 `Section` 하위에서 사용됩니다.

## Attributes

### Time

`number`

이벤트 발생 시각(초)입니다.

### EndpointId

`string`

런타임에서 호출할 이벤트 엔드포인트 식별자입니다.

### Payload

`string?`

이벤트에 전달할 직렬화된 추가 데이터입니다.
