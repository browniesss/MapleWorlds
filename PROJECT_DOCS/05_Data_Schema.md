# PROJECT_DOCS: 05. 데이터 명세 및 상수 (Data & Constants)

본 문서는 `MapleWorlds-Defense` 프로젝트에서 사용하는 주요 데이터 구조, 열거형(Enum), 그리고 상수 값들에 대해 정의합니다.

---

## 1. 정적 데이터셋 (DataSets)

프로젝트는 Excel CSV 파일을 변환한 `UserDataSet` 형식을 사용하여 기획 밸런싱 데이터를 관리합니다.

### 1.1 유닛 데이터 (`Unit.csv`)
유닛의 기본 능력치와 외형 정보를 담고 있습니다.
| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| **id** | String | 유닛 고유 ID |
| **hp** | Number | 기본 최대 체력 |
| **damage** | Number | 기본 공격력 |
| **attackRange** | Number | 사거리 (1.0 = 약 1미터) |
| **isMelee** | Integer | 1: 근거리, 0: 원거리 |
| **jobId** | Integer | 시너지 계산을 위한 직업군 ID |

### 1.2 소환 설정 (`GachaSetting.csv`)
가챠 뽑기 확률 및 픽업 설정을 관리합니다.
- `summon_1_value`: 단차 비용
- `summon_10_value`: 10연차 비용
- `pickup_char_limit`: 천장 횟수
- `pickup_char_id`: 픽업 시 지급할 유닛 ID

---

## 2. 주요 열거형 및 상수 (Enums & Constants)

스크립트 전반에서 하드코딩을 방지하기 위해 사용하는 논리값 그룹입니다.

### 2.1 전투 상태 이상 (`eStatusDebuffType`)
| 상수명 | 값 | 설명 |
| :--- | :--- | :--- |
| **Silence** | 1 | 액티브 스킬 사용 불가 상태 |
| **AttackSpeedDecrease** | 2 | 공격 속도 감속 |
| **HealDecrease** | 4 | 치유 효과 감소 |
| **Burn** | 8 | 지속적인 화염 피해 |

### 2.2 버프 타입 (`eBuffType`)
- `AttackSpeedBuff (1)`: 공격 속도 증가
- `MaxHealthMultiplyBuff (2)`: 체력 % 증폭
- `AttackDamageBuff (3)`: 공격력 증가
- `MoveSpeedBuff (4)`: 이동 속도 증가

---

## 3. 데이터 모델 (Struct Models)

데이터 저장 및 전송을 위한 고정 클래스 구조들입니다.

### 3.1 `BuffedUnitModel`
기본 유닛 스탯에 성장 수치(레벨, 한계돌파) 및 시너지 버프가 합산된 최종 연산용 모델입니다.
- 연산 공식: `BaseStat * (1 + SynergyRate) + FlatBonus`

### 3.2 `UserUnitModel`
유저가 보유한 개별 유닛의 인스턴스 정보입니다.
- `level`, `exp`, `limitBreak` 상태 포함.

---

## 4. 시스템 상수 (System Constants)

`GameManager` 및 주요 스비스에서 사용하는 고정 설정값입니다.

- **HardModeMultiplier**: 1.5 (하드 모드 스테이지 클리어 보상 배율)
- **MaxDeckCount**: 5 (전투에 데려갈 수 있는 최대 유닛 수)
- **BaseIncomeInterval**: 2.0 (인게임 메소 자동 지급 주기)
