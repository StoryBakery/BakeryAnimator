---
title: Graph Editor
---

Blender `Graph Editor` 기준으로 커브 편집 기능을 정의합니다.
이 문서는 `Timeline`과 분리된 곡선 편집기의 책임을 다룹니다.

## 목표

- 키 타이밍 편집(`Timeline`)과 커브 형태 편집(`GraphEditor`)을 분리합니다.
- 채널 기반 편집 모델(`F-Curve`)을 유지하면서 키프레임 중심 UX를 제공합니다.
- 런타임 데이터([Section Evaluator](../../runtime/animation-track/section-evaluator.md))와
  동일한 보간 의미를 사용합니다.

## Blender 정렬 기준

- `Graph Editor`는 시간(X) 대비 값(Y) 곡선을 직접 편집합니다.
- 채널은 속성 단위 곡선(`F-Curve`)이며, 키프레임과 핸들로 곡선 형태를 제어합니다.
- 채널 영역은 트리 기반이며 채널 필터와 토글 상태를 지원합니다.
- `F-Curve Modifier`는 비파괴 스택이며 위에서 아래 순서로 평가됩니다.
- 기본 단축키는 Blender `Graph Editor` 단축키를 우선 매핑합니다.
  - 단, Roblox 스튜디오 고유 단축키와 충돌하면 Roblox 단축키를 우선합니다.

## 영역 구성

### Curve View

- 곡선, 키프레임, 핸들을 표시합니다.
- 수평/수직 팬, 확대/축소, 현재 프레임 포커스를 지원합니다.
- 핸들 표시 토글과 선택 키의 핸들만 표시하는 옵션을 제공합니다.
- 변형 후 같은 프레임으로 합쳐진 키 자동 병합 옵션을 제공합니다.

### Channels Region

- `BindingGroup/Binding/Track/Channel` 트리를 표시합니다.
- 검색 필터를 제공합니다.
- 채널 헤더 토글을 제공합니다.
  - `Pin`
  - `Hide`
  - `Modifiers`
  - `Mute`
  - `Lock`
- 채널 헤더 더블클릭으로 해당 채널 키 전체 선택을 지원합니다.

### Active Keyframe Inspector

- 활성 키의 `Frame`, `Value` 편집을 제공합니다.
- 보간 모드를 제공합니다.
  - `Constant`
  - `Linear`
  - `Bezier`
- 이징 타입을 제공합니다.
  - `Automatic Easing`
  - `Ease In`
  - `Ease Out`
  - `Ease In Out`
- 베지어 핸들 타입을 제공합니다.
  - `Automatic`
  - `Auto Clamped`
  - `Vector`
  - `Aligned`
  - `Free`

### Modifiers Panel

- 채널별 `F-Curve Modifier` 스택을 제공합니다.
- 스택은 위에서 아래 순서로 평가합니다.
- 공통 속성을 제공합니다.
  - `Mute`
  - `Influence`
  - `Restrict Frame Range` (`Start`, `End`, `Blend In`, `Blend Out`)
- 1차 지원 타입:
  - `Generator`
  - `Built-in Function`
  - `Envelope`
  - `Cycles`
  - `Noise`
  - `Limits`
  - `Stepped Interpolation`

## 데이터 매핑

- `GraphEditor`는 `Section` 단위 편집이 아니라 `Channel`/`Keyframe` 편집을 담당합니다.
- 키 데이터는 런타임 타입과 1:1 대응합니다.
  - `ChannelData.Keys[]`
  - `KeyframeData.Time`
  - `KeyframeData.Value`
  - `KeyframeData.InterpolationMode`
  - `KeyframeData.LeftHandle`
  - `KeyframeData.RightHandle`
- 편집기 전용 상태(핀/잠금/표시/필터)는 세션 상태로만 관리하고 파일에 직렬화하지 않습니다.

## 구현 단계

1. `GraphEditor` 기본 편집
   - 곡선 표시, 키 이동/스케일, `Constant/Linear/Bezier`
1. 핸들/이징 확장
   - 핸들 타입, 핸들 좌표 편집, 이징 타입
1. 채널 영역 완성
   - 검색, `Pin/Hide/Mute/Lock`, 채널 일괄 선택
1. Modifier 스택
   - 공통 인터페이스 + 핵심 Modifier 타입

## 참고

- [Blender Manual - Graph Editor Introduction](https://docs.blender.org/manual/en/latest/editors/graph_editor/introduction.html)
- [Blender Manual - Graph Editor Channels](https://docs.blender.org/manual/en/latest/editors/graph_editor/channels/introduction.html)
- [Blender Manual - F-Curve Properties](https://docs.blender.org/manual/en/latest/editors/graph_editor/fcurves/properties.html)
- [Blender Manual - F-Curve Modifiers](https://docs.blender.org/manual/en/latest/editors/graph_editor/fcurves/modifiers.html)
