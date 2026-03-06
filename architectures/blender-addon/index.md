---
title: Blender Addon
---

블렌더와 연동하여 로블록스와 양방향 통신이 가능한, 애니메이션 데이터
전송 및 동기화를 위한 애드온입니다.

## 왜 블렌더인가

블렌더는 로블록스에서 구현하기 어려운 정밀한 애니메이션을 구현할 수 있도록
도와줍니다.

복잡한 제약조건을 블렌더 내에서 구현하고, 로블록스 캐릭터 리그를 더 정확하게
다룰 수 있도록 도와줍니다.

## 문서 범위

이 문서는 블렌더 애드온의 전체 구현 계획을 다룹니다.
동기화 관련 상세 문서는 [Sync](./sync/index.md)에서 다룹니다.

## 목표

- 블렌더 타임라인의 포즈를 로블록스에서 같은 결과로 재현할 수 있어야 합니다.
- 파일 내보내기(`.sbbtra`)와 실시간 프리뷰(WebSocket)를 모두 지원합니다.
- FBX/glTF 변환 단계를 생략하고 Blender API에서 직접 데이터를 추출합니다.
- 양방향 동기화는 지원하되, 항목별 권한 소스와 라운드트립 한계를 명시적으로 유지합니다.

## MVP 범위

### 지원 대상

- Armature 기반 캐릭터 리그
- Bone 애니메이션
- Object Transform(Position/Rotation/Scale)

### 지원 기능

- 선택 대상 기준 샘플링 export (`.sbbtra`)
- 현재 타임라인 프레임 단위 미리보기 전송
- 로컬 좌표계 기준 데이터 추출 및 변환

### 제외 범위

- Shape Key / Material / Light 애니메이션
- 블렌더 외부 DCC 툴 직접 연동
- 양방향 편집 충돌 자동 병합
- Blender와 Studio를 완전 대칭 편집기로 만드는 것

## 애드온 구조

```text
blender_addon/
|- __init__.py
|- bl_info.py
|- properties/
|  |- scene_settings.py
|- ui/
|  |- panel_main.py
|- operators/
|  |- connect.py
|  |- disconnect.py
|  |- export_sbbtra.py
|  |- push_preview.py
|- services/
|  |- rig_scanner.py
|  |- sampler.py
|  |- transform_converter.py
|  |- serializer.py
|  |- websocket_client.py
|- models/
|  |- packet_types.py
```

## UI 설계

### Scene Settings

`server_url: string`
WebSocket 서버 주소입니다. 예: `ws://127.0.0.1:8765`

`target_armature: Object`
동기화할 Armature입니다.

`target_root: Object?`
내보내기 기준 루트 오브젝트입니다.

`sample_rate: number`
초당 샘플링 수(FPS)입니다.

`sync_mode: "SceneSampled" | "ActionDriven"`
블렌더 데이터 해석 모드입니다. 기본값은 `SceneSampled`입니다.
자세한 기준은 [Sync Mode](./sync/sync-mode.md)를 따릅니다.

`scene_scope: "ActiveScene" | "NamedScene"`
동기화 기준 Scene 선택 방식입니다. 기본값은 `ActiveScene`입니다.

`source_scene_name: string?`
`scene_scope = "NamedScene"`일 때 사용할 Scene 이름입니다.
기본값은 `"Scene"`입니다.
Scene 해석 실패 시에도 `"Scene"`으로 폴백합니다.

`initial_sync_source: "Auto" | "Blender" | "Studio"`
초기 동기화 기준 소스입니다. 기본값은 `Auto`입니다.
`Auto`는 값 존재 여부 기준으로 `Studio`/`Blender`를 선택합니다.
자세한 규칙은 [Sync Policy](./sync/sync-policy.md)의 초기 동기화 정책을 따릅니다.

`fill_empty_from_blender: boolean`
빈 값(`nil`/누락 필드)을 블렌더 값으로 채울지 여부입니다.
`Blender`가 초기 소스로 선택된 경우에만 적용됩니다. 기본값은 `true`입니다.
적용 조건은 [Sync Policy](./sync/sync-policy.md)를 기준으로 합니다.

### Panel Actions

- Connect: 서버 연결
- Disconnect: 서버 연결 해제
- Push Preview Frame: 현재 프레임 1회 전송
- Export `.sbbtra`: 프레임 범위 전체 내보내기

## 런타임 파이프라인

### 1. 대상 스캔

`target_armature`의 Bone 트리를 순회하고, 애니메이션 대상 채널 목록을 만듭니다.

### 2. 프레임 샘플링

프레임 범위(`frame_start` ~ `frame_end`)를 순회하며 각 Bone/Object의
Transform을 읽습니다.

### 3. 좌표계 변환

Blender 좌표를 Roblox 좌표로 변환하고, Local Delta 기준으로 정규화합니다.
변환 규칙은 [Transform Sync](./sync/transform-sync.md)와 동일해야 합니다.

### 4. 채널 최적화

변화가 없는 구간(Constant/Epsilon)을 제거해 데이터 크기를 줄입니다.

### 5. 직렬화

내부 모델을 `.sbbtra` 포맷으로 인코딩합니다.

### 6. 전송 또는 저장

- 실시간 모드: WebSocket으로 packet 전송
- 파일 모드: `.sbbtra` 파일 저장

## 작업 순서

1. 애드온 스캐폴딩과 등록/해제 코드 작성
2. Scene Property와 UI Panel 연결
3. Rig Scanner 구현
4. Sampler 구현
5. Transform Converter 구현
6. Serializer 구현 (`.sbbtra`)
7. WebSocket Client 구현
8. Export/Preview Operator 연결
9. 로그/에러 처리와 재시도 정책 추가

## 검증 체크리스트

- 블렌더 정지 포즈 1프레임 전송 시 로블록스 결과가 동일한가
- 키프레임 2개 사이 보간 결과가 시간축에서 일치하는가
- 축 반전/회전축 꼬임 없이 재현되는가
- 샘플링 FPS를 올려도 프레임 드랍 없이 export 가능한가
- 연결 끊김 후 재연결 시 상태가 정상 복구되는가

## 다음 문서

- [Sync](./sync/index.md): 동기화 문서 진입점
- [Sync Mode](./sync/sync-mode.md): Scene 평가값 샘플 기준/Action 기준 해석 모드
- [Sync Policy](./sync/sync-policy.md): 동기화 방향, 초기 소스, 충돌 처리 정책
- [Bidirectional Sync](./sync/bidirectional-sync.md): Blender/Studio 양방향 편집 범위와 권한 소스
- [Transform Sync](./sync/transform-sync.md): 좌표계, 패킷, Motor6D 변환 상세
