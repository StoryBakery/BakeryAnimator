# Animation System Structure

BakeryAnimator는 **시네마틱(Cinematic)**과 **대규모 군중 제어(Crowd Control)**를 지원하기 위해, **단일 트랙이 여러 모델을 동시에 제어**할 수 있는 강력한 계층 구조를 가집니다.

## Hierarchical Tree

**Sequence**가 **Track**들을 거느리고, **Track**이 **Model**의 **Property**를 제어하는 구조입니다.

```text
AnimationSequence (Director)
 ├── Clips (Nested AnimationData / Reusable Blocks)
 │    ├── [Clip: CameraWork.anim]
 │    └── [Clip: BackgroundFX.anim]
 │
 └── Models (Virtual Mapping Group)
      │
      ├── Model: Hero (Target: workspace.Dummy)
      │    └── AnimationTrack (Player)
      │         │
      │         ├── Clips (Nested AnimationData for this Model)
      │         │    └── [Clip: WalkCycle.anim]
      │         │
      │         ├── Object: RootPart
      │         │    └── Property: CFrame
      │         │         ├── Layer: Base (WeightedSet)
      │         │         └── Layer: Recoil (Add)
      │         │
      │         └── Object: Head
      │              └── ...
      │
      └── Model: Enemy
           └── AnimationTrack (Player)
                └── ...
```

## Core Components

### 1. AnimationSequence (The Director)
*   **역할**: 여러 배우(Actor)가 등장하는 **컷신(Cutscene)**이나 **복합 연출**을 총괄합니다.
*   **특징**:
    *   `Map<ActorName, Model>`을 통해 역할을 배정합니다.
    *   여러 `AnimationTrack`을 생성하고 동기화(Sync)를 맞춥니다.

### 2. AnimationTrack (The Player)
*   **역할**: **단일 모델(Model)**에 대한 애니메이션 재생 단위입니다. 로블록스의 `AnimationTrack`과 유사하지만, VBP 시스템과 통합되어 있습니다.
*   **특징**:
    *   `Animator`에 연결되어 VBP 노드 값을 공급합니다.
    *   독립적으로도 사용 가능합니다. (단일 캐릭터 단순 동작)

### 3. Animator (Per-Model Engine)
*   **역할**: 개별 모델(`Model`)에 부착되어, 해당 모델의 **모든 속성 값(Properties)**을 계산합니다.
*   **특징**:
    *   **ValueByPriority Evaluator**: 각 속성(`Property`)마다 `ValueByPriority` 그래프를 가집니다.
    *   **Layer Composition**: 여러 트랙(또는 하나의 트랙 내의 여러 레이어)이 보내는 신호를 VBP 로직(`Set`, `Add`, `Mul`)으로 합성합니다.

### 3. Layer (VBP Node)
*   **역할**: 특정 속성(`Property`)에 가해지는 **하나의 영향력(Influence)**입니다.
*   **특징**:
    *   기존의 단순 포토샵 레이어 개념을 넘어, **`ValueByPriority`의 노드(Node)** 하나에 해당합니다.
    *   **Priority & Function**: 우선순위와 합성 함수(`WeightedSet`, `Add` 등)를 가집니다.
