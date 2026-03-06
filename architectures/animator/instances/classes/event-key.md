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
기본 인코딩은 JSON 문자열을 사용하며, 런타임 로딩 시 `table?`로 역직렬화합니다.

### ExecutionMode

`"Trigger" | "Repeater"?`

이벤트 실행 모드입니다.
`Trigger`는 키 시점에 한 번 실행하고, `Repeater`는 섹션 활성 구간에서 프레임마다 실행합니다.

### FireWhenForwards

`boolean?`
`@default true`

정방향 재생에서 이벤트를 발화할지 여부입니다.

### FireWhenBackwards

`boolean?`
`@default false`

역방향 재생에서 이벤트를 발화할지 여부입니다.
