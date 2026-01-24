# ⚔️ PVP System Architecture Design

본 문서는 MapleWorld 2D 디펜스 게임의 PVP(Player vs Player) 모드 구현을 위한 기술 설계서입니다. 프로젝트의 핵심 아키텍처인 **3-Tier(Client-Protocol-Server)** 구조를 계승하며, 확장성과 보안성을 고려하여 설계되었습니다.

---

## 1. 📂 폴더 구조 및 파일 구성

PVP 관련 로직은 `RootDesk/MyDesk/Scripts/PVP` 디렉토리에 위치합니다.

```text
PVP/
├── Design/
│   └── architecture.md          # 본 설계 문서
├── Server/
│   ├── Dto/
│   │   ├── UserPvpModel.mlua    # 유저 PVP 상태 (티어, 승수, 재화 등)
│   │   ├── PvpDeckModel.mlua    # 공격/방어 덱 데이터 구조
│   │   └── PvpShopItemModel.mlua # 상점 아이템 및 구매 제한 데이터
│   ├── PvpService.mlua          # 랭킹, 매칭, 재화 관리 핵심 로직
│   ├── PvpDeckService.mlua      # 유저 덱 저장 및 검증 로직
│   ├── PvpShopService.mlua      # 아이템 교환 및 구매 제한 처리
│   └── PvpAILogic.mlua          # 전투 시 상대방(AI) 소환 및 스킬 제어
├── Protocol/
│   ├── PvpLobbyProtocol.mlua    # 로비/상대 선택 관련 통신
│   ├── PvpBattleProtocol.mlua   # 전투 시작/결과 처리 통신
│   └── PvpShopProtocol.mlua     # 상점 거래 통신
└── Client/
    ├── PvpDeckStore.mlua        # 덱 설정 UI 상태 관리
    ├── PvpLobbyStore.mlua       # 상대 선택 및 랭킹 UI 상태 관리
    ├── PvpBattleManager.mlua    # 전투 화면 제어 및 AI 구동 트리거
    └── PvpShopStore.mlua        # 상점 UI 및 재화 상태 관리
```

---

## 2. 🏛️ 계층별 상세 설계

### 2.1. Server (Authority & Logic)
- **PvpService**: `_DataStorageService`를 통해 유저의 PVP 메타데이터를 관리합니다.
    - `SortableDataStorage`를 사용하여 티어별 랭킹 시스템을 구현합니다.
    - 유저가 부족할 경우 `PvpDummyData` Dataset에서 더미 데이터를 추출하여 매칭 리스트를 생성합니다.
    - 일일 플레이 재화(AP) 충전 로직을 포함합니다 (로그인 시각 기준 갱신).
- **PvpAILogic**: 전투 중 상대방(방어덱 유저)의 행동을 결정합니다.
    - **소환 규칙**: 방어덱에 설정된 유닛 인덱스(1~8) 순서대로 우선순위를 부여합니다. 소환 포인트가 부족하면 해당 우선순위 유닛을 뽑을 수 있을 때까지 대기합니다.
    - **스킬 규칙**: 쿨타임이 완료되는 즉시 가장 가까운 타겟을 향해 스킬을 시전합니다.

### 2.2. Protocol (Communication)
- 모든 요청은 `CommonCall`을 통해 `senderUserId`를 검증합니다.
- **PvpBattleProtocol:StartBattleCall**: 클라이언트에서 공격 버튼 클릭 시 호출됩니다. 서버에서 재화(AP) 차감 후 성공 여부를 `StartBattleBack`으로 전달합니다.
- **PvpBattleProtocol:EndBattleCall**: 전투 종료 시 승패 결과와 함께 호출됩니다. 서버에서 승리 여부를 재검증하고 보상(교환 재화)을 지급합니다.

### 2.3. Client (UI & Interaction)
- **PvpDeckStore**: 공격덱과 방어덱을 로컬에 임시 저장하고, '저장' 버튼 클릭 시 프로토콜을 통해 서버에 일괄 전송합니다.
- **PvpBattleManager**: 유저의 소환은 기존 `SpawnManager`를 활용하며, 상대방(AI)의 소환 요청은 `PvpAILogic`의 판단에 따라 서버에서 `SpawnEnemy` 함수를 호출하도록 트리거를 관리합니다.

---

## 3. 💾 데이터 설계 (Data Schema)

### 3.1. UserPvpModel (DTO)
- `tier`: string (Bronze, Silver, Gold, Platinum, Diamond)
- `rank`: integer (티어 내 순위)
- `winCount`: integer (누적 승리)
- `maxWinStreak`: integer (최다 연승)
- `bestTier`: string (최고 달성 티어)
- `playCurrency`: integer (PVP 입장권, 일일 5개)
- `exchangeCurrency`: integer (PVP 상점 재화)
- `lastUpdate`: integer (재화 충전 확인용 타임스탬프)

### 3.2. PvpDeckModel (DTO)
- `offenseDeck`: table (유닛 ID 리스트)
- `defenseDeck`: table (유닛 ID 리스트, 리스트 순서가 곧 AI 소환 우선순위)

### 3.3. PvpShop (Dataset)
- `id`: string
- `itemName`: string
- `price`: integer
- `purchaseLimit`: integer (일일/주간/영구 제한)
- `itemRUID`: string

---

## 4. ⚔️ 전투 시스템 상세 (Battle Logic)

1.  **초기화**: `PvpBattleProtocol:StartBattleCall` 성공 시 전용 맵으로 텔레포트합니다.
2.  **유저**: 기존 인게임 UI와 동일하게 수동으로 유닛을 소환합니다.
3.  **컴포넌트(AI)**: 
    - 서버의 `PvpAILogic`이 루프를 돌며 방어덱의 1번 유닛부터 소환 가능 여부를 체크합니다.
    - `MesoWallet`과 유사한 PVP 전용 AI 지갑을 통해 포인트를 관리합니다.
    - 유닛이 소환되면 인공지능 상태 머신(FSM)에 따라 전진 및 공격을 수행합니다.
4.  **승리 조건**: 상대 타워의 `Tower.hp`가 0이 되면 승리하며, 결과 팝업을 띄우고 보상을 수령합니다.

---

## 5. 🛠️ 향후 구현 시 주의사항
- **데이터 일관성**: 상점 구매 제한 데이터는 `UserDataStorage`에 별도로 저장하여 세션이 종료되어도 유지되어야 합니다.
- **더미 유저 처리**: 실제 유저가 적은 초기 단계에서는 Dataset 기반의 더미 유저가 랭킹 리스트의 70% 이상을 차지하도록 설계하여 컨텐츠 소모 속도를 조절합니다.
- **네트워크 최적화**: PVP 전투 중 발생하는 대량의 유닛 소환 정보는 `@Sync` 대신 프로토콜을 통한 RPC 호출로 흐름을 제어하여 부하를 최소화합니다.
