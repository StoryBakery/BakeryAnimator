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

#### Models
`{ [string]: ModelSkeletonData }`
이 애니메이션에 등장하는 모델(Model)들의 데이터 맵입니다.
*   **Single Model**: `Models["Default"]` (또는 `Root`) 하나만 존재.
*   **Multi Model**: `Models["Hero"]`, `Models["Enemy"]` 등 다수 존재.

```lua
type ModelSkeletonData = {
    Tracks: { [string]: TrackData } -- Property Tracks (e.g. "Head.CFrame")
}
```

#### Clips
`{ AudioClip | AnimationClip }`
이 데이터 내부에 포함된 하위 클립들입니다.
*   **Global Level**: 전체 시퀀스 레벨에서 실행되는 클립 (예: 카메라, 환경 효과)
*   **Model Level**: 특정 모델 전체에 적용되는 클립 (예: 걷기 사이클 파일 통째로 삽입)
*   **Property Level**: 특정 속성 변화를 위해 삽입된 클립

```lua
type AnimationClip = {
    Type: "Animation",
    Data: AnimationData | ObjectValue<AnimationData>, -- Direct or Reference
    Scope: "Global" | "Model" | "Object" | "Property", -- Where it applies
    StartTime: number,
    Duration: number,
    Speed: number,
    Weight: number,
    Offset: number -- Start offset within the source clip
}
```

## Constructors

#### fromTable
(data: table, params: AnimationDataParams?) -> (AnimationData)
Raw 테이블 데이터로부터 AnimationData 인스턴스를 생성합니다.

## Methods

#### GetModelData
(modelName: string?) -> (ModelSkeletonData?)
특정 모델의 데이터를 가져옵니다. `modelName`이 없으면 기본 모델 데이터를 반환 시도합니다.

#### FindTrackData
(modelName: string, trackName: string) -> (table?)
특정 모델의 특정 트랙 데이터를 반환합니다.
