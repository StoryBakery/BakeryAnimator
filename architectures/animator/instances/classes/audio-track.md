---
title: AudioTrack
---

오디오 클립을 시간축에 배치하는 트랙입니다.
재생 시 활성 `Section`을 기준으로 사운드를 재생/정지하고, 볼륨/피치 자동화를 적용합니다.

## Attributes

### TrackName

`string`

오디오 트랙 이름입니다.

### Scope

`"Binding" | "BindingGroup" | "Global"?`

오디오의 적용 범위입니다.

### DefaultAudioAssetId

`string?`

섹션에 개별 오디오 에셋이 없는 경우 사용할 기본 에셋 ID입니다.

### DefaultAttachBindingId

`string?`

공간 오디오를 특정 바인딩에 붙여 재생할 때 사용하는 기본 바인딩 ID입니다.
