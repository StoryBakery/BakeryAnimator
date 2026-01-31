# 베이커리 애니메이터 (Bakery Animator) 설계도

## Overview

본 문서는 로블록스(Roblox) 플랫폼을 기반으로 하는 **초확장성(Endgame) 애니메이션 시스템**의 아키텍처 엔트리 포인트입니다.
시스템은 크게 **데이터(Data)**, **런타임(Runtime)**, **도구(Tools)**로 구분됩니다.

## Architecture Map

### Data Structure
애니메이션 데이터의 스키마 및 클래스 정의입니다.

*   **[Node](./data-structure/node.md):** 최상위 추상 클래스.
*   **[Animation](./data-structure/animation.md):** 애니메이션 루트 컨테이너.
*   **[Track](./data-structure/track.md):** 개별 인스턴스 트랙 (Object Identity).
*   **[Channel](./data-structure/channel.md):** 속성 타임라인 (Value Timeline).
*   **[Keyframe](./data-structure/keyframe.md):** 데이터 포인트.
*   **[Enums](./data-structure/enums.md):** 열거형(`EasingStyle`, `EasingDirection`) 정의.

### Runtime System
게임 내에서 애니메이션을 로드하고 재생하는 핵심 로직입니다.

*   **[AnimationData](./runtime/animation-data.md):** 정적 데이터 컨테이너.
*   **[AnimationTrack](./runtime/animation-track.md):** 런타임 재생 제어기.
*   **[Detection](./runtime/detection.md):** 인스턴스 감지 및 매핑 전략.

### Tools & Editor
애니메이션을 저작하는 에디터와 외부 도구 연동입니다.

*   **[Studio Editor](./tools/studio-editor.md):** Vide 기반 에디터 및 플러그인 시스템.
*   **[Blender Integration](./tools/blender-integration.md):** 블렌더 애드온 및 동기화 프로토콜.

---

## Philosophy

*   **Extensibility:** 모든 요소(타입, 포맷, 감지, UI)의 커스터마이징.
*   **Format Agnosticism:** 특정 포맷에 종속되지 않는 유연한 구조.
*   **Superset:** 로블록스 기본 동작 위에서 가상의 데이터를 지원.

---

## Implementation Roadmap

1.  **Core:** `TypeInfos` 레지스트리 및 해석기 구현.
2.  **Detection:** `ObjectValue` 재귀 탐색 및 커스텀 감지 어댑터 구현.
3.  **UI Framework (Vide):** 확장 가능한 UI 컨테이너 및 컴포넌트 시스템 구축.
4.  **Runtime Adapter:** `AnimationTrack` 클래스 구현 및 프리뷰 시스템 연동.