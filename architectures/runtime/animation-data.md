# AnimationData

애니메이션의 **정적 데이터(Static Data)**를 관리하는 불변(Immutable) 객체입니다.
"어떻게 움직여야 하는가"에 대한 정보만을 담으며, 런타임 상태를 가지지 않습니다.

## Properties

#### FrameRate
`number`

초당 프레임 수 (FPS) 입니다.

#### Looped
`boolean`

애니메이션 반복 재생 여부입니다.

#### Priority
`number`

애니메이션의 우선순위입니다.

#### Tags
`{string}`

애니메이션에 할당된 식별 태그 목록입니다.

#### Duration
`number`

애니메이션의 총 길이 (초 단위) 입니다.


## Constructors

#### fromTable
(data: table, params: AnimationDataParams?) -> (AnimationData)

Raw 테이블 데이터로부터 AnimationData 인스턴스를 생성합니다.

#### fromInstance
(root: Folder, params: AnimationDataParams?) -> (AnimationData)

로블록스 인스턴스(Folder/ObjectValue 구조)로부터 데이터를 파싱하여 생성합니다.


## Methods

#### FindTrackData
(trackName: string) -> (table?)

특정 트랙 이름에 해당하는 채널 데이터를 반환합니다.

#### GetTrackData
(trackName: string) -> (table)

특정 트랙 이름에 해당하는 채널 데이터를 반환하거나, 없으면 에러를 발생시킵니다.

#### Destroy
() -> ()

데이터 객체의 메모리를 명시적으로 해제해야 할 경우 사용합니다.
