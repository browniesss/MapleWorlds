# PROJECT_DOCS: 03. 성장 및 경제 시스템 (Progression & Economy)

본 문서는 유저의 게임 진행 상태 저장, 시너지 시스템 및 가챠(소환)를 포함한 경제 시스템에 대해 상세히 설명합니다.

---

## 1. 유저 진행 데이터 (User Stage Persistence)

`UserStage.mlua` 컴포넌트는 유저의 스테이지 진행 상태를 `DataStorage`에 영구적으로 저장하고 로드합니다.

### 1.1 데이터 카테고리
- **일반 모드 (Normal)**: `openChapter`, `openStage` 및 클리어 목록 (`clearedStages`).
- **하드 모드 (Hard)**: 하드 모드 전용 진행 데이터 및 클리어 목록.
- **이벤트 모드 (Event)**: `eventVersion`에 따라 초기화될 수 있는 시즌별 스테이지 데이터.

### 1.2 주요 기능
- **데이터 직렬화**: 테이블 형태의 클리어 목록을 쉼표로 구분된 문자열(`101,102,201...`)로 변환하여 저장합니다.
- **최초 보상 (Legacy Support)**: 이전에 누적된 클리어 스테이지 수에 따라 1회성 재화(메소) 보상을 지급하는 로직을 포함합니다.
- **버전 관리**: 서버의 `currentEventVersion`과 유저 데이터를 비교하여 새 시즌 시작 시 스테이지를 초기화합니다.

---

## 2. 시너지 시스템 (Synergy System)

플레이어가 보유한 유닛들의 조합에 따라 강력한 패시브 효과를 부여하는 핵심 시스템입니다.

### 2.1 시너지 계산 파이프라인
1. **덱 설정**: 유저가 선택한 유닛 덱 정보 로드.
2. **카운팅 (`UserSynergyComponent`)**: 덱의 각 유닛이 가진 속성(JobID, CountryID 등)을 집계하여 `countMap` 생성.
3. **유효 시너지 판정 (`SynergyRepo`)**: 현재 카운트가 시너지 발동 조건(예: 3마리 이상)을 충족하는지 확인.
4. **동적 적용**: 유닛 소환 시 `SpawnManager`에서 해당 속성에 맞는 `SynergyComponent`(예: `IncreaseHpByTime`)를 유닛에 부착.

---

## 3. 소환(가챠) 시스템 (Summon & Gacha)

`SummonService`를 통해 영웅/유닛을 획득하는 뽑기 시스템입니다.

### 3.1 가챠 로직
- **다양한 풀(Pool)**: `GachaSetting` 데이터셋에 정의된 여러 개의 소환 풀 지원.
- **천장 시스템 (Pity/Pick-up)**:
    - 소환 횟수(`summon_count`)를 누적하여 관리.
    - `pickup_char_limit` 도달 시 확정적으로 해당 픽업 캐릭터 지급.
    - 픽업 캐릭터 획득 시 횟수 초기화.
- **비용 처리**: `FirstCoin`(유료), `SecondCoin`(무료) 등 다양한 재화 타입을 연동하여 `BillingService`를 호출합니다.

---

## 4. 경제 인프라 (Billing & Shop)

- **BillingService**: 서버 사이드에서 재화의 가산/차감을 검증하며, 중복 요청 방지 및 트랜잭션 무결성을 유지합니다.
- **Cash Shop**: `ShopService`를 통해 유료 아이템 및 패키지를 판매하며, 구매 즉시 유저 데이터(인벤토리, 재화)를 갱신합니다.
- **Reward System**: 스테이지 클리어 결과에 따라 `HardModeMultiplier` 등을 적용하여 최종 보상을 계산하고 지급합니다.
