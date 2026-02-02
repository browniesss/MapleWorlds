---
name: msw-data-management
description: 메이플스토리 월드(MSW)의 영구 데이터 처리(DataSet, DataStorage)를 위한 전문 지침입니다.
---

# MSW Data Management (DataSet & DataStorage)

이 스킬은 메이플스토리 월드(MSW)에서 정적인 데이터셋(DataSet)을 조회하고, 동적인 NoSQL 스타일의 데이터 스토리지(DataStorage)를 사용하여 데이터를 영구적으로 저장 및 관리하는 방법을 안내합니다.

## 언제 이 스킬을 사용하는가 (When to use this skill)

- 아이템 정보, 몬스터 스탯 등 CSV 기반의 정적 데이터를 조회해야 할 때 (DataSet).
- 유저 아이템, 재화, 레벨 또는 월드 전체 랭킹 등 영구적인 저장이 필요한 데이터를 다룰 때 (DataStorage).
- 서버-클라이언트 간의 데이터 정합성을 유지하며 안정적인 DB 로직을 구현해야 할 때.

## 사용 방법 (How to use)

### 1. DataSet (정적 데이터 조회)
- **용도**: 읽기 전용 데이터 관리.
- **주요 API**:
    - `_DataService:GetTable("DataSetName")`: 데이터셋 로드.
    - `dataSet:GetCell(row, col)`: 특정 셀의 값을 문자열로 반환.

```lua
[ServerOnly]
void LoadItemInfo(integer itemId)
{
    local dataSet = _DataService:GetTable("ItemDataSet")
    -- 예: 1행은 컬럼명, 2행부터 데이터 시작
    local itemName = dataSet:GetCell(itemId + 1, 1)
    log("Loaded Item: " .. itemName)
}
```

### 2. DataStorage (동적 데이터 조작)
- **종류**: `GlobalDataStorage` (월드 전체), `UserDataStorage` (유저별), `CreatorDataStorage` (크리에이터용).
- **공간 제약**: 모든 DB 작업은 반드시 **[ServerOnly]** 환경에서 수행해야 합니다.
- **비동기 처리**: `Async` (콜백 기반) 또는 `AndWait` (동기식 대기) 중 상황에 맞는 접미사를 선택합니다.

### 3. 데이터 안정성 및 에러 핸들링
- **필수 로드 체크**: 유저 접속 시 필수 데이터 로딩에 실패하면 `_UserService:KickUser`를 통해 유저를 보호합니다.
- **정합성 유지**: `initialized` 플래그를 사용하여 데이터가 성공적으로 로드된 경우에만 저장을 허용합니다. (부분 저장 방지)
- **재시도 전략**: `TimedOut` 등 일시적인 네트워크 장애 시 일정 횟수 재시도 로직을 포함합니다.

### 4. 성능 및 Credit 최적화
- **데이터 묶음 저장 (JSON)**: 다수의 프로퍼티를 각각 저장하지 않고, 테이블로 묶어 `_HttpService:JSONEncode()`를 통해 단일 Key로 저장하여 Credit 소모를 최소화합니다.
- **저장 시점 (UserLeaveEvent)**: 실시간 저장이 불필요한 경우, 유저가 퇴장하는 시점인 `HandleUserLeaveEvent`에서 일괄 저장합니다. `OnEndPlay`는 저장 완료를 보장하지 않으므로 피해야 합니다.

### 5. 데이터 타입 변환
- `DataStorage`는 문자열(`string`)만 저장 가능하므로 숫자나 테이블은 변환이 필요합니다.
    - 테이블 <-> 문자열: `_HttpService:JSONEncode()`, `_HttpService:JSONDecode()`.
    - 숫자 <-> 문자열: `tostring()`, `tonumber()`.

## 의사 결정 가이드 (Decision Tree)

1. **데이터가 변하는가?**
    - No -> **DataSet** 사용 (CSV 관리).
    - Yes -> **DataStorage** 사용.
2. **누구의 데이터인가?**
    - 개별 플레이어 -> `_DataStorageService:GetUserDataStorage(userId)`.
    - 월드 공용/랭킹 -> `_DataStorageService:GetGlobalDataStorage("Name")`.
3. **데이터 로딩에 실패했는가?**
    - 필수 데이터인가? -> Yes: 유저 추방 또는 재시도 / No: 해당 기능만 차단.
4. **어제 저장해야 하는가?**
    - 실시간 저장이 필요한가? -> No (일반적): **UserLeaveEvent**에서 일괄 저장.
