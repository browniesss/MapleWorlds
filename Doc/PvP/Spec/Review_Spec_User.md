# PVP User Spec Review (Spec_User.md)

**Reviewer**: Antigravity (Senior Architect)  
**Date**: 2026-02-02  
**Status**: Review Completed / Awaiting Update

## 1. 종합 평가 (Executive Summary)
**"구조는 명확하나, 실제 구현 시 치명적인 버그를 유발할 수 있는 로직 오류와 누락이 존재합니다."**

*   **적절성 점수**: 70 / 100
*   **주요 문제**: 입장권 충전 수식 오류, 랭킹 동기화 누락, 데이터 중복 관리.
*   **권장 사항**: 아래 지적 사항을 반영하여 명세서를 수정(Refining)한 후 개발에 착수해야 합니다.

---

## 2. 상세 검토 (Technical Detail)

### 🚨 Critical Issues (즉시 수정 필요)

#### 1. 입장권 충전 로직 오류 (Ticket Recharge Logic)
*   **현재 명세**:
    > "만약 Count가 Max에서 줄어든 경우, LastRecharge를 현재 시간으로 설정하여 타이머 시작."
*   **문제점 (The Flaw)**:
    *   충전 중일 때(입장권이 소모된 상태) 또 플레이를 할 경우, `LastRecharge`를 현재 시간으로 갱신하면 **이미 흐른 충전 시간을 손해**보게 됩니다. (예: 29분 충전했는데 게임 한 판 했다고 타이머 리셋)
*   **개선 제안**:
    *   `CalculateRecharge()` 시 충전된 티켓 수 만큼 `LastRecharge`를 `Interval` 단위로 증가시켜야 합니다. (`LastRecharge += earned * 1800`)
    *   `Consume()` 시에는 **현재 Count가 Max였을 경우에만** `LastRecharge`를 `CurrentTime`으로 설정하고, 그 외 충전 중인 경우에는 기존 타이머를 유지해야 합니다.

#### 2. 글로벌 랭킹 동기화 누락 (Rank Synchronization)
*   **현재 명세**:
    *   `PVPUserManager`는 `UserDataStorage` (개인 데이터)만 갱신하는 것으로 기술됨.
*   **문제점 (The Flaw)**:
    *   `PVP_System.md`에 따르면 랭킹은 `SortableGlobalDataStorage`를 사용합니다. 명세서대로 구현하면 유저 개인 화면의 점수와 전체 랭킹보드의 데이터가 불일치하게 됩니다.
*   **개선 제안**:
    *   `UpdateScore()` 등의 메서드를 추가하고, 내부에서 `UserDataStorage`와 `SortableGlobalDataStorage`를 트랜잭션 수준으로 동시에 갱신하는 로직을 명시해야 합니다.

### ⚠️ Major Issues (잠재적 문제)

#### 1. 덱(Deck) 데이터의 불필요한 중복 (Redundancy)
*   **현재 명세**:
    *   `PVP_DeckSettings` 구조체 내에 `Level` 정보가 포함되어 있음.
*   **문제점**:
    *   실시간 성장을 반영할 것인지, 덱 저장 시점의 스냅샷을 사용할 것인지 불분명합니다. 통상적인 디펜스 게임에서는 실시간 성장을 반영하므로 레벨 정보를 덱에 저장하는 것은 데이터 정합성 문제를 야기합니다.
*   **개선 제안**:
    *   `Level` 필드를 삭제하고 `UnitId`만 저장합니다. 전투 진입 시 `Inventory`에서 최신 스펙을 조회하도록 명시하십시오.

#### 2. 실행 공간(ExecSpace) 명시 부재
*   **개선 제안**:
    *   `UserDataStorage` 접근 로직은 보안상 반드시 **Server Only**여야 합니다. 각 API 및 메서드에 `[Server]` 표기를 추가하여 혼선을 방지해야 합니다.

---

## 3. 결론 (Conclusion)
이 문서는 '개념 증명' 단계까지는 적합하나, 안정적인 서비스를 위한 예외 처리와 시스템 연동 로직이 부족합니다. **v1.1로의 레이아웃 및 로직 업데이트** 후에 코드를 작성하는 것을 강력히 추천합니다.
