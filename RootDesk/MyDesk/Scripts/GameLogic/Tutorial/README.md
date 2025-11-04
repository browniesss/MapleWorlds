# 튜토리얼 시스템 사용 가이드

## 개요
확장 가능한 DataSet 기반 튜토리얼 시스템입니다. 특정 영역만 터치 가능하도록 제한하고, 순차적으로 진행되는 튜토리얼을 구현할 수 있습니다.

## 파일 구조

### 1. TutorialUI.ui
- 튜토리얼 UI 그룹
- 위치: `ui/TutorialUI.ui`
- 구성 요소:
  - **DimmedTop**: 하이라이트 영역 위쪽을 어둡게 처리
  - **DimmedBottom**: 하이라이트 영역 아래쪽을 어둡게 처리
  - **DimmedLeft**: 하이라이트 영역 왼쪽을 어둡게 처리
  - **DimmedRight**: 하이라이트 영역 오른쪽을 어둡게 처리
  - **HighlightBorder**: 하이라이트 영역의 테두리 표시 (시각적 효과)
  - **TutorialTextPanel**: 중앙에 텍스트를 표시하는 패널
  - **TutorialText**: 실제 튜토리얼 텍스트
  - **TutorialArrow**: 방향을 가리키는 화살표 (옵션)

### 2. TutorialManager.mlua
- 튜토리얼 로직을 관리하는 Logic 스크립트 (전역 오브젝트)
- 위치: `RootDesk/MyDesk/Scripts/GameLogic/Tutorial/TutorialManager.mlua`
- DataSet에서 직접 데이터를 로드하고 튜토리얼 진행을 관리

## 동작 원리

### 실제 버튼 클릭 방식
이 튜토리얼 시스템은 하이라이트 영역의 **실제 버튼이 직접 클릭**되도록 설계되었습니다:

1. **자동 위치 감지**: targetButtonPath에서 버튼의 실제 위치와 크기를 자동으로 감지하여 하이라이트 영역 설정 (해상도 무관)
2. **4분할 Dimmed 패널**: DimmedTop, DimmedBottom, DimmedLeft, DimmedRight 4개의 패널로 하이라이트 영역 외부를 완벽히 차단
3. **이벤트 연결**: 실제 버튼의 `ButtonClickEvent`에 다음 단계로 진행하는 핸들러를 연결
4. **자동 진행**: 사용자가 실제 버튼을 클릭하면 버튼의 원래 기능이 실행되고, 동시에 튜토리얼이 다음 단계로 진행

### 해상도 대응
targetButtonPath를 통해 실제 버튼의 Transform을 읽어서 자동으로 위치를 계산합니다. 이 방식은 **모든 해상도에서 동작**합니다. Canvas의 RectSize를 읽어서 화면 크기를 자동 감지하므로 별도 설정이 필요하지 않습니다.

### UI 레이아웃 (4분할 시스템)
```
화면 전체 (1920x1080)
┌─────────────────────────────────┐
│      DimmedTop (상단 영역)        │
│      RaycastTarget = true        │
│      ← 클릭 차단                 │
├───┬───────────────┬───────────────┤
│ D │               │  DimmedRight  │
│ i │ HighlightArea │   (우측 차단)  │
│ m │ (투명 중앙)    │ Raycast=true  │
│ m │ [실제 버튼]    │               │
│ e │ 클릭 가능      │               │
│ d │               │               │
│ L │               │               │
│ e ├───────────────┤               │
│ f │ HighlightBorder│              │
│ t │ (노란 테두리)  │               │
├───┴───────────────┴───────────────┤
│     DimmedBottom (하단 영역)      │
│     RaycastTarget = true          │
│     ← 클릭 차단                   │
└─────────────────────────────────┘
```

## DataSet 설정

메이플스토리 월드 에디터에서 **Tutorial** 이름의 DataSet을 생성하고 다음 컬럼을 추가하세요:

### 컬럼 목록

| 컬럼명 | 타입 | 설명 | 예시 |
|--------|------|------|------|
| id | number | 튜토리얼 ID | 1 |
| order | number | 튜토리얼 순서 (1부터 시작) | 1 |
| text | string | 화면 중앙에 표시할 텍스트 | "상점 버튼을 눌러보세요!" |
| highlightType | string | 하이라이트 타입 ("rect" 또는 "circle") | "rect" |
| arrowDirection | string | 화살표 방향 ("up", "down", "left", "right", "none") | "down" |
| targetButtonPath | string | **(필수)** 클릭할 실제 버튼의 경로 (위치/크기 자동 감지) | "/ui/Ingame_Shop_Group/ShopButton" |

> **중요**: targetButtonPath는 필수 항목입니다. 이 경로를 통해 버튼의 위치와 크기를 자동으로 감지하므로 **모든 해상도에서 정확하게 동작**합니다.

### DataSet 예시 데이터

| id | order | text | highlightType | arrowDirection | targetButtonPath |
|----|-------|------|---------------|----------------|------------------|
| 1 | 1 | "상점 버튼을 눌러보세요!" | rect | down | "/ui/Ingame_Shop_Group/ShopButton" |
| 2 | 2 | "유닛을 선택하세요" | rect | right | "/ui/UnitPanel/Unit1" |
| 3 | 3 | "유닛을 배치하세요" | circle | up | "/ui/UnitPanel/DeployButton" |

## 설치 방법

### 1. DataSet 생성
메이플스토리 월드 에디터에서 DataSet을 생성:

1. 에디터에서 DataSet 생성 (이름: "Tutorial")
2. 위의 컬럼 목록대로 컬럼 추가
3. 예시 데이터 입력

### 2. TutorialUI 확인/수정
메이플스토리 월드 에디터에서 `ui/TutorialUI.ui` 파일을 확인하고 다음과 같은 구조로 설정:

**TutorialUI** (UIGroup)
  - **DimmedTop**: 하이라이트 영역 위쪽을 어둡게 처리 (검은색, Alpha: 0.7, **RaycastTarget: true**)
  - **DimmedBottom**: 하이라이트 영역 아래쪽을 어둡게 처리 (검은색, Alpha: 0.7, **RaycastTarget: true**)
  - **DimmedLeft**: 하이라이트 영역 왼쪽을 어둡게 처리 (검은색, Alpha: 0.7, **RaycastTarget: true**)
  - **DimmedRight**: 하이라이트 영역 오른쪽을 어둡게 처리 (검은색, Alpha: 0.7, **RaycastTarget: true**)
  - **HighlightBorder**: 하이라이트 영역의 테두리 표시 (노란색, Alpha: 1.0, **RaycastTarget: false**)
  - **TutorialTextPanel**: 텍스트 배경 패널
    - **TutorialText**: 튜토리얼 텍스트
  - **TutorialArrow**: 화살표 이미지

> **중요**: 4개의 Dimmed 패널은 RaycastTarget을 true로 설정하여 클릭을 차단하고, HighlightBorder는 false로 설정하여 클릭이 아래 실제 버튼으로 전달되도록 합니다.

### 3. TutorialManager 전역 변수 등록
메이플스토리 월드 에디터에서 TutorialManager를 전역 Logic으로 등록:

1. Hierarchy에서 Logic 폴더를 우클릭 또는 Create Empty
2. 이름을 "TutorialManager"로 변경
3. TutorialManager.mlua 스크립트 추가
4. Inspector에서 "Global Name"을 `_TutorialManager`로 설정

> **참고**: Logic 스크립트는 Unity의 GameManager처럼 실제 게임 오브젝트 없이 전역으로 동작하는 static class입니다.
>
> TutorialManager는 DataSet("Tutorial")에서 직접 데이터를 로드하므로 별도의 Repo가 필요 없습니다.

## 사용 방법

### 기본 사용법

```mlua
-- 튜토리얼 시작
_TutorialManager:StartTutorial()

-- 튜토리얼 스킵
_TutorialManager:SkipTutorial()

-- 튜토리얼 리셋
_TutorialManager:ResetTutorial()
```

### GameManager에 통합 예시

```mlua
@Logic
script GameManager extends Logic

	@ExecSpace("Client")
	method void OnBeginPlay()
		-- 첫 실행 시 튜토리얼 시작
		local hasCompletedTutorial = _DataStorageService:GetStringData("TutorialCompleted")

		if hasCompletedTutorial ~= "true" then
			_TimerService:SetTimerOnce(function()
				_TutorialManager:StartTutorial()
			end, 1.0)
		end
	end

	@ExecSpace("Client")
	method void OnTutorialComplete()
		-- 튜토리얼 완료 표시
		_DataStorageService:SetStringData("TutorialCompleted", "true")
	end
end
```

### 특정 이벤트에서 튜토리얼 시작

```mlua
-- 로비 진입 시 튜토리얼 시작
@EventSender("Service", "UserService")
handler HandlePlayerJoinEvent(PlayerJoinEvent event)
	if event.player == _UserService.LocalPlayer then
		_TimerService:SetTimerOnce(function()
			_TutorialManager:StartTutorial()
		end, 2.0)
	end
end
```

## 고급 기능

### 커스텀 콜백 추가

TutorialManager.mlua를 수정하여 각 단계 완료 시 콜백을 추가할 수 있습니다:

```mlua
@ExecSpace("Client")
method void OnHighlightClick()
	if not self.isActive then
		return
	end

	-- 현재 단계의 콜백 실행 (커스텀)
	local currentData = self.tutorialData[self.currentStep]
	if currentData.callbackFunction ~= nil then
		currentData.callbackFunction()
	end

	-- 다음 단계로 이동
	self.currentStep = self.currentStep + 1
	self:ShowCurrentStep()
end
```

### 동적 좌표 계산

UI 요소의 실제 좌표는 targetButtonPath를 통해 자동으로 계산됩니다:

```mlua
-- TutorialManager가 자동으로 처리하는 로직
local targetButton = _EntityService:GetEntityByPath(stepData.targetButtonPath)
if targetButton ~= nil and targetButton.UITransformComponent ~= nil then
    local transform = targetButton.UITransformComponent
    local rectSize = transform.RectSize
    local anchoredPos = transform.anchoredPosition

    -- 버튼의 중심 위치 기준으로 좌하단 좌표 계산
    stepData.targetX = anchoredPos.x - rectSize.x / 2
    stepData.targetY = anchoredPos.y - rectSize.y / 2
    stepData.targetWidth = rectSize.x
    stepData.targetHeight = rectSize.y
end
```

DataSet에는 targetButtonPath만 입력하면 위치와 크기가 자동으로 감지됩니다.

## 주의사항

1. **좌표 시스템**:
   - TutorialManager가 targetButtonPath를 통해 자동으로 버튼의 위치를 감지합니다
   - 화면 좌하단(0,0)을 기준으로 좌표가 계산됩니다
2. **UI 해상도**:
   - TutorialManager는 시작 시 **자동으로 현재 화면 해상도를 감지**합니다
   - Canvas의 RectSize를 읽어서 screenWidth, screenHeight에 저장
   - 버튼의 실제 위치를 사용하므로 **모든 해상도에서 정확하게 동작**합니다
3. **터치 영역**: 하이라이트 영역만 터치 가능하며, 나머지 영역은 4개의 Dimmed 패널(Top, Bottom, Left, Right)이 완벽히 차단합니다.
4. **순서**: order 값에 따라 튜토리얼이 순차적으로 진행됩니다.
5. **실제 버튼 클릭**:
   - targetButtonPath에 지정된 버튼이 실제로 클릭됩니다 (버튼의 원래 기능 실행)
   - 클릭 후 자동으로 다음 튜토리얼 단계로 진행
   - targetButtonPath는 필수 항목입니다
6. **이벤트 핸들러**: `ConnectEvent`로 연결된 핸들러는 튜토리얼이 진행되는 동안 누적될 수 있습니다. 필요시 이벤트 연결 해제 로직 추가 필요.

## 트러블슈팅

### 튜토리얼이 시작되지 않음
- TutorialManager가 전역 변수 `_TutorialManager`로 등록되었는지 확인
- DataSet "Tutorial"이 올바르게 생성되었는지 확인
- 컬럼 이름이 정확한지 확인 (대소문자 구분)
- 콘솔 로그에서 "Tutorial DataSet not found!" 메시지 확인

### 하이라이트 영역이 잘못된 위치에 표시됨
- targetButtonPath가 정확한 경로인지 확인
- 콘솔에서 "Auto-detected button position" 로그 확인
- 버튼 Entity에 UITransformComponent가 있는지 확인
- screenWidth, screenHeight 값이 실제 해상도와 일치하는지 확인 (콘솔에서 "Tutorial Screen Size" 로그 확인)

### Dimmed 영역이 제대로 표시되지 않음
- DimmedTop, DimmedBottom, DimmedLeft, DimmedRight가 모두 생성되었는지 확인
- 각 Dimmed 패널의 SpriteGUIRendererComponent에 Color가 설정되어 있는지 확인
- Alpha 값이 0.7 정도로 설정되어 있는지 확인

### 화살표가 표시되지 않음
- arrowDirection이 "none"이 아닌지 확인
- 화살표 이미지 RUID가 설정되어 있는지 확인

### 실제 버튼이 클릭되지 않음
- targetButtonPath가 올바른 경로인지 확인
- 해당 경로의 Entity가 실제로 존재하는지 확인
- 버튼이 하이라이트 영역 내에 위치하는지 확인
- Dimmed 패널이 버튼 위를 덮고 있지 않은지 확인
- 콘솔에서 "Auto-detected button position" 로그 확인

### 자동 위치 감지가 작동하지 않음
- targetButtonPath가 정확한 경로인지 확인 (대소문자 구분)
- 버튼 Entity에 UITransformComponent가 있는지 확인
- 콘솔에서 "Auto-detected button position" 로그가 출력되는지 확인
- 로그에 표시된 좌표가 예상과 다르다면 버튼의 Anchor 설정 확인

### 튜토리얼이 다음 단계로 넘어가지 않음
- targetButtonPath가 정확한지 확인
- 버튼에 ButtonClickEvent가 제대로 연결되어 있는지 확인
- 콘솔 로그에서 에러 메시지 확인

### 다른 해상도에서 위치가 맞지 않음
- targetButtonPath를 통한 자동 위치 감지가 활성화되어 있으므로 **모든 해상도에서 동작**합니다
- 콘솔에서 "Tutorial Screen Size" 로그 확인하여 감지된 해상도 확인
- 콘솔에서 "Auto-detected button position" 로그 확인하여 감지된 버튼 위치 확인

## 확장 아이디어

1. **애니메이션 효과**: 하이라이트 영역에 펄스 애니메이션 추가
2. **음성/사운드**: 각 단계마다 사운드 재생
3. **분기 튜토리얼**: 사용자 선택에 따라 다른 경로로 진행
4. **진행률 표시**: UI에 "1/5 단계" 같은 진행률 표시
5. **건너뛰기 버튼**: UI에 "건너뛰기" 버튼 추가
6. **패딩 옵션**: 버튼 주변에 여백(padding)을 추가하여 하이라이트 영역을 버튼보다 크게 표시
7. **튜토리얼 완료 보상**: 튜토리얼 완료 시 게임 내 보상 지급
8. **단계별 콜백**: 각 단계 완료 시 커스텀 함수 실행 기능 추가

## 라이선스
이 튜토리얼 시스템은 MapleWorlds 프로젝트의 일부입니다.
