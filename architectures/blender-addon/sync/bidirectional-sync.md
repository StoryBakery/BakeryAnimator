---
title: Blender Bidirectional Sync
---

Blender와 Roblox Studio 플러그인 사이의 양방향 편집 범위, 권한 소스, 손실 허용 범위를 정의합니다.

## 목표

- Blender가 강한 저작 영역과 Studio가 강한 저작 영역을 분리합니다.
- 실시간 프리뷰와 영속 라운드트립을 같은 개념으로 취급하지 않습니다.
- 어떤 항목이 안정적으로 왕복 가능한지 문서로 고정합니다.
- 손실 변환이 발생하는 경우 진단을 남기고 조용히 삼키지 않습니다.

## 기본 입장

- 이 시스템은 완전 대칭 편집기를 목표로 하지 않습니다.
- 같은 항목을 양쪽에서 무제한으로 동시에 편집하는 구조도 목표로 하지 않습니다.
- 항목별로 "누가 원본에 더 가깝게 저작할 수 있는가"를 기준으로 권한 소스를 정합니다.

## 기술 검증 요약

- Blender `SceneSampled`는 단순 원본 데이터 접근이 아니라, Scene 평가 결과를 읽는 방식으로 구현해야 합니다.
- Blender `ActionDriven`는 `Action.fcurves`만 보면 안 됩니다.
  최신 Blender API에서 `Action.fcurves`는 첫 번째 슬롯만 보는 legacy 경로이기 때문입니다.
- Blender NLA 재구성은 전부 컬렉션 API만으로 끝나지 않습니다.
  `Transition`, `Meta`, `Sound`는 operator 기반 재구성 경로까지 고려해야 합니다.
- Roblox Studio 실시간 경로는 `WebStreamClient`의 문자열 메시지 모델을 기준으로 잡아야 합니다.
- Roblox 쪽 WebSocket 생성 경로는 공식 문서 검색상 표면이 불완전하므로,
  transport 구현을 코어 계약 뒤로 숨기는 것이 안전합니다.

## 도구별 강점

### Blender가 강한 영역

- 베지어 핸들 기반 커브 편집
- `Action`과 `NLA` 기반 클립 조합
- 리그, 제약, 포즈 기반 저작
- 대량 키프레임 편집과 그래프 편집

### Studio 플러그인이 강한 영역

- 실제 Roblox `Instance` 바인딩 해석
- 어댑터별 속성 감지와 메타데이터 관리
- 런타임 함수 실행형 이벤트 엔드포인트 연결
- 실제 게임 오브젝트 기준 미리보기와 디버깅

### 공유 가능한 영역

- 전역/로컬 마커
- 단순 오디오 배치
- 기본 Transform 채널
- 시간축 범위와 섹션 구간 정보

## 라운드트립 등급

- `Stable`
  - 양방향 왕복 후에도 구조 손실이 거의 없고, 기본 워크플로우에서 신뢰 가능합니다.
- `Limited`
  - 핵심 값은 왕복되지만, 일부 메타나 고급 의미는 축약되거나 프록시로 바뀝니다.
- `PreviewOnly`
  - 결과 미리보기나 샘플 값 전달은 가능하지만, 구조적 저작 정보로 왕복하지 않습니다.

## 항목별 권한 소스

### BindingGroup / Binding

- 기본 권한 소스: `Studio`
- 라운드트립 등급: `Limited`
- 이유:
  - 최종 대상은 Roblox `Instance` 트리와 어댑터 해석 결과에 의해 결정됩니다.
  - Blender는 대상 이름, 계층, 사용자 지정 태그를 보조 정보로 가질 수는 있지만,
    최종 `BindingId -> Instance` 매핑은 Studio에서 확정해야 합니다.
- Blender로 내보낼 때:
  - 안정 식별자, 표시 이름, 부모 관계, 선택적 경로 힌트만 전송합니다.
- Blender에서 가져올 때:
  - 새 바인딩 제안은 받을 수 있지만, 실제 엔트리 등록과 메타데이터 확정은 Studio에서 검증합니다.

### PropertyTrack / Section / Channel / Keyframe

- 기본 권한 소스: `Blender`
- 라운드트립 등급: `Stable` 또는 `Limited`
- `Stable` 범위:
  - Transform 계열
  - 기본 스칼라 채널
  - `Bezier`/`Linear`/`Constant` 보간
  - 섹션 구간, `EaseIn`, `EaseOut`, `RowIndex`, `OverlapPriority`
- `Limited` 범위:
  - 어댑터가 해석하는 커스텀 속성
  - Blender 전용 드라이버나 수식 기반 값
- 규칙:
  - `ActionDriven` 구현은 `ActionSlot.identifier`를 함께 저장해야 합니다.
  - `Action.fcurves` 직접 순회는 첫 슬롯 legacy 경로에만 안전하므로,
    슬롯/레이어를 고려한 추출 경로를 우선합니다.
  - 진짜 라운드트립이 필요하면 `ActionDriven`을 권장합니다.
  - `SceneSampled`는 구조 보존보다 최종 결과 샘플링에 가깝기 때문에,
    역방향에서는 베이크된 커브나 임시 `Action`으로 들어갈 수 있습니다.

### Sequence Marker / Local Marker

- 기본 권한 소스: `Shared`
- 라운드트립 등급: `Stable`
- 규칙:
  - `SceneSampled`에서는 Blender `Scene.timeline_markers`를 시퀀스 전역 마커로 매핑합니다.
  - `ActionDriven`에서는 `Action.pose_markers`를 Action 소유 마커로 우선 사용합니다.
  - 바인딩 로컬 마커는 바인딩 식별자와 함께 저장해야 하며,
    Action 소유 마커만으로 부족한 경우 보조 메타데이터를 함께 둡니다.
  - 같은 이름 충돌 정책은 런타임과 동일하게 로컬 우선입니다.

### EventTrack

- 기본 권한 소스: `Studio`
- 라운드트립 등급: `Limited`
- 이유:
  - Roblox 런타임 함수 호출, allowlist 엔드포인트, 바인딩 재해결은 Studio 쪽 의미가 더 강합니다.
  - Blender에는 이를 그대로 표현하는 기본 개념이 없습니다.
- Blender로 내보낼 때:
  - 이벤트 키를 프록시 마커, 커스텀 스트립 메타, 또는 커스텀 프로퍼티로 표현합니다.
  - `EndpointId`, `Payload`, 실행 방향 플래그를 보존 가능한 범위에서 직렬화합니다.
- Blender에서 가져올 때:
  - 프록시 표현이 있으면 `EventTrack`으로 복원합니다.
  - 실행 의미가 불분명하면 마커로만 복원하고 진단을 남깁니다.

### AudioTrack

- 기본 권한 소스: `Shared`
- 라운드트립 등급: `Limited`
- `Stable`에 가까운 범위:
  - 시작 시간
  - 길이
  - 기본 오디오 에셋 ID
  - 볼륨/피치 기본 곡선
- 손실 가능 범위:
  - `AttachBindingId`
  - Roblox 공간 음향 연결 정책
  - Blender `Sound Strip` 외의 런타임 재생 옵션
  - Blender `Sound Strip`은 재생 시작 시점을 제어하지만, 재생 길이가 strip 길이와 완전히 같지 않을 수 있습니다.

### ConstraintTrack

- 기본 권한 소스: `Blender`
- 라운드트립 등급: `Limited`
- `Stable`에 가까운 범위:
  - `Parent`, `Position`, `Rotation`, `Scale`, `LookAt`
  - 단순 가중치 곡선
- 손실 가능 범위:
  - Blender 전용 제약 옵션
  - IK Solver 세부 설정
  - Driver, Helper Bone, Rig 전용 보조 상태
- 규칙:
  - Studio 역방향 동기화는 공통 부분집합만 복원합니다.
  - 복원할 수 없는 Blender 전용 의미는 섹션 메타 또는 진단으로 남깁니다.

### Adapter Metadata

- 기본 권한 소스: `Studio`
- 라운드트립 등급: `Limited`
- 이유:
  - `METADATA_<ADAPTER>_<PROP>`는 Roblox 인스턴스와 어댑터 계약에 종속됩니다.
  - Blender는 이 값을 읽기 전용 미러나 디버그 정보로 가질 수는 있지만,
    최종 해석 기준은 Studio 어댑터입니다.

### Runtime Parameters

- 기본 권한 소스: `Studio Runtime`
- 라운드트립 등급: `PreviewOnly`
- 규칙:
  - `AnimationTrack:GetParameter` 계열 값은 런타임 상태입니다.
  - 영속 데이터나 Blender 저작 구조로 직렬화하지 않습니다.

### UI / Visualization

- 기본 권한 소스: 각 편집기 로컬
- 라운드트립 등급: `PreviewOnly`
- 규칙:
  - 패널 배치, 선택 강조, 편집기 오버레이는 문서화된 데이터 모델에 넣지 않습니다.
  - Blender 뷰포트 표현과 Studio 뷰포트 표현은 비슷할 수는 있어도 같은 데이터로 강제하지 않습니다.

## 방향별 설계

### Blender -> Studio

1. Blender는 `SceneSampled` 또는 `ActionDriven`으로 데이터를 추출합니다.
1. 추출 결과는 [Transform Sync](./transform-sync.md) 패킷 규약으로 전송합니다.
1. Studio는 어댑터를 통해 실제 바인딩과 속성을 해석합니다.
1. 지원 가능한 항목은 `AnimationFile` 또는 `AnimationData`로 정규화합니다.
1. 손실 항목은 진단으로 누적하고, 가능하면 프록시 마커/메타로 보존합니다.

이 방향은 정밀 저작의 기본 경로입니다.

### Studio -> Blender

1. Studio는 왕복 가능한 항목만 선택적으로 내보냅니다.
1. `SceneSampled` 대상이면 현재 평가 결과를 베이크된 커브나 샘플 값으로 보냅니다.
1. `ActionDriven` 대상이면 가능한 범위에서 `Action`, `Strip`, `Marker`, `Sound Strip`으로 재구성합니다.
1. 어댑터 메타데이터와 함수 실행 의미는 Blender 네이티브 의미로 강제하지 않고,
   커스텀 프로퍼티나 보조 메타로 전달합니다.
1. 재구성 불가능한 의미는 생략하지 않고 진단으로 알립니다.

이 방향은 "역저작"이 아니라 "제한적 복원"으로 취급합니다.

## Blender 구현 경로

### SceneSampled 추출

- `scene.frame_set(frame, subframe=...)`로 현재 Scene을 원하는 시각으로 이동합니다.
- `context.evaluated_depsgraph_get()`와 `evaluated_get(depsgraph)`를 사용해 평가 결과를 읽습니다.
- 이 경로를 기본값으로 삼아야 제약, 드라이버, 모디파이어, NLA 누적 결과가 반영됩니다.
- 단순 `FCurve.evaluate()`만으로는 Scene 전체 평가 결과를 재현할 수 없습니다.

즉, `SceneSampled`는 "곡선 샘플링"이 아니라 "평가된 Scene 스냅샷 샘플링"으로 구현해야 합니다.

### ActionDriven 추출

- `AnimData.action`, `AnimData.action_slot`, `AnimData.nla_tracks`를 기준으로 활성 Action과 NLA 배치를 읽습니다.
- `Action.fcurves`와 `Action.groups`는 최신 Blender API에서 legacy 경로이므로,
  첫 번째 슬롯만 지원하는 임시 구현이 아니라면 그대로 단일 원본으로 쓰면 안 됩니다.
- 최소 저장 단위는 다음을 권장합니다.
  - `ActionName`
  - `ActionSlotIdentifier`
  - `NlaTrackName`
  - `StripLocalId`
  - `DataPath`
  - `ArrayIndex`
- layered/slotted Action을 완전히 지원하지 못하는 단계에서는
  부분 지원 상태를 진단으로 명시하고 `SceneSampled` 폴백을 허용하는 편이 안전합니다.

### Blender 역구성

- F-Curve 생성은 Action 쪽 보조 API를 사용합니다.
  - 우선 경로: `Action.fcurve_ensure_for_datablock(...)`
  - 제한적 레거시 경로: `Action.fcurves.new(...)`
- 키프레임 생성은 `FCurve.keyframe_points.insert(...)`로 추가한 뒤,
  각 키의 `interpolation`, `handle_left`, `handle_right`, 핸들 타입을 채웁니다.
- 일괄 수정 후에는 다음 정리 단계가 필요합니다.
  - `keyframe_points.sort()`
  - `keyframe_points.deduplicate()`
  - `keyframe_points.handles_recalc()`
  - 필요 시 `fcurve.update()`
- `SceneSampled` 역방향 복원은 수동 키 삽입보다 bake 경로를 우선합니다.
  - `bpy.ops.nla.bake(...)`
  - 필요 시 `visual_keying=True`
  - 필요 시 `channel_types`에 `PROPS` 포함

### NLA 재구성

- Track 생성은 `AnimData.nla_tracks.new(...)`로 처리합니다.
- Action Clip 생성은 `NlaTrack.strips.new(name, start, action)`로 처리합니다.
- 하지만 `Transition`, `Meta`, `Sound`는 컬렉션 API만으로 끝나지 않습니다.
  - `bpy.ops.nla.transition_add()`
  - `bpy.ops.nla.meta_add()`
  - `bpy.ops.nla.soundclip_add()`
- 따라서 역방향 복원 파이프라인은 두 단계로 나누는 편이 좋습니다.
  1. 데이터 기반 생성이 가능한 Track/Action Clip 먼저 복원
  1. operator 문맥이 필요한 Strip 타입은 후처리 단계에서 복원

### 프록시 메타데이터

- Blender 커스텀 프로퍼티는 모든 RNA 타입에 자유롭게 붙는 것이 아닙니다.
- 공식 API 기준으로 커스텀 프로퍼티는 주로 `ID`, `Bone`, `PoseBone` 계열에 두는 것이 안전합니다.
- 복잡한 구조는 add-on이 등록한 `PropertyGroup`으로 노출하는 편이 좋습니다.
- 따라서 `EventTrack` 프록시, 어댑터 메타 미러, 라운드트립 진단 키는
  `Object`, `Armature`, `Bone`, `PoseBone`, 또는 등록된 `PropertyGroup`에 배치하는 것을 권장합니다.

## Roblox Studio transport 구현

### WebSocket 경로

- 공식 문서 기준 `WebStreamClient`는 다음 성질이 확인됩니다.
  - `Not Creatable`
  - `Send(data: string)`
  - `MessageReceived(message: string)`
  - `Opened(responseStatusCode, headers)`
  - `Error(responseStatusCode, errorMessage)`
  - `Closed()`
- 따라서 실시간 동기화 프로토콜은 binary frame 전제보다 문자열 메시지 전제를 먼저 두는 편이 맞습니다.
- 권장 포맷은 JSON envelope입니다.
  - `ProtocolVersion`
  - `SessionId`
  - `Revision`
  - `PacketType`
  - `Payload`
  - `FeatureFlags`
- 큰 스냅샷은 chunk 단위로 쪼개고,
  실시간 미리보기는 작은 `Delta` 메시지 위주로 유지해야 합니다.

### 생성 경로 추상화

- 공식 Creator Hub 검색 기준으로는 `WebStreamClient` 객체 자체는 보이지만,
  현재 public factory 경로는 문서 표면이 불명확합니다.
- 이 상태에서 코어가 특정 생성 API 이름에 직접 의존하면 문서/엔진 변동에 취약합니다.
- 따라서 Studio 쪽 구현은 `StudioSyncTransport` 같은 추상 계약 뒤로 숨기는 것을 권장합니다.

```lua
type StudioSyncTransport = {
    Connect: (url: string, headers: {[string]: string}?) -> (),
    SendPacket: (packet: SyncPacketEnvelope) -> (),
    Close: () -> (),
    ConnectionStateChanged: RBXScriptSignal,
    PacketReceived: RBXScriptSignal,
    TransportError: RBXScriptSignal,
}
```

- 구현체는 최소 3개를 권장합니다.
  - `WebSocketTransport`
  - `HttpPollingTransport`
  - `FileImportTransport`

### HTTP 폴백

- 공식 스크립트 capability 문서는 네트워크 기능으로
  `HttpService:GetAsync()`, `RequestAsync()`, `PostAsync()`, `UrlEncode()`, `GetSecret()`를 명시합니다.
- 따라서 WebSocket 경로가 불가능하거나 불안정한 Studio 빌드에서는
  `RequestAsync()` 기반 폴링/롱폴링 폴백을 두는 것이 현실적입니다.
- 이 폴백은 실시간성은 떨어지지만, 초기 스냅샷/수동 동기화/재동기화에는 충분합니다.

### 패킷 적용 루프

1. transport가 문자열 메시지를 수신합니다.
1. 코어가 JSON decode 후 `ProtocolVersion`과 `PacketType`을 검증합니다.
1. `Revision` 순서를 확인해 누락 시 `RequestResync`를 보냅니다.
1. 적용 가능한 항목만 `AnimationFile` 또는 `AnimationData`에 반영합니다.
1. 손실/거부 항목은 진단으로 누적하고 `Ack`에 요약을 포함합니다.

## 권장 구현 순서

1. `SceneSampled`의 Blender -> Studio 실시간 프리뷰를 먼저 완성합니다.
1. Studio transport는 `WebSocketTransport`와 `FileImportTransport`를 먼저 붙입니다.
1. Blender 역방향은 `SceneSampled` baked import를 먼저 구현합니다.
1. 그 다음 `ActionDriven`를 slot-aware Action 경로로 확장합니다.
1. 마지막으로 `Transition`, `Meta`, `Sound`의 operator 기반 NLA 재구성을 추가합니다.

## `SceneSampled`와 `ActionDriven`의 차이

### SceneSampled

- 초보자 프리뷰와 결과 일치성에는 유리합니다.
- 하지만 역방향 동기화 시 구조를 되살리기 어렵습니다.
- 따라서 양방향 편집보다는 `Blender -> Studio` 저작 경로에 적합합니다.

### ActionDriven

- 진짜 라운드트립에 더 적합합니다.
- Action, Strip, 섹션, 오디오 배치를 개별 구조로 남길 수 있습니다.
- 반대로 규칙이 더 엄격하고, Action 식별자와 참조 검증이 필요합니다.

## 충돌 처리 기준

- 같은 항목에 대해 두 편집기에서 동시에 수정이 들어오면, 항목별 기본 권한 소스를 먼저 봅니다.
- 같은 권한 등급 안에서는 [Sync Policy](./sync-policy.md)의 활성 권한 소스와 `Revision` 규칙을 따릅니다.
- 손실이 예상되는 역방향 동기화는 자동 병합하지 않고 진단을 우선 남깁니다.

## 권장 기본값

- 초보 사용자:
  - `SceneSampled`
  - `Blender -> Studio` 중심
  - `Studio -> Blender`는 미리보기나 마커 복원 정도만 사용
- 고급 사용자:
  - `ActionDriven`
  - 곡선/클립/제약 저작은 Blender 중심
  - 바인딩 해석, 이벤트 엔드포인트, 어댑터 메타데이터는 Studio 중심

## 구현 요구사항

- 모든 손실 변환은 `BindingId`, `TrackName`, `PropertyKey`, 시간 범위 중 가능한 식별 정보를 포함해 진단합니다.
- 역방향 가져오기에서는 "복원 실패"와 "의도적 축약"을 구분해 표시합니다.
- 프리뷰 전송과 저장 전송은 같은 패킷 계층을 사용하되, 권한 소스 판정은 transport 계층에 넣지 않습니다.
- 최소 진단 코드 집합을 권장합니다.
  - `UnsupportedLayeredAction`
  - `UnsupportedActionSlot`
  - `OperatorContextUnavailable`
  - `AudioStripLengthLossy`
  - `MissingWebSocketFactory`
  - `UnsupportedCustomPropertyHost`
