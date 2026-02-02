# [Domain] Event 구현 계획

## 1. 개요
PVP 시스템의 생명주기(Lifecycle)인 '시즌(Season)'과 데이터 정합성을 위한 '버전(Version)'을 통합 관리하는 컨트롤 타워입니다.

## 2. 주요 기능 (Features)

### 2.1 시즌 제어 (Season Control)
*   **시즌 ID 관리**: `Season_ID`(예: 2024_S01)를 통해 현재 활성화된 시즌을 식별.
*   **생명주기 관리**:
    *   **Season Start**: 시즌 시작 시각 및 설정값 로드.
    *   **Season Emd**: 시즌 종료 시각 도달 시 `Ranking` 및 `User` 도메인에 종료 이벤트 전파.

### 2.2 버전 관리 (Version Control)
*   **데이터 정합성**: 서버와 클라이언트의 `Version` 값이 일치해야만 PVP 진입 허용.
*   **데이터 갱신 트리거**: 시즌 변경 등으로 버전이 올라갈 때, 타 도메인(User, Ranking)에 **새 버전에 맞는 신규 데이터를 생성**하도록 지시. (기존 데이터 보존, 신규 데이터셋 사용)

### 2.3 이벤트 데이터 (Event Data)
*   **설정 로드**: `DataSets`을 통해 시즌별 보상 테이블, 일정, 규칙 등을 관리 및 로드.

## 3. 데이터 구조
*   **Storage**: `DataSets` (데이터셋 이름: `PVP_Season_Info` 등)
    *   Columns: `SeasonID`, `Version`, `StartTime`, `EndTime`, `ConfigJSON`

## 4. 구현 파일
*   `Scripts/GameLogic/PVP/Event/PVPEventService.mlua`
