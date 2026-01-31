# AnimationTrack

`AnimationData`를 실제 로블록스 인스턴스(`Rig`)에 바인딩하여 재생하는 **런타임 실행기(Runtime Player)**입니다.

## Properties

#### IsDestroyed
`boolean`

객체의 파괴 여부입니다.

#### Maid
`Maid`

객체의 수명 관리를 담당하는 Maid 인스턴스입니다.

#### Data
[AnimationData](./animation-data.md)

재생 중인 원본 애니메이션 데이터입니다.

#### TimePosition
`number`

현재 재생 시간 (초) 입니다.

#### Speed
`number`

재생 속도 배율입니다.

#### IsPlaying
`boolean`

현재 재생 중인지 여부입니다.


## Constructors

#### fromData
(data: AnimationData, targets: {[string]: Instance}, params: AnimationTrackParams?) -> (AnimationTrack)

데이터와 대상 리그 매핑을 받아 애니메이션 트랙을 생성합니다.

```lua
AnimationTrackParams = {
    Speed: number?
}
```


## Methods

#### Destroy
() -> ()

트랙을 파괴하고 관련 리소스를 정리합니다.

#### Play
() -> ()

애니메이션 재생을 시작하거나 재개합니다.

#### Stop
() -> ()

애니메이션을 일시 정지합니다.

#### SetTimePosition
(time: number) -> ()

현재 진행 시간을 설정하고 즉시 포즈를 업데이트합니다.
