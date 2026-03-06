---
title: Blender Transform Sync
---

블렌더 애드온과 로블록스 플러그인 간의 동기화 및 데이터 전송 설계입니다.

## 범위

- Blender `Action`/`NLA` 기반 편집 결과를 애니메이터 런타임 데이터로 동기화합니다.
- 실시간 프리뷰(WebSocket)와 파일 내보내기(`.sbbtra`)를 같은 규약으로 다룹니다.
- 좌표계/시간축/데이터 매핑/패킷 계약을 단일 문서로 고정합니다.
- `SceneSampled`/`ActionDriven` 해석 선택은 [Sync Mode](./sync-mode.md)에서 다룹니다.

## 동기화 정책 문서

- 동기화 해석 모드는 [Sync Mode](./sync-mode.md)에서 관리합니다.
- 동기화 방향, 초기 동기화 소스, 충돌 규약은
  [Sync Policy](./sync-policy.md)에서 관리합니다.
- 항목별 권한 소스와 양방향 라운드트립 범위는
  [Bidirectional Sync](./bidirectional-sync.md)에서 관리합니다.

## 시간축 규약

### 기준

- 편집 UX 기준 시간축은 Blender 프레임을 우선합니다.
- 내부 동기화/평가 시간축은 고정밀 시간 단위를 사용합니다.

### 매핑

- `DisplayRate`는 Blender Scene FPS를 사용합니다.
- `TickResolution`은 `DisplayRate`보다 높은 정밀도로 설정합니다.
- 패킷은 다음 값을 함께 보냅니다.
  - `Frame`: 정수 프레임
  - `SubFrame`: 서브프레임(0 이상)
  - `TimeSeconds`: 초 단위 절대 시간

즉, 편집은 Blender처럼 프레임 중심으로 보이고,
평가는 Unreal 계열처럼 고정밀 시간축으로 처리합니다.

## 데이터 매핑

Blender 편집 단위를 런타임 데이터로 매핑합니다.

- `Action` -> `BindingGroupAnimationData` 내 트랙 묶음
- `NLA Strip` -> `SectionData` 또는 `LevelSequenceClip`
- `F-Curve` -> `ChannelData`
- `Keyframe Point` -> `KeyframeData`
- `Timeline Marker` -> `SequenceMarkers`
- 바인딩 로컬 마커 -> `TrackData.Markers`

모드별 매핑 차이(`SceneSampled`, `ActionDriven`)는
[Sync Mode](./sync-mode.md)를 기준으로 해석합니다.

매핑 원문은 동기화 문서에서 관리하며,
런타임 데이터 문서에는 DCC별 매핑 정책을 넣지 않습니다.

## 희소 키프레임 정책

- 전송 포맷은 키프레임 하나에 여러 채널 값을 담을 수 있습니다.
- 사용하지 않는 채널 값은 비워둘 수 있습니다.
- 권장 필드:
  - `ValuesByChannelKey: {[string]: any}?`
  - `InterpolationMode`, 핸들 정보는 채널별로 선택 포함

수신 측은 희소 키를 채널별 키로 확장하여 저장합니다.

## 패킷 계약

### Envelope

```lua
type SyncPacketEnvelope = {
    SessionId: string,
    Source: "Blender" | "Studio",
    PacketId: string,
    Revision: number,
    AckRevision: number?,
    SentAtUnixMs: number,
    PacketType: string,
    Payload: table,
}
```

- `PacketId`는 멱등 처리 키입니다.
- `Revision`은 세션 내 단조 증가 값입니다.
- `AckRevision`으로 수신 측 적용 완료 지점을 알려 재전송/재개에 사용합니다.

### 최소 PacketType

- `Snapshot`
  - 전체 스냅샷 전송(초기 동기화)
  - `SourceSceneName`을 포함해 추출 기준 Scene을 함께 전송합니다.
- `Delta`
  - 변경분 전송(실시간 동기화)
- `Ack`
  - 적용 확인 응답
- `RequestResync`
  - 세션 드리프트 또는 누락 패킷 복구 요청

## 어댑터 연계

- 속성/값 해석은 어댑터가 담당합니다.
  - [AdapterContract](../../adapter/adapter-contract.md)의 `DetectProperties`, `Write` 경로 사용
- 코어/애드온은 추출/전송/재생 파이프라인을 담당합니다.
- 어댑터가 해석하지 못한 속성은 무시하지 않고 진단으로 반환합니다.
- 전송 계층은 값 전달만 담당하며, 어느 쪽이 권한 소스인지 여부는 정책 문서에서 결정합니다.

## 좌표계

로블록스와 블렌더의 좌표계 차이를 자동 보정합니다.

- Roblox: +X(Right), +Y(Up), -Z(Forward)
- Blender: +X(Right), +Y(Forward), +Z(Up)
- Conversion: `Roblox(x, y, z) = (Blender.x, Blender.z, -Blender.y)`

## Motor6D 조작

### Motor6D 생성 규칙

Blender에서 `Armature` 모디파이어가 있는 메쉬 오브젝트의 이름과
`Armature`의 Bone 이름이 일치하면 `Motor6D`로 변환합니다.
일치하지 않으면 `Bone` 인스턴스로 변환합니다.

### 회전 보정

Roblox에서 `glTF`/`FBX`를 통해 연결된 `Motor6D`는
초기 회전(Rest Rotation) 기준이 달라질 수 있습니다.

블렌더에서 애니메이션을 가져올 때 다음 과정을 적용합니다.

1. Neutral Pose Correction
   - Blender Bone의 월드 회전 축을 Roblox `Motor6D C0/C1` 기준으로 보정합니다.
1. Transform Calculation
   - `m.Transform = (Part0.CFrame * C0):Inverse() * (Part1.CFrame * m.C1)` 공식을
     역이용해 정확한 포즈를 계산합니다.
