---
name: bakery-animator-architecture
description: "Use when evolving BakeryAnimator architecture with a mode-driven, highly extensible direction. Apply when deciding boundaries between animator core and modes, including UI/UX customization, runtime behavior replacement, and compatibility-preserving contract changes across `architectures/animator`, `architectures/runtime`, and `architectures/mode`."
---

# Bakery Animator Architecture

## Overview

이 스킬은 BakeryAnimator를 "기본 기능이 강한 코어 + 시스템 교체가 가능한 모드" 구조로
지속 발전시키기 위한 의사결정 가이드입니다.
아키텍처 규칙 원문은 항상 `architectures/**` 문서에 유지합니다.

## Current Direction

1. 플랫폼화
   - Animator 코어는 강력한 UI/UX와 기본 애니메이터 기능을 제공합니다.
   - 모드는 코어 위에서 동작 의미를 바꾸거나 확장합니다.
1. 모드 중심 확장
   - 모드는 인식/감지/적용/런타임 훅뿐 아니라 UI 표시 보정까지 제어할 수 있어야 합니다.
   - 필요하면 기존 워크플로우와 다른 애니메이션 시스템도 모드로 수용 가능해야 합니다.
1. 기본 경로와 확장 경로 분리
   - `DefaultRoblox`는 기본 경로를 담당합니다.
   - 제3자 모드는 같은 계약에서 고급/대체 동작을 구현합니다.
1. 계약 안정성
   - 코어와 모드의 경계는 계약으로 고정하고, 새 기능은 optional 훅으로 확장합니다.

## Boundary Model

- 코어 책임:
  - 타임라인/커브 에디터/선택/편집 UX
  - 기본 재생 제어, 파일 구조, 공통 파이프라인
  - 모드 로딩/검증/선택
- 모드 책임:
  - 바인딩 감지와 속성 채널 해석
  - 런타임 동작과 이벤트 엔드포인트 해석
  - 표시 이름/그룹/숨김 등 UI 메타 보정
  - 모드별 메타데이터 스키마와 재해결 전략

## Working Process

1. 요청 분류
   - 코어 기능 강화인지, 모드 확장인지, 계약 변경인지 먼저 분류합니다.
1. 계약 우선 설계
   - 확장 요구는 먼저 `mode-contract`의 optional 훅으로 설계합니다.
1. 기본 모드 폴백 유지
   - 새 훅이 없어도 `DefaultRoblox`가 기존 동작을 유지하도록 설계합니다.
1. 문서 반영
   - 방향/계약/구현 문서를 책임 경계에 맞춰 각각 반영합니다.
1. 연쇄 동기화
   - 용어/트리/링크/예시를 관련 문서에서 동시에 갱신합니다.

## Priority Rules

의사결정 우선순위는 다음 순서를 따릅니다.

1. 모드 확장 여지를 충분히 열어두는가
1. 기본 모드 호환성을 깨지 않는가
1. 코어 책임과 모드 책임이 섞이지 않는가
1. Unreal/Blender/Roblox 의미와 설명 가능하게 정렬되는가

## Document Responsibility

- 방향/정책: `architectures/mode/index.md`
- 계약: `architectures/mode/mode-contract.md`, `architectures/mode/mode-validator.md`
- 기본 모드 구현 정책: `architectures/mode/default-roblox-mode/index.md`
- 런타임 데이터/재생: `architectures/runtime/animation-data/*.md`, `architectures/runtime/animation-track/*.md`
- 에디터 인스턴스 계층: `architectures/animator/instances/*.md`

## Completion Checklist

1. 코어와 모드의 책임 경계가 문서에 명확합니다.
1. 새 확장이 optional 계약으로 추가되어 기존 기본 경로가 유지됩니다.
1. 모드가 UI 보정과 런타임 동작 확장을 수행할 수 있음을 문서로 설명합니다.
1. 아키텍처 규칙 원문은 `architectures/**`에 있고, 스킬은 운영 방향만 담습니다.
