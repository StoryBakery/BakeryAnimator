# Architecture Design Principles (설계 원칙)

BakeryAnimator 아키텍처가 반드시 따라야 할 핵심 가치와 설계 철학입니다.
모든 기능 추가와 리팩토링은 이 원칙을 기준으로 결정됩니다.

## 1. Hierarchy Visualization (계층의 시각화)
*   **"구조가 곧 기능이다."**
*   `Sequence -> Actor -> Track -> Property -> Layer -> Value` 로 이어지는 데이터의 흐름이 한 눈에 파악되어야 합니다.
*   복잡한 로직일수록 폴더/파일 구조와 문서 목차에서 그 위계가 명확히 드러나야 합니다.

## 2. Industry Standardization (커뮤니티 표준 준수)
*   **"바퀴를 다시 발명하지 않는다."**
*   가급적 **Unreal Engine(Sequencer)**, **Unity(Timeline)**, **Blender(NLA)** 등 업계 표준 툴의 용어와 개념을 차용합니다.
    *   예: `Sequence`, `Track`, `Clip`, `Keyframe`, `BlendMode`.
*   로블록스 개발자들에게 익숙한 용어(`AnimationTrack`, `Motor6D`)와도 자연스럽게 매핑되어야 합니다.

## 3. Scalability & Recursion (확장성과 재귀성)
*   **"하나는 전체와 같다."**
*   단일 캐릭터를 제어하는 구조가 그대로 다중 캐릭터(Crowd) 제어에도 쓰여야 합니다.
*   애니메이션 파일은 다른 애니메이션 파일을 포함(`Clip`)할 수 있어야 하며, 이 깊이에는 제한이 없어야 합니다.

## 4. Mathematical Precision (수학적 정밀함)
*   **"감(Feel)이 아닌 수식(Math)으로."**
*   모든 움직임은 `Set`, `Add`, `Multiply` 등 명확한 수학적 연산으로 정의되어야 합니다.
*   모호한 "블렌딩" 대신 `Weight`와 `Priority`에 기반한 결정로직(`ValueByPriority`)을 따릅니다.

## 5. Developer Convenience (편의성)
*   복잡한 내부 로직(`Evaluator`, `VBP Graph`)은 숨기고, 사용자는 직관적인 API(`Play`, `Stop`, `Load`)만으로 기능을 온전히 활용할 수 있어야 합니다.
