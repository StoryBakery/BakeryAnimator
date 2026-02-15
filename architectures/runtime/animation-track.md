# AnimationTrack

## 개요

`AnimationData`를 실제 로블록스 인스턴스(`Rig`)에 바인딩하여 재생하는 **런타임 실행기(Runtime Player)**입니다.
단순한 재생을 넘어, 트랙 간의 **블렌딩(Blending)**, **합성(Composition)**, **가중치(Weight)** 제어를 수행하는 핵심 단위입니다.

## Constructors

#### fromData
`(data: AnimationData, target: Model, params: AnimationTrackParams?) -> (AnimationTrack)`

단일 모델(`target`)을 대상으로 하는 애니메이션 트랙을 생성합니다.
통합 데이터(`AnimationData`) 내에서 특정 모델(`modelName`)의 데이터를 추출하여 사용합니다.

*   `data`: 통합 애니메이션 데이터.
*   `target`: 애니메이션을 적용할 실제 모델.

```lua
AnimationTrackParams = {
    Speed: number?,
    Weight: number?,
    Priority: number?
}
```

#### fromFile
`(file: AnimationFile, target: Model, params: AnimationTrackParams?) -> (AnimationTrack)`


## Properties

#### IsDestroyed
`boolean`

객체의 파괴 여부입니다.

#### Maid
`Maid`

객체의 수명 관리를 담당하는 Maid 인스턴스입니다.

#### Data
`AnimationData`

재생 중인 원본 애니메이션 데이터입니다.

#### TimePosition
`number`

현재 재생 시간 (초) 입니다.

#### Speed
`number`

재생 속도 배율입니다. 기본값은 `1.0`입니다.

#### Weight
`number`

애니메이션의 영향력(가중치)입니다. `0.0` ~ `1.0` 사이의 값을 가집니다.
여러 트랙이 섞일 때 이 값이 클수록 더 강하게 표현됩니다.

#### BlendMode
`AnimationBlendMode`
`@default WeightedSet`

이 트랙이 **레이어 내부에서** 다른 트랙들과 합성되는 방식입니다.

*   `WeightedSet`: 가중치(Weight) 비율에 따라 값을 적용합니다.
    *   단일 트랙일 경우 Weight만큼 하위 레이어를 덮어씁니다. (Alpha Blend)
    *   같은 우선순위에 여러 트랙이 있을 경우, 가중치 비율대로 섞입니다. (BlendSpace)
*   `Additive`: (`Add`) 현재 값 위에 더합니다. (반동, 흔들림 등)
*   `Multiply`: (`Mul`) 현재 값에 곱합니다.
*   `Relative`: 초기 포즈 대비 변화량만을 적용합니다.

#### IsPlaying
`boolean`

현재 재생 중인지 여부입니다.

#### Priority
`number`
`@default 0`

**레이어 내부에서의** 연산 우선순위입니다. 
`ValueByPriority` 노드의 우선순위로 매핑됩니다. 높은 우선순위의 트랙이 나중에 연산되어 결과를 덮어쓰거나 합성합니다.

#### Looped
`boolean`
`@default false`

`Data`의 설정을 덮어쓰고 반복 여부를 실시간으로 제어합니다.

## Methods

#### Destroy
() -> ()

트랙을 파괴하고 관련 리소스를 정리합니다.

#### Play
(fadeTime: number?, weight: number?, speed: number?) -> ()

애니메이션 재생을 시작하거나 재개합니다.
*   `fadeTime`: 지정한 시간(초) 동안 Weight를 0에서 목표값까지 부드럽게 올립니다. (Fade In)
*   `weight`: 재생 시 목표 Weight입니다.
*   `speed`: 재생 속도입니다.

#### Stop
(fadeTime: number?) -> ()

애니메이션을 정지합니다.
*   `fadeTime`: 지정한 시간(초) 동안 Weight를 현재 값에서 0으로 부드럽게 내린 후 정지합니다. (Fade Out)

#### AdjustWeight
(targetWeight: number, fadeTime: number?) -> ()

실행 중에 트랙의 가중치를 부드럽게 변경합니다. 다른 동작과 섞이거나 서서히 사라지게 할 때 사용합니다.

#### SetTimePosition
(time: number) -> ()

현재 진행 시간을 설정하고 즉시 포즈를 업데이트합니다.
