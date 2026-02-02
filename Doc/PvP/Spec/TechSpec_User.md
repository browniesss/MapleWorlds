# PVP User Domain Technical Specification (TechSpec)

**Version**: 1.0  
**Authors**: Antigravity (Senior Architect)  
**Status**: Development Ready

## 1. 개요 (Overview)
본 문서는 `Spec_User.md`를 바탕으로, 메이플스토리 월드(MSW)의 mLua 개발 컨벤션과 프로젝트 표준을 준수하여 작성된 기술 명세서입니다. 본 명세는 유저의 PVP 성적, 덱 설정, 입장권 관리를 위한 `PVPUserManager` 핵심 로직을 정의합니다.

---

## 2. 규약 및 컨벤션 (Conventions)

### 2.1 실행 공간 제어 (Execution Space)
*   모든 데이터 스토리지 접근 및 자원 소모 로직은 서버 권한을 보장하기 위해 `@ExecSpace("Server")`를 사용하여 정의합니다.
*   클라이언트 UI 갱신을 위한 데이터 요청 시에만 `@ExecSpace("Client")`를 사용하며, 필요 시 RPC를 호출합니다.

### 2.2 데이터 직렬화 (Serialization)
*   `UserDataStorage`에 테이블 데이터를 저장하거나 로드할 때, 원자성과 표준 준수를 위해 `_HttpService:JSONEncode()` 및 `JSONDecode()`를 사용합니다.

---

## 3. 데이터 모델 보강 (Technical Data Models)

### 3.1 UserDataStorage Schema
*   **Key**: `PVP_Stats_v{SeasonVersion}` (예: `PVP_Stats_v1`)
*   **Value (JSON)**: `PVP_SeasonStats` 구조체

### 3.2 Struct Definitions (mLua)

```lua
-- @Struct: PVP_SeasonStats
struct PVP_SeasonStats {
    integer Score = 0;
    string Tier = "Bronze_4";
    integer WinStreak = 0;
    integer TotalWins = 0;
    integer TotalLosses = 0;
    table DefenseLog = {};     -- List of PVP_DefenseLog (JSON format)
}

-- @Struct: PVP_DeckSettings
struct PVP_DeckSettings {
    table OffenseDeck = {};    -- List of { string UnitId, integer Priority }
    table DefenseDeck = {};    -- List of { string UnitId, integer Priority }
}

-- @Struct: PVP_TicketInfo
struct PVP_TicketInfo {
    integer Count = 5;
    number LastRecharge = 0;   -- Server Timestamp
}
```

---

## 4. API 및 로직 명세 (Logic Specs)

### 4.1 Module: `PVPUserManager` (@Logic)

#### `README()`
*   스크립트의 역할: 유저 PVP 데이터(성적, 덱, 티켓) 통합 관리 및 스토리지 동기화.
*   주요 책임: 
    *   입장권 자동 충전 로직 수행.
    *   유저 점수 변동 시 `UserDataStorage` 및 `SortableGlobalDataStorage` 동시 갱신.

#### `@ExecSpace("Server")`
#### `void _rechargeTicket(PVP_TicketInfo ticketInfo)`
*   **Description**: 입장권 충전량을 계산하고 갱신합니다.
*   **Logic**:
    1.  `elapsed = _UtilLogic.ServerTimestamp - ticketInfo.LastRecharge`
    2.  `rechargeCount = math.floor(elapsed / 1800)`
    3.  `rechargeCount > 0`인 경우:
        - `ticketInfo.Count = math.min(ticketInfo.Count + rechargeCount, 5)`
        - `ticketInfo.LastRecharge = ticketInfo.LastRecharge + (rechargeCount * 1800)`

#### `@ExecSpace("Server")`
#### `boolean ConsumeTicket(string userId)`
*   **Description**: 전투 진입 시 입장권을 1매 소모합니다.
*   **Logic**:
    1.  `_rechargeTicket` 호출로 최신 정보 업데이트.
    2.  `ticketInfo.Count >= 1`인 경우:
        - 만약 `Count == 5`였다면 `LastRecharge`를 현재 서버 시간으로 설정.
        - `Count = Count - 1`
        - `JSONEncode` 후 `UserDataStorage:SetAndWait`
        - `true` 반환.
    3.  그 외 `false` 반환.

#### `@ExecSpace("Server")`
#### `void UpdateUserScore(string userId, integer scoreChange)`
*   **Description**: 전투 종료 후 점수를 갱신합니다.
*   **Logic**:
    1.  `UserDataStorage`에서 현재 시즌 성적표 로드 및 `JSONDecode`.
    2.  `Score` 정산 (Min 0 제한) 및 `Tier` 판정.
    3.  `UserDataStorage:Set` (개인 전적).
    4.  `SortableGlobalDataStorage:Set` (버전별 랭킹 보드).

---

## 5. 예외 처리 가이드 (Exception Handling)

*   **DataStorage Failure**: `SetAndWait` 및 `GetAndWait` 호출 시 에러 코드가 0이 아닌 경우 로그를 남기고, 유저에게 "데이터 동기화 실패" 알림을 전달해야 합니다.
*   **Validation**: 덱 저장 시 유저가 보유하지 않은 `UnitId`가 포함되어 있는지 반드시 `UserInventory` 서비스와 연동하여 검증합니다.
