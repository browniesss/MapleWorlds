# PVP User Domain Technical Specification (v1.1)

> [!NOTE]
> 본 문서는 `Domain_User.md`의 요구사항과 `PVP_System.md`의 구조를 기술적으로 구체화한 명세서입니다. 구현 시 이 문서를 기준으로 코드를 작성하십시오.

## 1. 개요 (Overview)
PVP 유저 도메인은 유저의 **덱 설정, 전적(Stats), 입장권** 정보를 관리합니다.
데이터 무결성과 시즌제 운영을 위해 **Key-Based Versioning** 전략과 **Lazy Initialization** 패턴을 채택합니다.

## 2. 데이터 모델 (Data Models)

### 2.1 Storage Key Strategy
`UserDataStorage`를 사용하며, 시즌 데이터와 공용 데이터를 키(Key)로 구분합니다.

| 구분 | Key Format | 설명 |
| --- | --- | --- |
| **Season Stats** | `PVP_Stats_v{Version}` | 시즌별 독립적인 전적/점수 데이터 (예: `PVP_Stats_v2024S1`) |
| **Shared Deck** | `PVP_Deck` | 시즌과 무관하게 유지되는 덱 설정 |
| **Shared Ticket** | `PVP_Ticket` | 시즌과 무관하게 유지되는 입장권 정보 |

### 2.2 Data Structures (Type Definitions)

#### `Type: PVP_SeasonStats`
시즌별로 격리되는 유저의 성적표입니다.

```lua
-- @Type: PVP_SeasonStats
{
    Score        = 0,            -- integer (Current Score, Min 0)
    Tier         = "Bronze_4",   -- string (Enum: PVP_Tier)
    WinStreak    = 0,            -- integer (Current Win Streak)
    TotalWins    = 0,            -- integer
    TotalLosses  = 0,            -- integer
    DefenseLog   = {},           -- List<PVP_DefenseLog> (Max 20)
    LastUpdated  = 0             -- double (Timestamp)
}
```

#### `Type: PVP_DeckSettings`
유저가 설정한 공격/방어 덱 구성입니다.
> [!WARNING]
> 유닛의 레벨이나 스탯은 저장하지 않습니다. 전투 진입 시 반드시 `Inventory`에서 최신 정보를 조회해야 합니다.

```lua
-- @Type: PVP_DeckSettings
{
    OffenseDeck = {             -- List<DeckSlot> (Fixed Size: 8)
        [1] = { UnitId = "Warrior_01", Priority = 1 }, -- Level Removed
        ...
    },
    DefenseDeck = {             -- List<DeckSlot> (Fixed Size: 8)
        [1] = { UnitId = "Archer_05", Priority = 1 },
        ...
    }
}
```

#### `Type: PVP_TicketInfo`
입장권 수량 및 자동 충전 타이머 정보입니다.

```lua
-- @Type: PVP_TicketInfo
{
    Count           = 5,        -- integer (Current Holding Count)
    LastRecharge    = 0,        -- double (Timestamp of last recharge calculation)
    MaxCount        = 5         -- integer (Constant)
}
```

---

## 3. 로직 및 API 명세 (Logic & API Specs)

### 3.1 모듈 정의
*   **Module**: `PVPUserManager` (Logic)
*   **Dependencies**: `UserDataStorage`, `SortableGlobalDataStorage`, `PVP_SeasonManager`, `UserService`

### 3.2 Key Generation Logic
모든 시즌 데이터 접근 시 `PVP_SeasonManager`를 통해 현재 활성화된 시즌 키를 동적으로 생성해야 합니다.

```lua
-- Global Ranking Storage Key is also Versioned
function GetCurrentRankKey()
    return "PVP_Rank_Master_v" .. _PVP_SeasonManager:GetCurrentVersion()
end

function GetCurrentStatsKey()
    return "PVP_Stats_v" .. _PVP_SeasonManager:GetCurrentVersion()
end
```

### 3.3 Methods Specification

#### `[Server Only] GetProfile(userId)` -> `PVP_SeasonStats`
유저의 현재 시즌 전적을 조회합니다. **Lazy Initialization**이 적용됩니다.

1.  `GetCurrentStatsKey()`로 Key 획득.
2.  `UserDataStorage:Get(key)` 호출.
3.  **If Data exists**: 반환.
4.  **If Data is nil**:
    *   `CreateNewSeasonStats()`로 기본 구조체(`Score=0` 등) 생성.
    *   `UserDataStorage:Set(key, newData)`로 저장 (초기화).
    *   생성된 데이터 반환.

#### `[Server Only] SaveDeck(deckType, slots)` -> `boolean`
유저의 덱 설정을 검증하고 저장합니다.

1.  **Validation**:
    *   `slots` 개수가 8개인지 확인.
    *   `UnitId` 보유 여부 검증 (Level 정보는 무시/제거).
    *   중복 유닛 체크.
2.  **Save**:
    *   `UserDataStorage` Update.

#### `[Server Only] UpdateScore(userId, scoreChange)` -> `void`
점수 변동 시 개인 기록과 글로벌 랭킹을 동기화합니다.

1.  `GlobalKey` = `GetCurrentRankKey()`
2.  `StatsKey` = `GetCurrentStatsKey()`
3.  **Transaction-like Update**:
    *   `UserDataStorage:Get(StatsKey)` -> Update Score -> `Set`
    *   `SortableGlobalDataStorage:Get(GlobalKey)` -> Update Score -> `Set`
    *   (주의: MSW는 완벽한 트랜잭션을 지원하지 않으므로, 순차 실행하되 실패 로그를 남길 것)

#### `[Server Only] ConsumeTicket()` -> `boolean`
전투 진입 시 입장권을 차감하고 시간을 갱신합니다.

1.  `UserDataStorage:Get("PVP_Ticket")` 로드.
2.  **Recharge Calculation** (Correct Usage):
    *   `elapsed = CurrentTime - LastRecharge`
    *   `rechargedCount = floor(elapsed / 1800)`
    *   If `rechargedCount > 0` and `Count < MaxCount`:
        *   `Count = min(Count + rechargedCount, MaxCount)`
        *   `LastRecharge = LastRecharge + (rechargedCount * 1800)`
3.  **Consumption**:
    *   Check `Count > 0`.
    *   If `Count == MaxCount` (Full):
        *   `LastRecharge = CurrentTime` (타이머 시작)
    *   `Count = Count - 1`
4.  Save & Return `true`.

---

## 4. 제약 사항 및 예외 처리 (Constraints)

### 4.1 동시성 및 레이스 컨디션
*   **Server Authority**: 모든 데이터 변경 로직(`SaveDeck`, `ConsumeTicket` 등)은 반드시 **Server** 공간에서 실행되어야 합니다. (`[Server Only]` 명시 준수)
*   **Rank Sync**: 개인 데이터와 글로벌 랭킹 간 불일치가 발생하지 않도록 갱신 시점을 일원화(`UpdateScore`)해야 합니다.

### 4.2 시즌 전환 시점
*    API 호출(`GetProfile` 등) 시 자동으로 새로운 Key(`vNext`)를 참조하게 되므로 자연스럽게 데이터가 스위칭됩니다.
