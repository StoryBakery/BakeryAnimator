# AnimationLayer

`AnimationTrack` (또는 내부의 개별 속성 커브)가 `Animator`의 **ValueByPriority** 시스템에 값을 주입하기 위해 사용하는 **합성 인터페이스(Composition Interface)**입니다.
물리적인 레이어라기보다, VBP 그래프 상의 **Node** 하나를 제어하는 핸들(Handle/Proxy)에 가깝습니다.

## Core Concept

#### VBP Integration
각 `AnimationLayer`는 `Animator`가 관리하는 특정 속성(예: `"Head.CFrame"`)의 VBP 그래프에 하나의 **Node**로 등록됩니다.
트랙이 재생되는 동안 이 노드는 활성화되어 값을 공급하며, 정지되면 그래프에서 비활성화됩니다.

```
[Track (Provider)] -> [Layer (Node Handle)] -> [Animator (VBP Graph)]
```

## Properties

#### Priority
`number`

VBP 노드의 우선순위입니다.

#### BlendMode
`AnimationBlendMode`

이 레이어가 그래프의 다른 노드들과 합성되는 방식입니다. `ValueByPriority`의 **NodeFunction**에 매핑됩니다.

*   **`WeightedSet`** (Default): Priority 그룹 내의 값들을 가중치 비율로 혼합합니다.
    *   **합(Sum) < 1.0**: 남은 비율만큼 하위 Priority의 값을 유지합니다. (투명도 적용)
    *   **합(Sum) >= 1.0**: 하위 Priority의 값을 완전히 덮어쓰고, 그룹 내에서 정규화(Normalize)하여 섞습니다.
*   **`Add`**: 하위 Priority의 결과값 위에 현재 값을 더합니다. (Additive Animation)
*   **`Multiply`**: 하위 Priority의 결과값에 현재 값을 곱합니다.

#### Weight
`number`

(0.0 ~ 1.0) 이 레이어의 영향력입니다.
*   `WeightedSet` 모드에서는 **혼합 비율**로 사용됩니다.
*   `Add`/`Multiply` 모드에서는 **강도(Intensity)**로 사용됩니다.

## Methods

#### Write
(value: any) -> ()

현재 프레임의 속성 값을 VBP 노드에 기록합니다.

#### SetWeight
(weight: number) -> ()

레이어의 가중치를 실시간으로 변경합니다. (Fade In/Out 구현 시 사용)

#### SetBlendMode
(mode: AnimationBlendMode) -> ()

합성 방식을 변경합니다.

#### Destroy
() -> ()

VBP 그래프에서 노드를 제거하고 연결을 끊습니다.
