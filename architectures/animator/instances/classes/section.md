---
title: Section
---

트랙 내부의 시간 구간 단위입니다.
겹치는 구간은 섹션 블렌드 규칙으로 합성됩니다.

## Attributes

### SectionId

`string`

섹션 고유 식별자입니다.

### StartTime

`number`

섹션 시작 시각(초)입니다.

### EndTime

`number?`

섹션 종료 시각(초)입니다.
없으면 열린 구간으로 취급합니다.

### BlendType

`"Absolute" | "Additive" | "Relative"`

섹션 합성 방식입니다.

### CompletionMode

`"KeepState" | "RestoreState" | "ProjectDefault"`

섹션 종료 후 상태 처리 방식입니다.
