---
title: ConstraintTrack
---

바인딩 간 제약을 시간 구간 단위로 적용하는 트랙입니다.
`PropertyTrack` 평가 이후에 합성됩니다.

## Attributes

### TrackName

`string`

제약 트랙 이름입니다.

### ConstraintType

`"Parent" | "Position" | "Rotation" | "Scale" | "LookAt"`

제약 종류입니다.

### SourceBindingId

`string`

제약을 거는 기준 바인딩 ID입니다.

### TargetBindingId

`string`

제약을 받는 대상 바인딩 ID입니다.
