# Enums

데이터 구조에서 사용되는 열거형(Enumeration) 정의입니다.
모든 Enum 값은 **문자열(String)**로 저장됩니다.
이는 `TypeInfos`를 통한 커스텀 Enum 처리를 기반으로 합니다.

## EasingStyle

보간 곡선의 형태를 정의합니다. (TweenService 스타일)

*   `"Linear"`
*   `"Sine"`
*   `"Back"`
*   `"Quad"`
*   `"Quart"`
*   `"Quint"`
*   `"Bounce"`
*   `"Elastic"`
*   `"Exponential"`
*   `"Circular"`
*   `"Cubic"`

## EasingDirection

보간의 적용 방향을 정의합니다.

*   `"In"`
*   `"Out"`
*   `"InOut"`
*   `"OutIn"`

## InterpolationMode

키프레임 간의 보간 방식을 정의합니다.

*   `"Constant"`: 보간 없이 해당 키프레임 값을 유지합니다 (Step).
*   `"Linear"`: 선형 보간을 수행합니다.
*   `"Bezier"`: 베지어 곡선 보간을 수행합니다. (핸들 속성 필요)

## HandleType

베지어 핸들의 동작 방식을 정의합니다. (Blender 호환)

*   `"Free"`: 핸들을 독립적으로 조절할 수 있습니다 (방향/길이 개별).
*   `"Aligned"`: 좌우 핸들의 방향이 일직선으로 유지되지만 길이는 독립적입니다.
*   `"Vector"`: 자동 선형 보간을 생성합니다 (핸들 조작 시 Free로 변경).
*   `"Automatic"`: 부드러운 곡선을 위해 핸들 위치가 자동 계산됩니다.
*   `"AutoClamped"`: 오버슈트(Overshoot)를 방지하며 자동으로 핸들이 계산됩니다.
