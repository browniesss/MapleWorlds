# PVP 시스템 도메인별 구현 마스터 플랜 (Master Implementation Plan)

> [!NOTE]
> **Living Document**: 본 문서는 개발 진행 상황에 따라 지속적으로 수정 및 보완될 수 있습니다.

## 1. 개요 (Overview)
본 문서는 `Doc/PvP/PVP_System.md` 기술 사양서를 기반으로, PVP 시스템을 **Ranking, Combat, User, Shop, Event**의 5대 핵심 도메인으로 분리하여 구현하는 로드맵을 정의합니다.

## 2. 도메인 정의 (Domain Definitions)

### [Domain 1] Ranking (경쟁 및 매칭)
*   **책임**: 유저 간의 랭킹 산정, 티어 관리, 시즌 운영, 그리고 적절한 대전 상대 매칭.
*   **상세 계획**: [Domain_Ranking.md](./Domain_Ranking.md)

### [Domain 2] Combat (전투 및 판정)
*   **책임**: 스쿼드 기반의 유닛 소환(우선순위), 전투 진행, 승패 판정(성 파괴/타임아웃).
*   **상세 계획**: [Domain_Combat.md](./Domain_Combat.md)

### [Domain 3] User (유저 정보)
*   **책임**: 공격/방어 덱 설정, 개인 승률/로그 기록, 입장권 관리.
*   **상세 계획**: [Domain_User.md](./Domain_User.md)

### [Domain 4] Shop (보상 및 교환)
*   **책임**: PVP 전용 재화(투신의 증표) 관리 및 아이템 교환.
*   **상세 계획**: [Domain_Shop.md](./Domain_Shop.md)

### [Domain 5] Event (시즌 및 버전)
*   **책임**: 시즌 생명주기 관리, 데이터 버전 통제, 시즌별 설정 로드.
*   **상세 계획**: [Domain_Event.md](./Domain_Event.md)

## 3. 개발 원칙 (Principles)
*   **도메인 간 느슨한 결합**: 각 도메인은 명확한 인터페이스(Service/Function)를 통해서만 상호작용합니다.
*   **기능 중심 구현**: 특정 레이어(Data/Logic)를 한 번에 구현하지 않고, 각 도메인 내에서 수직적으로 기능(Data->Logic->Service)을 완성합니다.

## 4. 핵심 아키텍처 결정사항 (Key Architectural Decisions)
*   **저장소 전략 (Storage Strategy)**:
    *   **랭킹**: `SortableGlobalDataStorage` (실시간 정렬 및 범위 검색 Key: `PVP_Rank_Master`)
    *   **개인 데이터**: `UserDataStorage` (덱 구성, 승률, 방어 로그)
*   **더미 에이전트 (Dummy Agents)**:
    *   매칭 풀 부족(4명 미만) 시, 티어별 사전 정의된 더미 데이터를 생성하여 매칭을 성사시킵니다.

## 5. UI 통합 전략 (UI Integration Strategy)
각 도메인의 기능을 사용자에게 시각화하는 통합 계획입니다.
*   **Lobby UI**:
    *   [Ranking] 내 티어, 점수, 순위 표시.
    *   [User] 덱 편집 진입, 입장권 수량 표시.
    *   [Combat] 매칭 시작 버튼.
*   **Ingame UI**:
    *   [Combat] 아군/적군 소환 현황, 성 체력, 남은 시간.
    *   [Ranking] 전투 결과에 따른 점수 변동 연출.
*   **Shop UI**:
    *   [Shop] 상품 목록 및 구매 팝업.
    *   [User] 재화 보유량 표시.

## 6. 검증 계획 (Verification Plan)
각 도메인별 핵심 검증 항목입니다.
1.  **Ranking**: 점수 변동에 따른 랭킹 갱신 및 $\pm N$ 범위 매칭(더미 포함) 정확성.
2.  **Combat**: 우선순위 기반 소환 시퀀스 및 승패 판정(성 파괴/타임아웃) 로직.
3.  **User**: 덱 저장/로드의 데이터 무결성 및 입장권 충전/소모 사이클.
4.  **Shop**: 투신의 증표 획득/차감 및 아이템 지급 트랜잭션.
