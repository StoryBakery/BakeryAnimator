---
title: Architecture Main
---

전체 설계도의 공통 원칙과 문서 진입점을 정의합니다.

## 공통 레이어 기준

- 에디터 UI/UX는 Blender 편집 경험을 기준으로 설계합니다.
- 데이터 구조는 Unreal Engine 시퀀스 계층을 기준으로 설계합니다.
- 실행 엔진과 적용 제약은 Roblox 런타임 규칙을 기준으로 설계합니다.

## 충돌 해결 우선순위

1. Roblox 실행 가능성
1. 데이터 구조 일관성
1. 에디터 UX 편의성

## 문서 진입점

- 애니메이터 코어: [Animator](./animator/index.md)
- 어댑터 시스템: [Adapter System](./adapter/index.md)
- 런타임: [Runtime](./runtime/index.md)
- 런타임 데이터: [AnimationData](./runtime/animation-data/index.md)
- 런타임 실행기: [AnimationTrack](./runtime/animation-track/index.md)
- 블렌더 애드온: [Blender Addon](./blender-addon/index.md)

## 레이어별 진실 원천

- `AnimationFile`과 그 하위 인스턴스 계층은 스튜디오 내 영속 편집 데이터의 진실 원천입니다.
- `AnimationData`는 인스턴스 계층이나 외부 동기화 결과를 런타임 평가용으로 정규화한 정적 데이터입니다.
- `LevelSequence`와 `AnimationTrack`은 재생 시간, 가중치, 파라미터 같은 런타임 상태의 진실 원천입니다.
- Blender 동기화 문서는 외부 DCC와 스튜디오 사이의 추출, 전송, 라운드트립 규약만 다룹니다.

즉, 같은 애니메이션이라도 "편집 저장", "런타임 평가", "외부 동기화"는 서로 다른 계층에서 책임집니다.

## 용어 경계

- `Adapter`: 실제 Roblox 인스턴스/속성 해석, UI 표시 보정, 시각화 확장을 담당합니다.
- `Sync Mode`: Blender 데이터를 어떤 형태로 추출하고 구조화할지 결정합니다.
- `LevelSequence`: 여러 바인딩 그룹/루트 바인딩을 묶는 시퀀스 디렉터입니다.
- `AnimationTrack`: 단일 바인딩 그룹 범위를 평가하는 런타임 실행기입니다.

## 단일 원본 규칙

- 공통 원칙(레이어 기준, 충돌 우선순위)은 이 문서만 수정합니다.
- 각 하위 설계 문서는 공통 원칙을 복제하지 않고 이 문서를 참조합니다.
- 타입 계약 원문은 각 계약 문서에서만 유지합니다.
- 어댑터 계약 원문은 [AdapterContract](./adapter/adapter-contract.md)에서만 유지합니다.
