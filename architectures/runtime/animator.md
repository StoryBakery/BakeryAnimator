# Animator

개별 모델(`Model`)에 부착되어, 해당 모델의 **모든 속성 값(Properties)**을 계산하고 적용하는 **로컬 평가 엔진(Local Evaluation Engine)**입니다.
`AnimationTrack` (Director)에 의해 제어되며, 독립적으로 재생을 주관하지 않습니다.

## Responsibilities

1.  **Graph Management**: 각 속성(`Property`)마다 `ValueByPriority` 그래프를 생성하고 관리합니다.
2.  **Signal Composition**: 여러 트랙에서 들어오는 신호(Layer)를 VBP 로직으로 합성합니다.
3.  **Local Evaluation**: 매 프레임 모델의 최종 포즈를 계산(`Compute`)하고 적용(`Apply`)합니다.

## Properties

#### Root
`Instance` (Model)
이 애니메이터가 담당하는 모델입니다.

#### Evaluators
`{ [string]: VBP.Evaluator }`
속성 이름(예: `"Head.CFrame"`)을 키로 하는 VBP 평가기들의 집합입니다.

## Methods

#### LinkTrack
(track: AnimationTrack, priority: number) -> (AnimationLayer)
`AnimationTrack`으로부터 입력을 받을 수 있는 **전용 레이어(Interface Layer)**를 생성하여 반환합니다.
트랙은 이 레이어를 통해 자신의 값을 `Animator`에 주입합니다.

#### Compute
(deltaTime: number) -> ()
모든 VBP 그래프를 업데이트합니다.

#### Apply
() -> ()
계산된 최종 값을 실제 인스턴스에 반영합니다.
