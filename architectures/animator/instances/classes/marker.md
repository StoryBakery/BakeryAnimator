---
title: Marker
---

애니메이션 파일 전체 타임라인의 기준점입니다.
전역(`AnimationFile`)과 로컬(`Binding`) 두 범위에서 사용합니다.

## Attributes

### Name

`string`

마커 이름입니다.

### Time

`number`

마커 시각(초)입니다.

### Value

`string?`

Roblox `KeyframeMarker.Value`와 매핑되는 추가 문자열입니다.

## Parent Rules

- `AnimationFile` 하위에 두면 전역 마커로 동작합니다.
- `Binding` 하위에 두면 해당 바인딩 트랙 전용 로컬 마커로 동작합니다.
- 같은 이름이 충돌하면 로컬 마커가 전역 마커보다 우선합니다.
