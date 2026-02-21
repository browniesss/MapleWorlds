# PvP 시스템 MSW Studio 설정 가이드

> 이 문서는 PvP 스크립트 코드가 모두 작성된 상태에서,
> MSW Studio에서 **엔티티 생성 / 컴포넌트 부착 / UI 계층 구조 / 프로퍼티 바인딩**을
> 올바르게 설정하기 위한 단계별 가이드입니다.

---

## 목차

1. [아키텍처 개요](#1-아키텍처-개요)
2. [StagePvP 맵 엔티티 구조](#2-stagepvp-맵-엔티티-구조)
3. [Player 엔티티 컴포넌트 추가](#3-player-엔티티-컴포넌트-추가)
4. [PvP UI 그룹 4개 생성](#4-pvp-ui-그룹-4개-생성)
5. [프로퍼티 바인딩 체크리스트](#5-프로퍼티-바인딩-체크리스트)
6. [검증 방법](#6-검증-방법)

---

## 1. 아키텍처 개요

PvP 시스템의 스크립트는 세 가지 배치 레이어로 나뉩니다.

### 1-1. Logic 스크립트 (자동 등록 — 별도 엔티티 불필요)

| 스크립트 | 전역 접근자 | 역할 |
|---------|-----------|------|
| `pvpService` | `_pvpService` | PvP 전역 설정값 (스테이지 경로, 점수, 타이머, 타워 HP) |
| `pvpUserService` | `_pvpUserService` | 유저 데이터 CRUD (DataStorage) |
| `pvpRankingService` | `_pvpRankingService` | 랭킹 조회·점수 변동 (SortableDataStorage) |
| `eGameMode` | `_eGameMode` | 게임 모드 enum (PvP = 6) |

> Logic 스크립트(`@Logic`)는 스크립트 파일이 프로젝트에 존재하기만 하면 자동으로 등록됩니다.
> Studio에서 별도로 엔티티에 부착할 필요가 없습니다.

### 1-2. Map 엔티티 컴포넌트 (StagePvP 맵 내 "PvPManager" 엔티티)

| 컴포넌트 | 역할 |
|---------|------|
| `pvpCombatComponent` | 게임 시작/종료, 타이머, 승패 판정 |
| `pvpAiPlayerComponent` | 적 AI 자동 소환 |
| `pvpUserSpawnComponent` | 유저 유닛 소환 & 포인트 관리 |

> 이 3개 컴포넌트는 **같은 엔티티**(PvPManager)에 부착합니다.
> 코드에서 `self.Entity.pvpCombatComponent`, `self.Entity.pvpAiPlayerComponent` 등으로
> 형제 컴포넌트를 참조하기 때문입니다.

### 1-3. Player 엔티티 컴포넌트

| 컴포넌트 | 역할 |
|---------|------|
| `pvpUserComponent` | 플레이 코인, 공격/방어 덱 관리 |
| `pvpRankingComponent` | 랭킹 조회 & 내 순위/점수 |
| `pvpMatcherMakerComponent` | 매칭 풀 생성 & 대전 시작 |

> Player 엔티티(`_UserService.LocalPlayer`)에 부착합니다.

### 1-4. UI 컴포넌트 (4개 UI 그룹)

| UI 엔티티 경로 | 컴포넌트 스크립트 | 역할 |
|---------------|----------------|------|
| `/ui/PvPLobbyUI` | `PvPLobbyUI` | 매칭 로비 화면 |
| `/ui/PvPIngameUI` | `PvPIngameUI` | 인게임 HUD (HP바, 타이머, 소환 슬롯) |
| `/ui/PvPResultUI` | `PvPResultUI` | 승패 결과 화면 |
| `/ui/PvPRankingUI` | `PvPRankingUI` | 랭킹 표시 화면 |

---

## 2. StagePvP 맵 엔티티 구조

### 2-1. 기존 맵 엔티티 (이미 존재 — 변경 불필요)

StagePvP.map에 이미 포함된 엔티티들:

```
/maps/StagePVP                           (루트: MapComponent, FootholdComponent)
├── Background                            (배경)
├── Background Root                       (배경 오브젝트 그룹)
│   ├── foothold-2452
│   └── MapObject_1 ~ MapObject_13
├── MapleMapLayer
├── TileMap
├── EnemyTower                            ★ UUID: 3c23fbd3-219d-4f45-8f64-1a96e29a64c6
│   Components: TransformComponent, SpriteRendererComponent, TagComponent(Enemy,Tower),
│               MonsterSpawnService, Tower, HitComponent, Unit, GameClearChecker
├── ProjectileParent                      (빈 컨테이너)
├── MyMonsterList                         (빈 컨테이너)
│   └── MySpawnPoint                      (TransformComponent: x=-5.0, y=-2.4)
├── EnemyList                             (빈 컨테이너)
│   └── EnemySpawnPoint                   (TransformComponent: x=11.0, y=-1.975)
├── SkillParent                           (빈 컨테이너)
├── PlayerTower                           ★ UUID: add7715d-0f35-4a67-8da8-9b662dd3d920
│   Components: TransformComponent, SpriteRendererComponent, TagComponent(MyMonster,Tower),
│               HitComponent, Tower, Unit
├── ingameIncome                          (MesoIncome)
├── userWallet                            (MesoWallet)
└── ingameWalletUI                        (MesoUI)
```

### 2-2. 신규 생성: PvPManager 엔티티

StagePvP 맵 안에 새로운 엔티티를 하나 만들어야 합니다.

**생성 위치:** `/maps/StagePVP/PvPManager`

| 항목 | 값 |
|-----|---|
| 이름 | `PvPManager` |
| 경로 | `/maps/StagePVP/PvPManager` |
| Enable | `true` |
| Visible | `false` (비주얼 없음) |

**부착할 컴포넌트 3개:**

| 순서 | 컴포넌트 | 프로퍼티 설정 |
|-----|---------|------------|
| 1 | `pvpCombatComponent` | 아래 참조 |
| 2 | `pvpAiPlayerComponent` | 기본값 유지 |
| 3 | `pvpUserSpawnComponent` | 기본값 유지 |

**pvpCombatComponent 프로퍼티 설정:**

| 프로퍼티 | 타입 | 값 | 설명 |
|---------|-----|---|------|
| `UserTower` | Entity | `add7715d-0f35-4a67-8da8-9b662dd3d920` | PlayerTower 엔티티 UUID |
| `EnemyTower` | Entity | `3c23fbd3-219d-4f45-8f64-1a96e29a64c6` | EnemyTower 엔티티 UUID |
| `isPlaying` | boolean | `false` | 기본값 |
| `gameTimer` | number | `0` | 기본값 |
| `maxGameTime` | number | `120` | 기본값 |

> **중요:** `UserTower`와 `EnemyTower`의 UUID는 StagePvP.map에 이미 존재하는 엔티티입니다.
> Studio의 프로퍼티 편집기에서 해당 엔티티를 드래그하거나 UUID를 직접 입력합니다.

---

## 3. Player 엔티티 컴포넌트 추가

Player 엔티티는 `_UserService.LocalPlayer`가 가리키는 엔티티입니다.
기존에 `PlayerComponent`, `UserDeck`, `UserStage` 등이 부착되어 있는 엔티티에 추가합니다.

**추가할 컴포넌트 3개:**

| 컴포넌트 | 프로퍼티 | 기본값 | 비고 |
|---------|---------|-------|------|
| `pvpUserComponent` | `play_coin` | `5` | |
| | `offense_deck` | `{}` | |
| | `defense_deck` | `{}` | |
| | `next_play_coin` | `0` | |
| `pvpRankingComponent` | `ranking_list` | `{}` | |
| | `score` | `0` | |
| | `rank` | `0` | |
| `pvpMatcherMakerComponent` | `targetUserList` | `{}` | |
| | `selected` | `0` | @Sync |

> **참조 관계:** 코드에서 `user.pvpUserComponent`, `user.pvpRankingComponent`,
> `user.pvpMatcherMakerComponent`로 접근하므로 반드시 Player 엔티티에 부착해야 합니다.

---

## 4. PvP UI 그룹 4개 생성

MSW Studio에서 새 UI Group을 4개 만들어야 합니다.
각 UI Group은 `/ui/` 경로 하위에 생성합니다.

---

### 4-1. PvPLobbyUI (매칭 로비)

**경로:** `/ui/PvPLobbyUI`

코드에서 접근하는 경로:
- `_EntityService:GetEntityByPath("/ui/PvPLobbyUI")` — PvPResultUI.mlua:92

**루트 엔티티 설정:**

| 항목 | 값 |
|-----|---|
| 이름 | `PvPLobbyUI` |
| 컴포넌트 | `UITransformComponent`, `UIGroupComponent`, `PvPLobbyUI` (스크립트) |
| Enable | `false` (초기 비활성) |
| DefaultShow | `false` |

**자식 엔티티 계층:**

```
/ui/PvPLobbyUI
├── CoinText                ← property Entity coinText
│   Components: UITransformComponent, TextComponent
│   용도: 보유 코인 수 표시 (pvpUserComponent.play_coin)
│
├── ScoreText               ← property Entity scoreText
│   Components: UITransformComponent, TextComponent
│   용도: 티어 + 점수 표시 (예: "Gold (250)")
│
├── MatchButton             ← property Entity matchButton
│   Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
│   용도: 대전 시작 버튼
│
├── RefreshButton           ← property Entity refreshButton
│   Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
│   용도: 상대 목록 새로고침 버튼
│
├── Slot_1                  ← property table slotList[1]
│   Components: UITransformComponent, SpriteGUIRendererComponent
│   Children:
│   ├── [1] NameText        (TextComponent — 닉네임/UID 표시)
│   └── [2] ScoreText       (TextComponent — 점수 표시)
│
├── Slot_2                  ← property table slotList[2]
│   (Slot_1과 동일 구조)
│
├── Slot_3                  ← property table slotList[3]
│   (Slot_1과 동일 구조)
│
└── Slot_4                  ← property table slotList[4]
    (Slot_1과 동일 구조)
```

**프로퍼티 바인딩 (PvPLobbyUI 컴포넌트):**

| 프로퍼티 | 바인딩 대상 |
|---------|-----------|
| `coinText` | → CoinText 엔티티 |
| `scoreText` | → ScoreText 엔티티 |
| `matchButton` | → MatchButton 엔티티 |
| `refreshButton` | → RefreshButton 엔티티 |
| `slotList` | → 코드에서 초기화 (`slotList[1]`~`slotList[4]`에 Slot_1~Slot_4 바인딩) |

> **slotList 주의:** `slotList`는 `property table` 타입이므로 Studio에서 직접 바인딩할 수 없습니다.
> 대안 1: OnBeginPlay에서 `self.Entity.Children`을 순회하여 이름으로 찾기
> 대안 2: 개별 `property Entity slot1/slot2/slot3/slot4`로 변경 후 바인딩
> 현재 코드는 `slotList[i]`로 접근하므로, **OnBeginPlay에서 수동 초기화 코드를 추가**하거나
> **스크립트에 4개의 개별 property Entity를 추가**해야 합니다.

**Slot 자식 엔티티 접근 패턴:**

코드에서 각 Slot의 자식에 `Children[1]`, `Children[2]`로 접근합니다:
```lua
-- PvPLobbyUI.mlua:146-155
local nameText = slot.Children[1]     -- 첫 번째 자식 = 닉네임
nameText.TextComponent.Text = info.nickname or info.uid or "???"

local scoreText = slot.Children[2]     -- 두 번째 자식 = 점수
scoreText.TextComponent.Text = tostring(info.score or 0)
```

따라서 각 Slot 엔티티 안의 **자식 순서가 중요**합니다:
- Children[1] = NameText (TextComponent 필수)
- Children[2] = ScoreText (TextComponent 필수)

---

### 4-2. PvPIngameUI (인게임 HUD)

**경로:** `/ui/PvPIngameUI`

코드에서 접근하는 경로:
- `_EntityService:GetEntityByPath("/ui/PvPIngameUI")` — pvpCombatComponent.mlua:82, 166

**루트 엔티티 설정:**

| 항목 | 값 |
|-----|---|
| 이름 | `PvPIngameUI` |
| 컴포넌트 | `UITransformComponent`, `UIGroupComponent`, `PvPIngameUI` (스크립트) |
| Enable | `false` (초기 비활성, 게임 시작 시 활성화) |
| DefaultShow | `false` |

**자식 엔티티 계층:**

```
/ui/PvPIngameUI
├── UserHPBar               ← property Entity userHPBar
│   Components: UITransformComponent, SliderComponent, OurHealthBar (스크립트)
│   용도: 아군 타워 HP 바
│
├── EnemyHPBar              ← property Entity enemyHPBar
│   Components: UITransformComponent, SliderComponent, EnemyHealthBar (스크립트)
│   용도: 적 타워 HP 바
│
├── TimerText               ← property Entity timerText
│   Components: UITransformComponent, TextComponent
│   용도: 남은 시간 표시 (예: "1:45")
│
├── PointText               ← property Entity pointText
│   Components: UITransformComponent, TextComponent
│   용도: 현재 소환 포인트 표시
│
├── UnitSlot_1              ← property table unitSlots[1]
│   Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
│   Children:
│   ├── [1] NameText        (TextComponent — 유닛 이름)
│   └── [2] CostText        (TextComponent — 소환 코스트)
│
├── UnitSlot_2              ← property table unitSlots[2]
│   (UnitSlot_1과 동일 구조)
│
├── ... (최대 8개)
│
└── UnitSlot_8              ← property table unitSlots[8]
    (UnitSlot_1과 동일 구조)
```

**프로퍼티 바인딩 (PvPIngameUI 컴포넌트):**

| 프로퍼티 | 바인딩 대상 |
|---------|-----------|
| `userHPBar` | → UserHPBar 엔티티 |
| `enemyHPBar` | → EnemyHPBar 엔티티 |
| `timerText` | → TimerText 엔티티 |
| `pointText` | → PointText 엔티티 |
| `unitSlots` | → 코드에서 초기화 (아래 참고) |

> **unitSlots 주의:** `slotList`와 동일한 이슈 — `property table` 타입은 Studio에서 엔티티 바인딩 불가.
> OnBeginPlay 또는 InitUI에서 수동으로 테이블을 채우거나,
> 개별 `property Entity unitSlot1` ~ `unitSlot8`로 변경해야 합니다.

**HP 바 컴포넌트 설정:**

UserHPBar와 EnemyHPBar 각각에 기존 프로젝트의 HP 바 스크립트를 부착합니다:

- **UserHPBar**: `SliderComponent` + `OurHealthBar` 스크립트
  - OurHealthBar.tower 프로퍼티: 게임 시작 시 코드에서 자동 설정됨 (InitUI 호출)
  - SliderComponent.Value: 1 (초기 풀HP)

- **EnemyHPBar**: `SliderComponent` + `EnemyHealthBar` 스크립트
  - EnemyHealthBar.tower 프로퍼티: 게임 시작 시 코드에서 자동 설정됨
  - SliderComponent.Value: 1 (초기 풀HP)

**유닛 슬롯 자식 접근 패턴:**

```lua
-- PvPIngameUI.mlua:71-80
local nameText = slot.Children[1]     -- 첫 번째 자식 = 유닛 이름
nameText.TextComponent.Text = unitData["name"] or ""

local costText = slot.Children[2]     -- 두 번째 자식 = 코스트
costText.TextComponent.Text = tostring(unitData["cost"] or unitData["summonCost"] or 0)
```

각 UnitSlot 엔티티의 **자식 순서**:
- Children[1] = NameText (TextComponent 필수)
- Children[2] = CostText (TextComponent 필수)

---

### 4-3. PvPResultUI (대전 결과)

**경로:** `/ui/PvPResultUI`

코드에서 접근하는 경로:
- `_EntityService:GetEntityByPath("/ui/PvPResultUI")` — pvpCombatComponent.mlua:173

**루트 엔티티 설정:**

| 항목 | 값 |
|-----|---|
| 이름 | `PvPResultUI` |
| 컴포넌트 | `UITransformComponent`, `UIGroupComponent`, `PvPResultUI` (스크립트) |
| Enable | `false` (초기 비활성, 게임 종료 시 활성화) |
| DefaultShow | `false` |

**자식 엔티티 계층:**

```
/ui/PvPResultUI
├── ResultTitle             ← property Entity resultTitle
│   Components: UITransformComponent, TextComponent
│   용도: "VICTORY!" 또는 "DEFEAT" 텍스트
│
├── ScoreChangeText         ← property Entity scoreChangeText
│   Components: UITransformComponent, TextComponent
│   용도: 점수 변동 표시 (예: "+10" 또는 "-5")
│
├── CurrentScoreText        ← property Entity currentScoreText
│   Components: UITransformComponent, TextComponent
│   용도: 현재 총 점수 표시
│
├── TierText                ← property Entity tierText
│   Components: UITransformComponent, TextComponent
│   용도: 현재 티어 이름 (Bronze/Silver/Gold/Platinum/Diamond)
│
├── RetryButton             ← property Entity retryButton
│   Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
│   용도: "다시하기" 버튼 → PvPLobbyUI 열기
│
└── LobbyButton             ← property Entity lobbyButton
    Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
    용도: "로비로" 버튼 → 메인 로비로 텔레포트
```

**프로퍼티 바인딩 (PvPResultUI 컴포넌트):**

| 프로퍼티 | 바인딩 대상 |
|---------|-----------|
| `resultTitle` | → ResultTitle 엔티티 |
| `scoreChangeText` | → ScoreChangeText 엔티티 |
| `currentScoreText` | → CurrentScoreText 엔티티 |
| `tierText` | → TierText 엔티티 |
| `retryButton` | → RetryButton 엔티티 |
| `lobbyButton` | → LobbyButton 엔티티 |

---

### 4-4. PvPRankingUI (랭킹 화면)

**경로:** `/ui/PvPRankingUI`

코드에서 직접 `GetEntityByPath`로 접근하지 않으나, PvPLobbyUI 등에서 열 수 있도록 설정.

**루트 엔티티 설정:**

| 항목 | 값 |
|-----|---|
| 이름 | `PvPRankingUI` |
| 컴포넌트 | `UITransformComponent`, `UIGroupComponent`, `PvPRankingUI` (스크립트) |
| Enable | `false` (초기 비활성) |
| DefaultShow | `false` |

**자식 엔티티 계층:**

```
/ui/PvPRankingUI
├── RankListParent          ← property Entity rankListParent
│   Components: UITransformComponent
│   용도: 랭킹 항목의 부모 컨테이너
│   Children: (최소 10~20개의 RankItem 엔티티를 미리 생성)
│   ├── RankItem_1
│   │   Components: UITransformComponent, SpriteGUIRendererComponent
│   │   Children:
│   │   ├── [1] RankText    (TextComponent — 순위)
│   │   ├── [2] NameText    (TextComponent — 닉네임/UID)
│   │   ├── [3] ScoreText   (TextComponent — 점수)
│   │   └── [4] TierText    (TextComponent — 티어)
│   ├── RankItem_2
│   │   (RankItem_1과 동일 구조)
│   └── ... (필요한 만큼 반복)
│
├── MyRankText              ← property Entity myRankText
│   Components: UITransformComponent, TextComponent
│   용도: 내 순위 표시 (예: "#5" 또는 "-")
│
├── MyScoreText             ← property Entity myScoreText
│   Components: UITransformComponent, TextComponent
│   용도: 내 점수 표시
│
├── MyTierText              ← property Entity myTierText
│   Components: UITransformComponent, TextComponent
│   용도: 내 티어 이름
│
└── CloseButton             ← property Entity closeButton
    Components: UITransformComponent, SpriteGUIRendererComponent, ButtonComponent
    용도: 랭킹 창 닫기
```

**프로퍼티 바인딩 (PvPRankingUI 컴포넌트):**

| 프로퍼티 | 바인딩 대상 |
|---------|-----------|
| `rankListParent` | → RankListParent 엔티티 |
| `myRankText` | → MyRankText 엔티티 |
| `myScoreText` | → MyScoreText 엔티티 |
| `myTierText` | → MyTierText 엔티티 |
| `closeButton` | → CloseButton 엔티티 |

**RankItem 자식 접근 패턴:**

```lua
-- PvPRankingUI.mlua:81-109
for i, child in ipairs(self.rankListParent.Children) do
    local rankText  = child.Children[1]   -- 순위
    local nameText  = child.Children[2]   -- 닉네임
    local scoreText = child.Children[3]   -- 점수
    local tierText  = child.Children[4]   -- 티어
end
```

각 RankItem의 **자식 순서가 반드시 위 순서와 일치**해야 합니다.

---

## 5. 프로퍼티 바인딩 체크리스트

Studio에서 모든 설정이 끝난 후 아래 항목을 하나씩 확인합니다.

### 5-1. PvPManager 엔티티 (맵 내)

- [ ] `/maps/StagePVP/PvPManager` 엔티티 생성됨
- [ ] `pvpCombatComponent` 부착됨
- [ ] `pvpCombatComponent.UserTower` → PlayerTower UUID (`add7715d-...`)
- [ ] `pvpCombatComponent.EnemyTower` → EnemyTower UUID (`3c23fbd3-...`)
- [ ] `pvpAiPlayerComponent` 부착됨
- [ ] `pvpUserSpawnComponent` 부착됨

### 5-2. Player 엔티티

- [ ] `pvpUserComponent` 부착됨
- [ ] `pvpRankingComponent` 부착됨
- [ ] `pvpMatcherMakerComponent` 부착됨

### 5-3. PvPLobbyUI (/ui/PvPLobbyUI)

- [ ] UI Group 엔티티 생성됨 (경로: `/ui/PvPLobbyUI`)
- [ ] `PvPLobbyUI` 스크립트 컴포넌트 부착됨
- [ ] `coinText` → CoinText 자식 엔티티 (TextComponent 포함)
- [ ] `scoreText` → ScoreText 자식 엔티티 (TextComponent 포함)
- [ ] `matchButton` → MatchButton 자식 엔티티 (ButtonComponent 포함)
- [ ] `refreshButton` → RefreshButton 자식 엔티티 (ButtonComponent 포함)
- [ ] Slot_1~4 각각 Children[1]=NameText(TextComponent), Children[2]=ScoreText(TextComponent)

### 5-4. PvPIngameUI (/ui/PvPIngameUI)

- [ ] UI Group 엔티티 생성됨 (경로: `/ui/PvPIngameUI`)
- [ ] `PvPIngameUI` 스크립트 컴포넌트 부착됨
- [ ] `userHPBar` → UserHPBar 자식 엔티티 (SliderComponent + OurHealthBar 포함)
- [ ] `enemyHPBar` → EnemyHPBar 자식 엔티티 (SliderComponent + EnemyHealthBar 포함)
- [ ] `timerText` → TimerText 자식 엔티티 (TextComponent 포함)
- [ ] `pointText` → PointText 자식 엔티티 (TextComponent 포함)
- [ ] UnitSlot_1~8 각각 Children[1]=NameText(TextComponent), Children[2]=CostText(TextComponent)
- [ ] UnitSlot_1~8 각각 ButtonComponent 포함

### 5-5. PvPResultUI (/ui/PvPResultUI)

- [ ] UI Group 엔티티 생성됨 (경로: `/ui/PvPResultUI`)
- [ ] `PvPResultUI` 스크립트 컴포넌트 부착됨
- [ ] `resultTitle` → ResultTitle 자식 엔티티 (TextComponent 포함)
- [ ] `scoreChangeText` → ScoreChangeText 자식 엔티티 (TextComponent 포함)
- [ ] `currentScoreText` → CurrentScoreText 자식 엔티티 (TextComponent 포함)
- [ ] `tierText` → TierText 자식 엔티티 (TextComponent 포함)
- [ ] `retryButton` → RetryButton 자식 엔티티 (ButtonComponent 포함)
- [ ] `lobbyButton` → LobbyButton 자식 엔티티 (ButtonComponent 포함)

### 5-6. PvPRankingUI (/ui/PvPRankingUI)

- [ ] UI Group 엔티티 생성됨 (경로: `/ui/PvPRankingUI`)
- [ ] `PvPRankingUI` 스크립트 컴포넌트 부착됨
- [ ] `rankListParent` → RankListParent 자식 엔티티
- [ ] RankItem 자식들: Children[1]=순위, [2]=닉네임, [3]=점수, [4]=티어 (모두 TextComponent)
- [ ] `myRankText` → MyRankText 자식 엔티티 (TextComponent 포함)
- [ ] `myScoreText` → MyScoreText 자식 엔티티 (TextComponent 포함)
- [ ] `myTierText` → MyTierText 자식 엔티티 (TextComponent 포함)
- [ ] `closeButton` → CloseButton 자식 엔티티 (ButtonComponent 포함)

---

## 6. 검증 방법

### 6-1. 엔티티 경로 일치 확인

코드에서 `_EntityService:GetEntityByPath()`로 접근하는 모든 경로:

| 코드 위치 | 경로 | 확인 |
|----------|------|------|
| pvpCombatComponent.mlua:82 | `/ui/PvPIngameUI` | UI Group 존재? |
| pvpCombatComponent.mlua:166 | `/ui/PvPIngameUI` | UI Group 존재? |
| pvpCombatComponent.mlua:173 | `/ui/PvPResultUI` | UI Group 존재? |
| pvpCombatComponent.mlua:197 | `/maps/StagePVP` + `/MyMonsterList` | 맵 엔티티 존재? ✓ |
| pvpCombatComponent.mlua:212 | `/maps/StagePVP` + `/EnemyList` | 맵 엔티티 존재? ✓ |
| pvpAiPlayerComponent.mlua:133 | `/maps/StagePVP` + `/EnemyList` | 맵 엔티티 존재? ✓ |
| pvpAiPlayerComponent.mlua:134 | `/maps/StagePVP` + `/EnemyList/EnemySpawnPoint` | 맵 엔티티 존재? ✓ |
| pvpUserSpawnComponent.mlua:57 | `/maps/StagePVP` + `/MyMonsterList` | 맵 엔티티 존재? ✓ |
| pvpUserSpawnComponent.mlua:58 | `/maps/StagePVP` + `/MyMonsterList/MySpawnPoint` | 맵 엔티티 존재? ✓ |
| PvPResultUI.mlua:92 | `/ui/PvPLobbyUI` | UI Group 존재? |
| GameManager.mlua:249 | `/maps/StagePVP` (via GetGameMapName) | 맵 존재? ✓ |

### 6-2. property Entity 바인딩 확인

코드에서 `property Entity xxx = nil`로 선언된 프로퍼티가 Studio에서 바인딩되었는지 확인:

| 스크립트 | 프로퍼티 | Studio 바인딩 대상 |
|---------|---------|------------------|
| pvpCombatComponent | UserTower | PlayerTower UUID |
| pvpCombatComponent | EnemyTower | EnemyTower UUID |
| PvPLobbyUI | coinText | CoinText 엔티티 |
| PvPLobbyUI | scoreText | ScoreText 엔티티 |
| PvPLobbyUI | matchButton | MatchButton 엔티티 |
| PvPLobbyUI | refreshButton | RefreshButton 엔티티 |
| PvPIngameUI | userHPBar | UserHPBar 엔티티 |
| PvPIngameUI | enemyHPBar | EnemyHPBar 엔티티 |
| PvPIngameUI | timerText | TimerText 엔티티 |
| PvPIngameUI | pointText | PointText 엔티티 |
| PvPResultUI | resultTitle | ResultTitle 엔티티 |
| PvPResultUI | scoreChangeText | ScoreChangeText 엔티티 |
| PvPResultUI | currentScoreText | CurrentScoreText 엔티티 |
| PvPResultUI | tierText | TierText 엔티티 |
| PvPResultUI | retryButton | RetryButton 엔티티 |
| PvPResultUI | lobbyButton | LobbyButton 엔티티 |
| PvPRankingUI | rankListParent | RankListParent 엔티티 |
| PvPRankingUI | myRankText | MyRankText 엔티티 |
| PvPRankingUI | myScoreText | MyScoreText 엔티티 |
| PvPRankingUI | myTierText | MyTierText 엔티티 |
| PvPRankingUI | closeButton | CloseButton 엔티티 |

### 6-3. 알려진 제한사항 / TODO

1. **`property table` 바인딩 불가**: PvPLobbyUI.slotList, PvPIngameUI.unitSlots는
   `property table` 타입이므로 Studio에서 엔티티를 직접 바인딩할 수 없습니다.
   → OnBeginPlay에서 `self.Entity.Children`을 순회하여 이름으로 찾는 초기화 코드를 추가하거나,
   → 개별 `property Entity`로 분리해야 합니다.

2. **PvPIngameUI의 OnUpdate에서 `user.pvpCombatComponent` 접근**:
   PvPIngameUI.mlua:114에서 `user.pvpCombatComponent`에 접근하지만,
   pvpCombatComponent는 PvPManager(맵 엔티티)에 부착되어 있습니다.
   → 이 부분은 `_EntityService:GetEntityByPath(_pvpService.PVP_STAGE_PATH .. "/PvPManager")`
   등으로 수정이 필요할 수 있습니다.

3. **PvPIngameUI의 OnUnitSlotClick에서 `user.pvpUserSpawnComponent` 접근**:
   PvPIngameUI.mlua:101에서 `user.pvpUserSpawnComponent`에 접근하지만,
   pvpUserSpawnComponent는 PvPManager에 부착되어 있습니다.
   → 마찬가지로 PvPManager 엔티티를 통해 접근하도록 수정이 필요합니다.

---

## 부록: 티어 기준표

| 티어 | 최소 점수 | pvpRankingService 프로퍼티 |
|------|----------|--------------------------|
| Bronze | 0 | (기본) |
| Silver | 100 | `silver = 100` |
| Gold | 200 | `gold = 200` |
| Platinum | 300 | `platinum = 300` |
| Diamond | 400 | `dia = 400` |

## 부록: pvpService 설정값

| 프로퍼티 | 기본값 | 설명 |
|---------|-------|------|
| `PVP_MODE_VER` | `0` | 버전 (랭킹 키 구분) |
| `PVP_STAGE_PATH` | `"/maps/StagePVP"` | PvP 맵 경로 |
| `WIN_SCORE` | `10` | 승리 시 점수 |
| `LOSE_SCORE` | `-5` | 패배 시 점수 |
| `MAX_GAME_TIME` | `120` | 최대 게임 시간 (초) |
| `USER_TOWER_HP` | `500` | 아군 타워 HP |
| `ENEMY_TOWER_HP` | `500` | 적 타워 HP |
