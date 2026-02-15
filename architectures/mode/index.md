## 개요

**Animation Mode**는 애니메이션 엔진이 다양한 대상을 제어할 수 있도록 도와주는 **"확장 플러그인"** 개념입니다.

기본적으로는 로블록스 인스턴스(Part, Motor6D)를 제어하지만, Mode를 교체하면 `UI`, `Camera`, 또는 `커스텀 데이터 구조`도 제어할 수 있습니다. 시스템의 유연성을 위해 "무엇을(Target), 어떻게(How) 제어할지"를 Mode가 정의합니다.

## 구성 요소

하나의 Mode는 다음 3가지 핵심 모듈을 구현하여 엔진에 제공합니다.

#### Detector
애니메이션을 재생할 `Root` 대상이 주어졌을 때, 하위 트랙(Track)들과 실제 조작 객체(Target)를 찾아냅니다.

#### Adapter
찾아낸 객체(Target)의 특정 속성(Property)에 값을 읽거나 씁니다.

#### Type Extension
*   **역할:** 기본 타입(Vector3, CFrame 등) 외에 커스텀 타입이 필요할 경우 정의합니다.
*   **필수 구현:** 각 타입을 숫자로 쪼개는 **스칼라 분해(Scalar Decomposition)** 규칙.

---

## 작동 흐름

애니메이션이 재생될 때 엔진은 **현재 활성화된 Mode**에게 구체적인 작업을 위임합니다.

1.  **탐색 (Binding):** `Mode.Detector`가 트랙 이름/경로에 맞는 실제 객체(Instance)를 찾습니다.
2.  **계산 (Calculation):** 엔진 코어(Animator)가 키프레임을 보간하여 현재 값을 계산합니다.
3.  **적용 (Application):** `Mode.Adapter`가 계산된 값을 실제 객체에 방금 찾은 경로로 적용합니다.

---

## 활용 예시

| Mode 이름    | 제어 대상             | 특징                                              |
| :----------- | :-------------------- | :------------------------------------------------ |
| **Standard** | `Model`, `BasePart`   | 로블록스 기본 캐릭터 및 사물 제어 (기본값)        |
| **UI**       | `GuiObject`           | Position, Size(UDim2), Transparency 제어          |
| **Camera**   | `Camera`              | FieldOfView, CFrame 직접 제어                     |
| **Virtual**  | `Table` (No Instance) | 인스턴스 없이 순수 데이터(Lua Table)만 시뮬레이션 |
