# PVP User Spec 2차 기술 검토 보고서 (Re-review)

**Reviewer**: Antigravity (Senior Architect)  
**Date**: 2026-02-02  
**Target Document**: `Doc/PvP/Spec/Spec_User.md` (v1.1)

## 1. 개요 (Overview)
본 2차 리뷰는 메이플스토리 월드(MSW) 플랫폼의 **DataStorage 동작 방식과 멀티플레이어 환경에서의 데이터 원자성(Atomicity)**에 집중하여 수행되었습니다. 현재 명세서대로 구현할 경우, 단순한 로직 오류를 넘어 서비스 운영 중 **데이터 유실이나 정합성 파괴**가 발생할 위험이 높습니다.

---

## 2. 핵심 기술적 리스크 (Critical Technical Risks)

### 🚨 R1. 오프라인 유저 데이터 수정 시 Race Condition (치명적)
*   **현재 명세**: `UpdateScore` 및 `DefenseLog` 기록 시 공격자가 방어자의 `UserDataStorage`에 직접 `Set`을 수행함.
*   **리스크 분석**:
    *   MSW의 `UserDataStorage`는 유저가 온라인일 때 메모리 캐시를 우선시합니다. 
    *   방어자가 다른 채널/인스턴스에서 접속 중일 때, 공격자가 해당 유저의 `UserDataStorage`를 수정하더라도 **방어자가 로그아웃하는 순간 방어자의 로컬 데이터가 서버 데이터를 덮어쓰게(Overwrite) 됩니다.**
    *   이로 인해 **PVP 승패 기록과 Defense Log가 증발**하는 버그가 발생할 수 있습니다.
*   **권장 해결안**:
    *   `DefenseLog`와 같이 타인이 수정해야 하는 데이터는 `UserDataStorage` 대신 별도의 `GlobalDataStorage` (Key: `Log_{UserId}`)를 사용하여 유저 프로필과 생명주기를 분리하십시오.
    *   점수(Score)는 반드시 `SortableGlobalDataStorage`의 `UpdateAndWait` (또는 원자적 연산)을 통해 관리하고 개인 프로필은 참조만 해야 합니다.

### 🚨 R2. 시즌 전환(Transition) 시점의 정합성 (치명적)
*   **현재 명세**: API 호출마다 `_PVP_SeasonManager:GetCurrentVersion()`을 참조함.
*   **리스크 분석**:
    *   전투 종료 시점에 시즌이 정각에 바뀌었을 경우, `ConsumeTicket`은 시즌 A에서 발생하고 `UpdateScore`는 시즌 B에서 발생하여 **보상은 받지 못하고 티켓만 날리는** 현상이 발생할 수 있습니다.
*   **권장 해결안**:
    *   전투 진입(`Match Start`) 시점에 사용된 **`Version` 정보를 세션 데이터로 고정(Locking)**하고, 결과 저장 시 해당 고정된 버전을 키로 사용해야 합니다.

### ⚠️ R3. 데이터 직렬화 및 크기 제한 (주요)
*   **현재 명세**: `DefenseLog`를 리스트 형태로 성적표 구조체 안에 포함함.
*   **리스크 분석**:
    *   `UserDataStorage`는 문자열만 저장 가능하므로 매번 `TableToString` / `StringToTable` 변환이 발생합니다.
    *   로그가 20개 쌓이고 각 로그가 상세 정보(상대 덱 등)를 가질 경우, 데이터 크기가 커져 `SetAndWait` 지연 시간이 증가하고 네트워크 부하가 발생합니다.
*   **권장 해결안**:
    *   성적표(`Stats`)와 기록(`Log`)을 별도 키로 분리하여 필요한 경우에만 로그를 로드(On-demand Loading)하도록 설계하십시오.

---

## 3. 구현 단계의 체크리스트 (Implementation Checklist)
1.  [ ] **Exception Handling**: `DataStorage` 호출 실패(`ErrorCode ~= 0`) 시 유저에게 재시도를 유도하거나 대기 로직이 있는가?
2.  [ ] **Atomic Operations**: 점수 갱신 시 `Get-Modify-Set` 사이의 갭을 어떻게 메울 것인가? (MSW `SortableDataStorage` 활용 권장)
3.  [ ] **Serialization**: `_UtilLogic:TableToString()` 사용 시 순환 참조나 지원되지 않는 타입(Function 등)이 포함되지 않았는가?

## 4. 최종 의견
현 명세서는 '기능의 나열'에는 성공했으나 '안정적인 데이터 유지' 측면에서 MSW 특성을 충분히 반영하지 못했습니다. 특히 **오프라인 유저 데이터 수정 이슈**는 서비스 중 장애로 이어질 가능성이 매우 높으므로, 데이터 저장소 전략(Storage Strategy)을 다시 수립할 것을 강력히 권고합니다.
