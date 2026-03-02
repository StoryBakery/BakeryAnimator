---
title: ModeValidator
---

`ModeValidator`는 모드 등록 전 계약 위반을 검출하는 정적 검증 모듈입니다.

## 입력과 출력

```lua
(modeDefinition: ModeDefinition, existingModeIds: {[string]: boolean}) -> (ValidationResult)
```

```lua
type ValidationResult = {
    IsValid: boolean,
    Diagnostics: {ModeDiagnostic},
}
```

## 검증 항목

### 필수 필드

- `Manifest.Id`
- `Manifest.DisplayName`
- `Manifest.Version`

### 필수 함수

- `CanHandle`
- `DetectBindings`
- `DetectProperties`

### ID 충돌

이미 등록된 `Manifest.Id`와 중복되는지 검사합니다.

## 실패 처리

- 검증 실패 시 모드를 레지스트리에 등록하지 않습니다.
- 오류 상세는 `ValidationResult.Diagnostics`로 반환합니다.
- 예외(`throw`) 대신 진단 배열 누적을 사용합니다.
