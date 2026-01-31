# Instance Detection System

애니메이터가 조작할 대상(`Instance`)을 인식하고 트랙과 매핑하는 감지 전략(Detection Strategy)입니다.

## Overview

ReflectionService를 기반으로 하며, 필요에 따라 교체 가능한 전략 패턴(Strategy Pattern)을 사용합니다.

## Strategies

### Standard Strategy
기본 감지 방식입니다.

*   **Entry Point:** 사용자가 선택한 Model 또는 Folder.
*   **Structure Discovery:**
    *   단순 `GetDescendants()`가 아닌 설계된 구조 우선.
    *   **ObjectValue Link:** 계층 내 `ObjectValue`가 가리키는 대상을 논리적 자식(Sub-Track)으로 간주.
*   **Property Discovery:**
    *   감지된 인스턴스에 대해 `ReflectionService`로 조작 가능 속성(Property) 목록 추출.

### Custom Strategy
사용자 정의 `ModuleScript`를 통한 감지입니다.

*   `CollectionService` (Tag) 기반 감지.
*   이름 패턴 매칭 (Regex).
*   특수 리깅 구조 지원.
