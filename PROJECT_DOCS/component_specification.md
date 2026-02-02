# Component Specification: Unit Synergy

본 문서는 `MapleWorlds-Defense` 프로젝트의 시너지 시스템에 의해 유닛에 동적으로 부착되거나 참조되는 **개별 스크립트 컴포넌트**의 상세 로직을 명세합니다.

---

## 1. 기반 클래스 (Base Class)

### 1.1 `BaseSynergyComponent`
모든 시너지 관련 컴포넌트의 부모 클래스로, 공통 데이터 구조와 수치 계산 메서드를 제공합니다.

- **Properties**:
  - `@Sync string type`: 시너지 수치 적용 방식 (`"절대"` 또는 `"비율"`)
  - `@Sync number value`: 실제 적용될 수치 값 (비율일 경우 100,000 기준)
  - `string target`: 시너지 타겟 식별자

- **Core Methods**:
  - `number CalValue(number baseValue)`: 
    - `"절대"`: `value` 자체를 반환.
    - `"비율"`: `value / 100000`을 반환 (계수로서 작동).
  - `number CalValueInClient(number baseValue)`:
    - `"비율"`: `value / 100000 + 1`을 반환 (배율로서 작동).
  - `SynergyOnHitResult OnHit(...)`: 피격/타격 시의 결과 객체를 생성하는 기본 가상 메서드.

---

## 2. 재생 관련 컴포넌트 (Regen Components)

### 2.1 `IncreaseHpByTime`
유닛의 체력을 주기적으로 회복시키는 컴포넌트입니다.

- **Heritage**: `BaseSynergyComponent`
- **Logic**:
  - 1초 간격(`ts >= 1`)으로 실행.
  - `CalValue(target.maxHP)`를 통해 계산된 값만큼 현재 `hp` 증가.
  - 최대 체력(`maxHP`)을 초과할 수 없음.
- **Execution Space**: `ClientOnly` (OnUpdate)

### 2.2 `IncreaseMpByTime`
유닛의 마나를 주기적으로 회복시키는 컴포넌트입니다.

- **Heritage**: `BaseSynergyComponent`
- **Logic**:
  - 1초 간격(`ts >= 1`)으로 실행.
  - `CalValue(target.maxMP)`를 통해 계산된 값만큼 현재 `mp` 증가.
  - 최대 마나(`maxMP`)를 초과할 수 없음.
- **Execution Space**: `ClientOnly` (OnUpdate)

---

## 3. 전투 관련 컴포넌트 (Combat Components)

### 3.1 `CriticalAttack`
공격 시 치명타 발동 및 데미지 증폭을 담당합니다.

- **Heritage**: `BaseSynergyComponent`
- **Logic**:
  - `OnHit` 이벤트 발생 시 호출.
  - `math.random`을 사용하여 `CalValue`로 계산된 확률(`baseCritial`) 내에 있는지 검사.
  - 치명타 성공 시: `isCritical = true`, `damage *= 1.5` (데미지 1.5배 증폭).
- **Execution Space**: 호출 방향에 따름 (Common logic)

---

## 4. 경제/자원 관련 컴포넌트 (Economy Components)

### 4.1 `AddSummonPointAfterKill`
적 처치 시 보너스 재화를 획득하는 로직입니다.

- **Heritage**: `BaseSynergyComponent`
- **Logic**:
  - 유닛이 타격(`OnHit`) 시 `AddMoney` 호출. (실제로는 처치 판정 시점이 중요하나 현재 코드는 타격 시마다 자금 추가 로직이 포함됨).
  - `userWallet`의 `MesoWallet:AddMeso`를 호출하여 `CalValue`만큼 재화 추가.
- **Execution Space**: `Server` (AddMoney)

### 4.2 자원 관리 삼인방
아래 컴포넌트들은 특정 기능에서 요청하는 수치 계산을 대행합니다.

| 컴포넌트 명 | 주요 메서드 | 역할 |
| :--- | :--- | :--- |
| `AddSummonPointAfterStart` | `GetInitPoint` | 게임 시작 시 초기 포인트에 시너지 효과 반영 |
| `DecreaseSummonCost` | `GetSummonCost` | 유닛 소환 비용 감소 수치 계산 |
| `IncreaseSummonPoint` | `AddMeso` | 주기적 또는 이벤트성 수입(Income) 증가 수치 계산 |
