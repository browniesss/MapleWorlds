# TechSpec_User.md Critical Review

**Reviewer**: Antigravity (Senior Architect)  
**Date**: 2026-02-02  
**Target Document**: `Doc/PvP/Spec/TechSpec_User.md`

## 1. 개요
해당 문서는 기획 의도를 기술적으로 잘 표현하고 있으나, **실제 메이플스토리 월드(MSW)의 mLua 문법과는 상이한 가상 코드(Pseudo-code)**가 포함되어 있어 개발자가 이를 그대로 복사하여 사용할 경우 컴파일 에러가 발생할 소지가 큽니다.

---

## 2. 주요 지적 사항 (Critical Issues)

### 2.1 mLua 문법과 맞지 않는 구조체 선언 (`struct`)
*   **지적 내용**: 문서에서 `struct PVP_SeasonStats { ... }` 문법을 사용하고 있습니다.
*   **문제점**: mLua(Lua)에는 `struct` 키워드가 존재하지 않습니다. 모든 데이터 구조는 **Table**로 관리되어야 합니다.
*   **권장 수정**:
    *   실제 코드에서는 주석으로 구조를 명시하거나, 초기화 함수(Factory Method)를 통해 Table을 반환하는 형태로 기술해야 합니다.
    *   예시는 가독성을 위한 의사 코드임을 명시해야 합니다.

### 2.2 실행 공간 어노테이션의 부정확성 (`@ExecSpace`)
*   **지적 내용**: `@ExecSpace("Server")`, `@ExecSpace("Client")`로 표기되어 있습니다.
*   **문제점**:
    *   MSW의 공식 어노테이션 명칭이나 에디터 상의 표기는 `[server only]`, `[client only]`, `[server]`, `[client]` 등으로 다양하나, 스크립트 상단에 명시할 때는 보편적으로 기입하지 않거나 MSW API 포맷에 맞춰야 합니다.
    *   특히 문자열 인자가 `"ServerOnly"`인지 `"Server"`인지 불분명하면 라이브러리나 CLI 도구가 인식하지 못할 수 있습니다.
*   **권장 수정**:
    *   MSW 프로퍼티 설정에 맞게 **Method Definition** 섹션에 `Method: [Server Only] void FunctionName()` 형태로 명확히 표기하거나,
    *   프로젝트 내부 규칙이라면 `@ExecSpace("ServerOnly")`로 통일성을 확인해야 합니다. (공식 가이드라인 참조 위함)

### 2.3 `UserDataStorage` 인스턴스 획득 로직 누락
*   **지적 내용**: 로직 예시에서 `UserDataStorage:SetAndWait`를 바로 호출하고 있습니다.
*   **문제점**:
    *   `PVPUserManager`는 `@Logic`이므로 특정 유저의 엔티티 컨텍스트가 없습니다.
    *   따라서 `_DataStorageService:GetUserDataStorage(userId)`를 통해 해당 유저의 스토리지 인스턴스를 먼저 가져오는 과정이 필수적입니다.
    *   현재 코드는 마치 `UserDataStorage`가 전역 정적 객체인 것처럼 오해를 불러일으킬 수 있습니다.

---

## 3. 상세 코드 및 예제 비교 (Code Discrepancy)

### 예제 코드 (In Document)
```lua
-- @Struct: PVP_SeasonStats
struct PVP_SeasonStats {
    integer Score = 0;
    ...
}

-- Method
UserDataStorage:SetAndWait(key, value)
```

### 실제 구현되어야 할 mLua 코드 (Actual mLua)
```lua
-- Table Definition (No struct keyword)
---@class PVP_SeasonStats
---@field Score integer
---@field Tier string
local PVP_SeasonStats = {
    Score = 0,
    Tier = "Bronze_4",
    ...
}

-- Method Implementation
local ds = _DataStorageService:GetUserDataStorage(userId) -- 필수
if ds then
    ds:SetAndWait(key, value)
end
```

## 4. 결론
이 문서는 '구조 설계서'로는 훌륭하나 '구현 지침서'로는 **문법적 오류**를 포함하고 있습니다. 개발자가 혼동하지 않도록 **"위 코드는 구조 설명을 위한 의사 코드(Pseudo-code)입니다"** 라는 문구를 상단에 배치하거나, 실제 mLua Table 문법으로 수정하여 배포하는 것을 권장합니다.
