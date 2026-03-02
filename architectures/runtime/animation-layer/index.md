---
title: AnimationLayer
---

## Properties

### Priority

`number`

VBP 노드의 우선순위입니다.

### BlendMode

`AnimationBlendMode`

이 레이어가 그래프의 다른 노드들과 합성되는 방식입니다. `ValueByPriority`의 **NodeFunction**에 매핑됩니다.

- **`WeightedSet`** (Default): Priority 그룹 내의 값들을 가중치 비율로 혼합합니다.
  - **합(Sum) < 1.0**: 남은 비율만큼 하위 Priority의 값을 유지합니다. (투명도 적용)
  - **합(Sum) >= 1.0**: 하위 Priority의 값을 완전히 덮어쓰고, 그룹 내에서 정규화(Normalize)하여 섞습니다.
- **`Add`**: 하위 Priority의 결과값 위에 현재 값을 더합니다. (Additive Animation)
- **`Multiply`**: 하위 Priority의 결과값에 현재 값을 곱합니다.

`Relative`는 레이어 블렌드 모드가 아닙니다.
트랙의 `PoseMode = Relative`에서 평가 단계에서 Delta로 변환된 뒤
레이어에 입력됩니다.

### Weight

`number`

$[0.0, 1.0]$ 이 레이어의 영향력입니다.

- `WeightedSet` 모드에서는 **혼합 비율**로 사용됩니다.
- `Add`/`Multiply` 모드에서는 **강도**로 사용됩니다.

## Methods

### Write

`(value: any) -> ()`

현재 프레임의 속성 값을 VBP 노드에 기록합니다.

### SetWeight

`(weight: number) -> ()`

레이어의 가중치를 실시간으로 변경합니다. (Fade In/Out 구현 시 사용)

### SetBlendMode

`(mode: AnimationBlendMode) -> ()`

합성 방식을 변경합니다.

### Destroy

`() -> ()`

VBP 그래프에서 노드를 제거하고 연결을 끊습니다.
