# [Domain] Ranking 구현 계획

## 1. 개요
유저들의 경쟁 심리를 자극하고 공정한 매칭을 제공하는 시스템입니다.

## 2. 주요 기능 (Features)

### 2.1 하이브리드 티어 시스템
*   **Bronze ~ Diamond**: 절대 평가 (고정 점수 도달 시 승급).
*   **Red**: 상대 평가 (Diamond 이상 중 상위 10%).
*   **점수 로직**:
    *   기본 승/패 점수 + 점수차 보정.
    *   연승 보너스 및 연패 보호 적용.

### 2.2 시즌 관리 (Season)
*   **주기**: 1주일 단위 시즌제.
*   **초기화**: 시즌 종료 시 모든 점수 리셋 및 랭킹 보상 지급.

### 2.3 매칭 시스템 (Matchmaking)
*   **범위 검색**: 내 점수 기준 $\pm N$ 점수 내의 상대를 `SortableGlobalDataStorage`에서 검색.
*   **더미(Dummy) 생성**: 검색된 상대가 부족할(4명 미만) 경우, 티어별 스탯을 가진 더미 에이전트 생성.

## 3. 데이터 구조
*   `PVP_Rank_Master`: UserID(Key) - Score(Value) 매핑.

## 4. 구현 파일
*   `Scripts/GameLogic/PVP/Ranking/RankingService.mlua`
*   `Scripts/GameLogic/PVP/Ranking/ScoreLogic.mlua`
