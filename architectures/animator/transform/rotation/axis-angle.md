---
title: Axis-Angle Rotation
---

**축-각도 (Axis-Angle)**는 임의의 하나의 축(Axis, $x, y, z$)과
그 중심축에 대한 회전 정도(Angle, $\theta$)를 사용하여 정의된 공간 변형 방식입니다.
이 회전법은 특정 객체가 제자리에서나 외부 기준축을 중심으로 특수한 공전(Orbit) 등 회전 거동을 원할 때
시스템의 한 도구로 활약할 여지를 두고 있습니다.

## 활용 목적과 확장성

- **IK(Inverse Kinematics) 솔버 호환**:
  오일러(Euler) 분해 방식이 다루기 까다로운 복잡한 조인트의 연속성을 구하거나 IK 연산을 뒷받침 할 때 수학적 우위를 지닙니다.
- **컴포넌트 변환**:
  BakeryAnimator 코어에서 CFrame `fromAxisAngle` 메서드 활용 및
  커스텀 로컬(Local) 좌표계에 따른 즉시 회전 변형을 연산하는 스크립팅 과정에 쓰입니다.
