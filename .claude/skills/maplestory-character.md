# MapleStory Worlds Character/Unit Creator

캐릭터(유닛)를 자동 생성하는 스킬입니다. 최소한의 사양만 입력하면 모델, 스킬 스크립트, CSV 데이터를 자동으로 생성합니다.

## When to activate
Activate when the user's message contains any of:
- "캐릭터 만들어줘", "유닛 만들어줘", "소환수 만들어줘"
- "캐릭터 추가", "유닛 추가", "몬스터 만들어줘", "몬스터 추가"
- "Create character", "Create unit", "Create monster"

## 캐릭터 타입

### 1. Pickup Character (`pickup`)
- `isUsableCharacter=1`, 아바타 기반, 고등급 플레이어블 캐릭터
- 유닛 모델 경로: `RootDesk/MyDesk/Prefab/UnitPrefab/UsableCharacter/Pickup_Character/Unit_{Name}.model`
- 스킬 모델 경로: `RootDesk/MyDesk/Prefab/SkillPrefab/UnitSkill/UsableSkill/NormalSkills/Pick_Up_Character/{SkillName}.model`
- 스킬 스크립트 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/UsableSkill/NormalSkills/Pick_Up_Character/{SkillName}.mlua`
- 코드블록 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/UsableSkill/NormalSkills/Pick_Up_Character/{SkillName}.codeblock`

### 2. Normal Character (`normal`)
- `isUsableCharacter=0`, 아바타 기반, 소환 유닛
- 유닛 모델 경로: `RootDesk/MyDesk/Prefab/UnitPrefab/UsableCharacter/Normal_Character/Unit_{Name}.model`
- 스킬 모델 경로: `RootDesk/MyDesk/Prefab/SkillPrefab/UnitSkill/UsableSkill/NormalSkills/Normal_Character/{SkillName}.model`
- 스킬 스크립트 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/UsableSkill/NormalSkills/Normal_Character/{SkillName}.mlua`
- 코드블록 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/UsableSkill/NormalSkills/Normal_Character/{SkillName}.codeblock`

### 3. Monster (`monster`)
- 스프라이트 기반, 적 몬스터. **모델은 이미 존재** (엔진 프리셋으로 생성됨)
- entity ID를 받아서 기존 모델을 **수정**
- 스킬 모델 경로: `RootDesk/MyDesk/Prefab/SkillPrefab/UnitSkill/NormalUnitSkill/{폴더경로}/{SkillName}.model`
- 스킬 스크립트 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/NormalUnitSkill/{폴더경로}/{SkillName}.mlua`
- 코드블록 경로: `RootDesk/MyDesk/Scripts/Skills/UnitSkill/NormalUnitSkill/{폴더경로}/{SkillName}.codeblock`

## 사용자 입력

### 공통 입력
| 입력 | 필수 | 기본값 | 설명 |
|------|------|--------|------|
| 영문 이름 | YES | - | 파일명용 (e.g. `Dark_Knight`) |
| 한글 이름 | YES | - | CSV 표시명 (e.g. `다크나이트`) |
| 타입 | YES | - | `pickup` / `normal` / `monster` |
| 근/원거리 | YES | `melee` | `melee` / `ranged` |
| 등급 (3~5) | NO | `3` | 유닛 등급 |

### Monster 추가 입력
| 입력 | 필수 | 설명 |
|------|------|------|
| entity ID | YES | 기존 모델의 UUID (e.g. `34cf0b95-51d4-4958-baa4-2a1a89178ce3`) |
| 폴더명 | YES | 지역 폴더명만 (e.g. `Perion`) → 상위 폴더 자동 매칭 |

### 스킬 입력
스킬 이름을 물어볼 때 **반드시 아래 상세 사양도 함께 보여주고** 변경 여부를 확인합니다.

| 입력 | 필수 | 기본값 | 설명 |
|------|------|--------|------|
| 스킬 영문 이름 | YES | `{Name}_Attack` | 스킬 파일명/스크립트명 (e.g. `Octopus_Ink`) |
| 스킬 한글 이름 | YES | `{DisplayName} 공격` | Skill.csv 표시명 (e.g. `옥토퍼스 먹물`) |
| desc | NO | `더미 스킬입니다.` | 스킬 설명 |
| skillArea | NO | `1` | 스킬 범위 |
| skillAmount | NO | `1` | 데미지 배율 |
| skillProjectileSpeed | NO | `0.15` | 투사체 속도 |
| skillTime | NO | `1` | 지속시간 |
| skillAttackTiming | NO | `1` | 데미지 적용 타이밍 |
| coolTime | NO | `1` | 쿨타임 |

**스킬 이름과 사양을 묻는 방법:**
스킬 이름을 물어보면서, 아래와 같이 더미 기본값 테이블을 함께 보여줍니다:
```
스킬 정보를 입력해주세요:
- 스킬 영문 이름 (기본: {Name}_Attack):
- 스킬 한글 이름 (기본: {DisplayName} 공격):

아래는 더미 기본값입니다. 변경할 항목이 있으면 알려주세요:
  desc: 더미 스킬입니다.
  skillArea: 1
  skillAmount: 1
  skillProjectileSpeed: 0.15
  skillTime: 1
  skillAttackTiming: 1
  coolTime: 1
```
사용자가 특정 값을 변경 요청하면 해당 값만 반영하고, 나머지는 기본값을 유지합니다.

## Instructions

### Step 0: 사용자 입력 파싱
사용자 메시지에서 필수 정보를 추출합니다. 부족한 정보는 질문하여 확인합니다.
- `{Name}`: 영문 이름 (파일명용, 언더스코어 구분)
- `{DisplayName}`: 한글 이름
- `{Type}`: pickup / normal / monster
- `{Combat}`: melee / ranged
- `{Grade}`: 3~5 (기본 3)
- Monster인 경우: `{EntityID}`, `{FolderName}`
- `{SkillName}`: 스킬 영문 이름 (기본: `{Name}_Attack`)
- `{SkillDisplayName}`: 스킬 한글 이름 (기본: `{DisplayName} 공격`)

**스킬 이름은 반드시 사용자에게 확인합니다.** 유닛에는 무조건 스킬이 하나 붙어야 하므로 스킬 생성은 필수입니다.

### Step 1: UUID 생성
Bash로 Python을 사용하여 UUID를 생성합니다.

**pickup/normal의 경우 (3개):**
```bash
python -c "import uuid; print(str(uuid.uuid4())); print(str(uuid.uuid4())); print(str(uuid.uuid4()))"
```
- UUID_1: 유닛 모델 ID
- UUID_2: 스킬 모델 ID
- UUID_3: 코드블록 ID

**monster의 경우 (2개):**
```bash
python -c "import uuid; print(str(uuid.uuid4())); print(str(uuid.uuid4()))"
```
- UUID_1: 스킬 모델 ID
- UUID_2: 코드블록 ID

### Step 2: (pickup/normal만) 유닛 모델 생성

**pickup 타입** — `Unit_Hero.model` 기반으로 Unit_{Name}.model 생성:
경로: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\UnitPrefab\UsableCharacter\Pickup_Character\Unit_{Name}.model`

**normal 타입** — `Unit_Arrow_Blow.model` 기반으로 Unit_{Name}.model 생성:
경로: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\UnitPrefab\UsableCharacter\Normal_Character\Unit_{Name}.model`

기존 모델 파일을 Read로 읽고, UUID와 Name만 교체하여 Write로 생성합니다.
- `EntryKey`의 UUID → UUID_1
- `ContentProto.Json.Id` → UUID_1
- `ContentProto.Json.Name` → `Unit_{Name}`

### Step 2-M: (monster만) 기존 모델 찾기, 이동, 수정

1. entity ID로 기존 모델 파일 찾기:
```bash
grep -r "{EntityID}" D:/MapleWorlds/RootDesk/MyDesk/Prefab/UnitPrefab/ --include="*.model" -l
```

2. **모델 파일 경로 확인 및 이동**: 찾은 모델 파일의 현재 경로와 목표 폴더를 비교합니다.
   - 목표 경로: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\UnitPrefab\{매칭된폴더경로}\Unit_{Name}.model`
   - 현재 경로가 목표 폴더와 다르면 파일을 이동합니다:
   ```bash
   # 목표 폴더가 없으면 생성 (maplestory-folder 스킬 패턴에 따라 .directory도 생성)
   mkdir -p "D:/MapleWorlds/RootDesk/MyDesk/Prefab/UnitPrefab/{매칭된폴더경로}"
   # 모델 파일 이동
   mv "현재경로/원본파일명.model" "D:/MapleWorlds/RootDesk/MyDesk/Prefab/UnitPrefab/{매칭된폴더경로}/Unit_{Name}.model"
   ```
   - 현재 경로가 이미 올바른 폴더에 있으면 이동하지 않고, 파일명만 `Unit_{Name}.model`로 필요시 변경합니다.
   - **주의**: 이동 전에 목표 경로에 이미 동일 파일이 있는지 확인하세요.

3. 모델 파일을 Read로 읽기

4. JSON을 파싱하여 다음 수정 적용:

**제거:**
- Components 배열에서 `MOD.Core.AIWanderComponent` 제거
- Values 배열에서 TargetType이 `MOD.Core.AIWanderComponent`인 항목 모두 제거

**추가 (없는 경우만):**
Components 배열에 아래 항목 추가 + Values 배열에 Enable 엔트리 추가:

a) `MOD.Core.DamageSkinSettingComponent` (없으면 추가)
b) `MOD.Core.TriggerComponent` (없으면 추가) — HitComponent의 BoxSize/ColliderOffset 값 복사 + IsLegacy=false
c) `script.Unit` (없으면 추가)
d) melee인 경우만: `MOD.Core.TweenLineComponent` (없으면 추가) + AutoStart=false

**Enable 값 엔트리 형식 (추가된 컴포넌트마다):**
```json
{
  "TargetType": "컴포넌트명",
  "Name": "Enable",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
  },
  "Value": true
}
```

**TweenLineComponent AutoStart 값 엔트리:**
```json
{
  "TargetType": "MOD.Core.TweenLineComponent",
  "Name": "AutoStart",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
  },
  "Value": false
}
```

**TriggerComponent 추가 시 Values (HitComponent 값 참조):**
```json
{
  "TargetType": "MOD.Core.TriggerComponent",
  "Name": "BoxSize",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "MOD.Core.MODVector2, MOD.Core, Version=1.26.0.0, Culture=neutral, PublicKeyToken=null"
  },
  "Value": {
    "$type": "MOD.Core.MODVector2, MOD.Core",
    "x": <HitComponent BoxSize x>,
    "y": <HitComponent BoxSize y>
  }
},
{
  "TargetType": "MOD.Core.TriggerComponent",
  "Name": "ColliderOffset",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "MOD.Core.MODVector2, MOD.Core, Version=1.26.0.0, Culture=neutral, PublicKeyToken=null"
  },
  "Value": {
    "$type": "MOD.Core.MODVector2, MOD.Core",
    "x": <HitComponent ColliderOffset x>,
    "y": <HitComponent ColliderOffset y>
  }
},
{
  "TargetType": "MOD.Core.TriggerComponent",
  "Name": "IsLegacy",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
  },
  "Value": false
},
{
  "TargetType": "MOD.Core.TriggerComponent",
  "Name": "Enable",
  "ValueType": {
    "$type": "MODNativeType",
    "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
  },
  "Value": true
}
```

4. 수정된 JSON을 Write로 저장 (이동된 경로에 저장)

**NOTE**: Step 2-M의 폴더 경로 매칭은 모델 찾기(Step 2-M 1번) 직후, 이동(2번) 전에 수행해야 합니다.

**폴더 경로 매칭 방법:**
먼저 UnitPrefab 하위에서 해당 폴더명을 검색합니다:
```bash
find D:/MapleWorlds/RootDesk/MyDesk/Prefab/UnitPrefab/ -type d -name "{FolderName}"
```
이 결과에서 `UnitPrefab/` 이후의 상대 경로를 추출합니다. (e.g. `Victoria_Island/Perion`)

만약 UnitPrefab 하위에 해당 폴더가 없으면, 기존 SkillPrefab 경로에서 찾아 참조합니다:
```bash
find D:/MapleWorlds/RootDesk/MyDesk/Prefab/SkillPrefab/UnitSkill/NormalUnitSkill/ -type d -name "{FolderName}"
```

**주의**: Prefab 경로의 폴더명에 공백이 있을 수 있습니다 (e.g. `Elnath Mountain`). 스킬 Prefab/Script 경로에서는 언더스코어로 변환되는 경우도 있으니 (e.g. `Elnath_Mountain`), 양쪽 경로를 모두 확인하여 매칭합니다.

### Step 3: 스킬 모델 생성 (.model)
Write 도구로 스킬 모델 파일을 생성합니다.

**경로 결정:**
- pickup: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\SkillPrefab\UnitSkill\UsableSkill\NormalSkills\Pick_Up_Character\{SkillName}.model`
- normal: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\SkillPrefab\UnitSkill\UsableSkill\NormalSkills\Normal_Character\{SkillName}.model`
- monster: `D:\MapleWorlds\RootDesk\MyDesk\Prefab\SkillPrefab\UnitSkill\NormalUnitSkill\{매칭된경로}\{SkillName}.model`

**내용:**
```json
{
  "Id": "",
  "GameId": "",
  "EntryKey": "model://{SKILL_UUID}",
  "ContentType": "x-mod/model",
  "Content": "",
  "Usage": 0,
  "UsePublish": 1,
  "UseService": 0,
  "CoreVersion": "1.26.0.0",
  "StudioVersion": "0.1.0.0",
  "DynamicLoading": 0,
  "ContentProto": {
    "Use": "Json",
    "Json": {
      "Version": 1,
      "Name": "{SkillName}",
      "BaseModelId": null,
      "Id": "{SKILL_UUID}",
      "Components": [
        "MOD.Core.TransformComponent",
        "script.{SkillName}"
      ],
      "Properties": [],
      "Values": [
        {
          "TargetType": "MOD.Core.TransformComponent",
          "Name": "Enable",
          "ValueType": {
            "$type": "MODNativeType",
            "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
          },
          "Value": true
        },
        {
          "TargetType": "script.{SkillName}",
          "Name": "Enable",
          "ValueType": {
            "$type": "MODNativeType",
            "type": "System.Boolean, mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
          },
          "Value": true
        }
      ],
      "EventLinks": [],
      "Children": []
    }
  }
}
```

### Step 4: 스킬 스크립트 생성 (.mlua)

**경로 결정:**
- pickup: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\Skills\UnitSkill\UsableSkill\NormalSkills\Pick_Up_Character\{SkillName}.mlua`
- normal: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\Skills\UnitSkill\UsableSkill\NormalSkills\Normal_Character\{SkillName}.mlua`
- monster: `D:\MapleWorlds\RootDesk\MyDesk\Scripts\Skills\UnitSkill\NormalUnitSkill\{매칭된경로}\{SkillName}.mlua`

**melee 템플릿:**
```lua
@Component
script {SkillName} extends SkillParent

	@ExecSpace("Client")
	method void ActivateSkill()
		__base:ActivateSkill()
		
		self:CallAnimation("swingO1")
		local options = {
		    FlipX = true,
		}
		
		local function attackFunction()
			self.target.Unit:OnHit(self.caster, self.caster.Unit.damage * self.skillAmount, false, 1)
		end
		
		_TimerService:SetTimerOnce(attackFunction, self.skillAttackTiming)
	end

end
```

**ranged 템플릿:**
```lua
@Component
script {SkillName} extends SkillParent

	@ExecSpace("Client")
	method void ActivateSkill()
		__base:ActivateSkill()
		
		local function attackFunction()
			self.target.Unit:OnHit(self.caster, self.caster.Unit.damage * self.skillAmount, false, 1)
		end
		
		_TimerService:SetTimerOnce(attackFunction, self.skillAttackTiming)
	end

end
```

**IMPORTANT**: .mlua 파일은 반드시 **탭 들여쓰기**를 사용해야 합니다. 스페이스가 아닌 탭 문자(\t)를 사용하세요.

### Step 5: 코드블록 생성 (.codeblock)

**경로**: .mlua와 동일한 폴더에 `{SkillName}.codeblock`

**내용:**
```json
{
  "Id": "",
  "GameId": "",
  "EntryKey": "codeblock://{CB_UUID}",
  "ContentType": "x-mod/codeblock",
  "Content": "",
  "Usage": 0,
  "UsePublish": 1,
  "UseService": 0,
  "CoreVersion": "1.26.0.0",
  "StudioVersion": "0.1.0.0",
  "DynamicLoading": 0,
  "ContentProto": {
    "Use": "Json",
    "Json": {
      "CoreVersion": {
        "Major": 0,
        "Minor": 2
      },
      "ScriptVersion": {
        "Major": 1,
        "Minor": 1
      },
      "Description": "",
      "Id": "{CB_UUID}",
      "Language": 1,
      "Name": "{SkillName}",
      "Type": 1,
      "Source": 0,
      "Target": "codeblock://465afc70-0300-4f88-809a-ac205109680f"
    }
  }
}
```

**IMPORTANT**: Target은 항상 `codeblock://465afc70-0300-4f88-809a-ac205109680f` (SkillParent)입니다.

### Step 6: Unit.csv 행 추가

Bash로 `D:\MapleWorlds\RootDesk\MyDesk\DataSets\Unit.csv`에 행을 추가합니다.

**시너지 데이터**: 아래 중 하나를 랜덤 선택합니다:
```
"{""synergyData"": {""jobId"": 1,""countryId"": 11}}"
"{""synergyData"": {""jobId"": 2,""countryId"": 13}}"
"{""synergyData"": {""jobId"": 3,""countryId"": 14}}"
"{""synergyData"": {""jobId"": 4,""countryId"": 15}}"
"{""synergyData"": {""jobId"": 5,""countryId"": 12}}"
```

**CSV 행 (pickup):**
```
model://{UNIT_UUID},model://{SKILL_UUID},,1,{DisplayName},{Grade},1,1,100,10,{attackRange},{isMelee},1,{projectileSpeed},0.9,30,0,10,1,"""""","""""",{animationName},0,,0,1,{synergyData}
```

**CSV 행 (normal):**
```
model://{UNIT_UUID},model://{SKILL_UUID},,0,{DisplayName},{Grade},1,1,100,10,{attackRange},{isMelee},1,{projectileSpeed},0.9,30,0,10,1,"""""","""""","""""",0,,0,1,{synergyData}
```

**CSV 행 (monster):**
```
model://{EntityID},model://{SKILL_UUID},,0,{DisplayName},{Grade},1,1,100,10,{attackRange},{isMelee},1,{projectileSpeed},0.9,30,0,10,1,"""""","""""","""""",0,,0,1,{synergyData}
```

**값 매핑:**
- `{attackRange}`: melee=`1` / ranged=`3`
- `{isMelee}`: melee=`1` / ranged=`0`
- `{projectileSpeed}`: melee=`1` / ranged=`0.15`
- `{animationName}`: pickup+melee=`swingO1` / pickup+ranged=`shoot1` / normal/monster=`""""""`

**CSV 삽입 위치 규칙:**
- Unit.csv는 `isUsableCharacter` 컬럼(4번째) 기준으로 `0`인 유닛이 위, `1`인 유닛이 아래에 정렬되어 있음
- **monster/normal (isUsableCharacter=0)**: 첫 번째 `isUsableCharacter=1` 행 바로 위에 삽입
- **pickup (isUsableCharacter=1)**: 파일 맨 끝에 추가

**Bash 명령 (monster/normal — 삽입):**
```bash
# 첫 번째 isUsableCharacter=1 행 번호 찾기
INSERT_LINE=$(awk -F',' '$4 == "1" {print NR; exit}' "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Unit.csv")
# 해당 행 바로 위에 삽입
sed -i "${INSERT_LINE}i\\CSV행내용" "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Unit.csv"
```

**Bash 명령 (pickup — 끝에 추가):**
```bash
echo 'CSV행내용' >> "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Unit.csv"
```

### Step 7: Skill.csv 행 추가

**CSV 행 (monster — isUsableSkill=0):**
```
model://{SKILL_UUID},{SkillDisplayName},{desc},{skillArea},{skillAmount},{skillProjectileSpeed},{skillTime},{skillAttackTiming},,{skillIcon},0,{ruids}
```

**CSV 행 (pickup/normal — isUsableSkill=1):**
```
model://{SKILL_UUID},{SkillDisplayName},{desc},{skillArea},{skillAmount},{skillProjectileSpeed},{skillTime},{skillAttackTiming},,{skillIcon},1,{ruids}
```

**참고:**
- `coolTime`은 비워둡니다 (필드 자체는 존재하되 값 없음)
- `skillIcon`, `ruids`도 기본 비워둡니다

**CSV 삽입 위치 규칙:**
- Skill.csv는 `isUsableSkill` 컬럼(11번째) 기준으로 `0`인 스킬이 위, `1`인 스킬이 아래에 정렬되어 있음
- **monster/normal 스킬 (isUsableSkill=0)**: 첫 번째 `isUsableSkill=1` 행 바로 위에 삽입
- **pickup 스킬 (isUsableSkill=1)**: 파일 맨 끝에 추가

**Bash 명령 (monster/normal — 삽입):**
```bash
# 첫 번째 isUsableSkill=1 행 번호 찾기
INSERT_LINE=$(awk -F',' '$11 == "1" {print NR; exit}' "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Skill.csv")
# 해당 행 바로 위에 삽입
sed -i "${INSERT_LINE}i\\CSV행내용" "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Skill.csv"
```

**Bash 명령 (pickup — 끝에 추가):**
```bash
echo 'CSV행내용' >> "D:/MapleWorlds/RootDesk/MyDesk/DataSets/Skill.csv"
```

### Step 8: 결과 요약
생성/수정된 파일 목록과 주요 값을 사용자에게 보여줍니다:
- 유닛 모델 경로 (pickup/normal만)
- 스킬 모델 경로
- 스킬 스크립트 경로
- 코드블록 경로
- Unit.csv에 추가된 행 요약 (유닛 ID, 스킬 ID)
- Skill.csv에 추가된 행 요약

## Example Workflow

### 예시 1: Pickup Character
**User**: "히어로2 픽업캐릭터 melee 5등급으로 만들어줘"

**Assistant actions:**
1. 파싱: Name=`Hero2`, DisplayName=`히어로2`, Type=`pickup`, Combat=`melee`, Grade=`5`
2. 스킬 이름 확인: SkillName=`Sword_Illusion2`, SkillDisplayName=`소드 일루전2`
3. UUID 3개 생성
4. `Unit_Hero.model`을 읽고 UUID/Name 교체하여 `Unit_Hero2.model` 생성
5. `Sword_Illusion2.model` 스킬 모델 생성
6. `Sword_Illusion2.mlua` melee 스킬 스크립트 생성
7. `Sword_Illusion2.codeblock` 생성
8. Unit.csv에 행 추가: `model://UUID1,model://UUID2,,1,히어로2,5,...`
9. Skill.csv에 행 추가: `model://UUID2,소드 일루전2,...`
10. 결과 요약 출력

### 예시 2: Monster
**User**: "스켈레톤워리어 몬스터 melee 3등급 만들어줘, entity ID: abc123-..., 폴더: Perion"

**Assistant actions:**
1. 파싱: Name=`Skeleton_Warrior`, DisplayName=`스켈레톤워리어`, Type=`monster`, Combat=`melee`, Grade=`3`
2. 스킬 이름 확인: SkillName=`Skeleton_Slash`, SkillDisplayName=`스켈레톤 슬래시`
3. entity ID로 모델 파일 찾기
4. 모델 수정: AIWander 제거, DamageSkinSetting/Trigger/Unit/TweenLine 추가
5. 폴더 매칭: `Perion` → `Victoria_Island/Perion`, 모델 파일 이동
6. UUID 2개 생성
7. `Skeleton_Slash.model` 스킬 모델 생성
8. `Skeleton_Slash.mlua` melee 스킬 스크립트 생성
9. `Skeleton_Slash.codeblock` 생성
10. Unit.csv/Skill.csv에 행 추가
11. 결과 요약 출력

## Important Notes

- **탭 들여쓰기**: .mlua 파일은 반드시 탭(\t) 문자를 사용합니다
- **SkillParent Target UUID**: 코드블록의 Target은 항상 `codeblock://465afc70-0300-4f88-809a-ac205109680f`
- **CSV 이스케이프**: synergyData의 JSON은 쌍따옴표를 `""` 로 이스케이프하고 전체를 `""`로 감쌉니다
- **빈 문자열**: CSV의 빈 RUID 값은 `""""""` (triple double-quote)로 표현합니다
- **Monster 모델 수정 시**: 기존 값(Position, 애니메이션 등)은 절대 변경하지 않습니다
- **폴더 생성**: 필요한 폴더가 없으면 기존 `maplestory-folder` 스킬의 패턴에 따라 폴더와 .directory 파일을 함께 생성합니다
