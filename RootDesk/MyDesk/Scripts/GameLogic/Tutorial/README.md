# 튜토리얼 시스템 사용 가이드

## 개요
확장 가능한 DataSet 기반 튜토리얼 시스템입니다. 특정 영역만 터치 가능하도록 제한하고, 순차적으로 진행되는 튜토리얼을 구현할 수 있습니다.

## 파일 구조

### 1. TutorialRepo.mlua
- DataSet에서 튜토리얼 데이터를 로드하는 저장소
- 위치: `RootDesk/MyDesk/Scripts/GameLogic/Tutorial/TutorialRepo.mlua`

### 2. TutorialUI.ui
- 튜토리얼 UI 그룹
- 위치: `ui/TutorialUI.ui`
- 구성 요소:
  - **DimmedBackground**: 전체 화면을 어둡게 하고 터치를 차단
  - **HighlightArea**: 특정 영역만 하이라이트 및 터치 가능
  - **TutorialTextPanel**: 중앙에 텍스트를 표시하는 패널
  - **TutorialText**: 실제 튜토리얼 텍스트
  - **TutorialArrow**: 방향을 가리키는 화살표 (옵션)

### 3. TutorialManager.mlua
- 튜토리얼 로직을 관리하는 컴포넌트
- 위치: `RootDesk/MyDesk/Scripts/GameLogic/Tutorial/TutorialManager.mlua`

## DataSet 설정

메이플스토리 월드 에디터에서 **Tutorial** 이름의 DataSet을 생성하고 다음 컬럼을 추가하세요:

| 컬럼명 | 타입 | 설명 | 예시 |
|--------|------|------|------|
| id | number | 튜토리얼 ID | 1 |
| order | number | 튜토리얼 순서 (1부터 시작) | 1 |
| text | string | 화면 중앙에 표시할 텍스트 | "상점 버튼을 눌러보세요!" |
| targetX | number | 터치 가능 영역의 X 좌표 (화면 좌하단 기준) | 100 |
| targetY | number | 터치 가능 영역의 Y 좌표 (화면 좌하단 기준) | 200 |
| targetWidth | number | 터치 가능 영역의 너비 | 200 |
| targetHeight | number | 터치 가능 영역의 높이 | 150 |
| highlightType | string | 하이라이트 타입 ("rect" 또는 "circle") | "rect" |
| arrowDirection | string | 화살표 방향 ("up", "down", "left", "right", "none") | "down" |

### DataSet 예시 데이터

| id | order | text | targetX | targetY | targetWidth | targetHeight | highlightType | arrowDirection |
|----|-------|------|---------|---------|-------------|--------------|---------------|----------------|
| 1 | 1 | "상점 버튼을 눌러보세요!" | 1600 | 850 | 200 | 150 | rect | down |
| 2 | 2 | "유닛을 선택하세요" | 100 | 500 | 180 | 180 | rect | right |
| 3 | 3 | "유닛을 배치하세요" | 500 | 300 | 250 | 200 | circle | up |

## 설치 방법

### 1. 전역 변수 등록
메이플스토리 월드 에디터에서 TutorialRepo를 전역 변수로 등록:

1. Hierarchy에서 Logic 폴더를 우클릭
2. "Create Empty" 선택
3. 이름을 "TutorialRepo"로 변경
4. TutorialRepo 스크립트 컴포넌트 추가
5. Inspector에서 "Global Name"을 `_tutorialRepo`로 설정

### 2. TutorialManager 컴포넌트 추가
원하는 엔티티(예: GameManager)에 TutorialManager 컴포넌트를 추가합니다.

## 사용 방법

### 기본 사용법

```mlua
-- 튜토리얼 시작
_GameManager.TutorialManager:StartTutorial()

-- 튜토리얼 스킵
_GameManager.TutorialManager:SkipTutorial()

-- 튜토리얼 리셋
_GameManager.TutorialManager:ResetTutorial()
```

### GameManager에 통합 예시

```mlua
@Logic
script GameManager extends Logic

	property Entity tutorialManager = nil

	@ExecSpace("Client")
	method void OnBeginPlay()
		-- 첫 실행 시 튜토리얼 시작
		local hasCompletedTutorial = _DataStorageService:GetStringData("TutorialCompleted")

		if hasCompletedTutorial ~= "true" then
			_TimerService:SetTimerOnce(function()
				self.tutorialManager.TutorialManager:StartTutorial()
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
			_GameManager.TutorialManager:StartTutorial()
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

UI 요소의 실제 좌표를 가져와서 튜토리얼을 설정할 수 있습니다:

```mlua
-- 특정 UI 요소의 좌표 가져오기
local shopButton = _EntityService:GetEntityByPath("/ui/Ingame_Shop_Group/ShopButton")
local transform = shopButton.UITransformComponent

-- 스크린 좌표로 변환
local screenPos = transform.anchoredPosition
local width = transform.RectSize.x
local height = transform.RectSize.y

-- DataSet에 입력할 값
-- targetX = screenPos.x - width/2
-- targetY = screenPos.y - height/2
-- targetWidth = width
-- targetHeight = height
```

## 주의사항

1. **좌표 시스템**: targetX, targetY는 화면 좌하단(0,0)을 기준으로 합니다.
2. **UI 해상도**: 기본 해상도는 1920x1080입니다. 다른 해상도에서 테스트 필요.
3. **터치 영역**: HighlightArea만 터치 가능하며, 나머지 영역은 DimmedBackground가 차단합니다.
4. **순서**: order 값에 따라 튜토리얼이 순차적으로 진행됩니다.

## 트러블슈팅

### 튜토리얼이 시작되지 않음
- TutorialRepo가 전역 변수 `_tutorialRepo`로 등록되었는지 확인
- DataSet "Tutorial"이 올바르게 생성되었는지 확인
- 컬럼 이름이 정확한지 확인 (대소문자 구분)

### 하이라이트 영역이 잘못된 위치에 표시됨
- targetX, targetY 값 확인
- 화면 좌표계가 좌하단(0,0) 기준인지 확인
- UI 요소의 Anchor 설정 확인

### 화살표가 표시되지 않음
- arrowDirection이 "none"이 아닌지 확인
- 화살표 이미지 RUID가 설정되어 있는지 확인 (현재는 빈 값)

## 확장 아이디어

1. **애니메이션 효과**: 하이라이트 영역에 펄스 애니메이션 추가
2. **음성/사운드**: 각 단계마다 사운드 재생
3. **분기 튜토리얼**: 사용자 선택에 따라 다른 경로로 진행
4. **진행률 표시**: UI에 "1/5 단계" 같은 진행률 표시
5. **건너뛰기 버튼**: UI에 "건너뛰기" 버튼 추가

## 라이선스
이 튜토리얼 시스템은 MapleWorlds 프로젝트의 일부입니다.
