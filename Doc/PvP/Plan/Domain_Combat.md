# [Domain] Combat 구현 계획

## 1. 개요
플레이어의 개입 없이 사전에 설정된 전략(덱)에 따라 자동으로 진행되는 전투 시스템입니다.

## 2. 주요 기능 (Features)

### 2.1 스쿼드 소환 (Squad Command)
*   **우선순위 소환**: 1~8번 슬롯에 설정된 유닛을 순차적으로 확인.
*   **자원 체크**: 현재 마나가 소환 비용보다 클 경우 소환.
*   **쿨타임 없음**: PVP 모드에서는 유닛 소환 쿨타임을 적용하지 않음 (무한 소환 가능).

### 2.2 전투 시뮬레이션
*   **Entity 생성**: `SpawnManager`를 통해 실제 월드에 유닛 생성.
*   **Stats 적용**: `BuffedUnitService`를 통해 레벨/한계돌파가 반영된 능력치 적용.

### 2.3 승패 판정 (Win/Loss)
*   **Main Win**: 적 본진(Castle) HP가 0이 되면 즉시 승리.
*   **Timeout (180s)**:
    1.  생존 유닛 수 많은 쪽 승리.
    2.  (동률 시) 본진 HP 비율 높은 쪽 승리.

## 3. 구현 파일
*   `Scripts/GameLogic/PVP/Combat/PVPCombatLogic.mlua`
