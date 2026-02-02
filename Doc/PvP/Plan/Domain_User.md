# [Domain] User 구현 계획

## 1. 개요
PVP 컨텐츠를 즐기기 위한 개별 유저의 설정 및 기록을 관리합니다. **이벤트 도메인의 시즌/버전 정보**에 따라 데이터의 유효성을 검증하고 관리합니다.

## 2. 주요 기능 (Features)

### 2.1 데이터 버전 관리 (Data Versioning)
*   **Key-Based Versioning**: 데이터 내부에 버전을 저장하지 않고, **Storage Key 자체에 버전을 포함**하여 관리 (예: `PVP_Stats_v2024S01`).
*   **신규 데이터 생성**:
    *   해당 버전의 키(`Key_v{Version}`)로 데이터 조회 시 결과가 없다면, **새로운 시즌으로 간주하고 신규 데이터를 생성**하여 반환.
    *   별도의 마이그레이션 절차 없이 키 변경만으로 시즌 데이터 분리.

### 2.2 덱 관리 (Deck Management)
*   **공격 덱 (Offense)**: 침공 시 사용할 8개 유닛 구성.
*   **방어 덱 (Defense)**: 침공 받을 시 사용할 8개 유닛 구성.
*   **영속성**: `UserDataStorage`에 JSON 형태로 저장.

### 2.3 프로필 및 통계
*   **PVP Stats**: 총 승리, 패배, 현재 연승, 최고 달성 티어 기록 (시즌별 초기화 대상).
*   **Defense Log**: 오프라인 상태에서 발생한 방어전 기록.

### 2.4 입장권 (Entry Ticket)
*   **충전**: 일정 시간(예: 30분)마다 1장씩 자동 충전.
*   **소모**: 매칭 확정 및 전투 진입 시 1장 소모.

## 3. 데이터 구조
*   `UserDataStorage`:
    *   `PVP_Stats_v{Version}`: 버전별 승패, 점수, 티어 정보 (시즌별 격리).
    *   `PVP_Deck`: 버전과 무관하게 유지되는 공용 덱 설정.

## 4. 구현 파일
*   `Scripts/GameLogic/PVP/User/PVPUserManager.mlua`
