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

1. **Dimmed 영역 분할**: 화면을 4개 영역(Top, Bottom, Left, Right)으로 나눠서 하이라이트 영역 주변만 어둡게 처리
2. **중앙 영역 비우기**: 하이라이트 영역 중앙은 완전히 비워두어 실제 버튼이 클릭 가능
3. **이벤트 연결**: 실제 버튼의 `ButtonClickEvent`에 다음 단계로 진행하는 핸들러를 연결
4. **자동 진행**: 사용자가 실제 버튼을 클릭하면 버튼의 원래 기능이 실행되고, 동시에 튜토리얼이 다음 단계로 진행

### UI 레이아웃
```
화면 전체 (1920x1080)
┌─────────────────────────────────┐
│      DimmedTop (어두운 영역)      │
├─────┬───────────────┬─────────────┤
│Dimmed│ HighlightBorder│  DimmedRight │
│Left  │ (테두리만 표시) │  (어두운 영역)│
│(어두운│ [실제 버튼 클릭]│             │
│ 영역)│               │             │
├─────┴───────────────┴─────────────┤
│     DimmedBottom (어두운 영역)     │
└─────────────────────────────────┘
```

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
| targetButtonPath | string | 클릭할 실제 버튼의 경로 (옵션) | "/ui/Ingame_Shop_Group/ShopButton" |

### DataSet 예시 데이터

| id | order | text | targetX | targetY | targetWidth | targetHeight | highlightType | arrowDirection | targetButtonPath |
|----|-------|------|---------|---------|-------------|--------------|---------------|----------------|------------------|
| 1 | 1 | "상점 버튼을 눌러보세요!" | 1600 | 850 | 200 | 150 | rect | down | "/ui/Ingame_Shop_Group/ShopButton" |
| 2 | 2 | "유닛을 선택하세요" | 100 | 500 | 180 | 180 | rect | right | "/ui/UnitPanel/Unit1" |
| 3 | 3 | "유닛을 배치하세요" | 500 | 300 | 250 | 200 | circle | up | "" |

## 설치 방법

### 1. DataSet 생성
메이플스토리 월드 에디터에서 DataSet을 생성:

1. 에디터에서 DataSet 생성 (이름: "Tutorial")
2. 위의 컬럼 목록대로 컬럼 추가
3. 예시 데이터 입력

### 2. TutorialUI 생성
메이플스토리 월드 에디터에서 UI를 생성:

1. `/ui` 폴더에 "TutorialUI" 그룹 생성
2. 다음 UI 요소들을 추가:
   - **DimmedTop**: Image, 검은색 반투명 (Alpha: 0.7)
   - **DimmedBottom**: Image, 검은색 반투명 (Alpha: 0.7)
   - **DimmedLeft**: Image, 검은색 반투명 (Alpha: 0.7)
   - **DimmedRight**: Image, 검은색 반투명 (Alpha: 0.7)
   - **HighlightBorder**: Image, 노란색 테두리 또는 강조 효과
   - **TutorialTextPanel**: Panel 또는 Image (배경)
   - **TutorialText**: TextMeshPro 또는 Text
   - **TutorialArrow**: Image (화살표 이미지)

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
3. **터치 영역**: 하이라이트 영역만 터치 가능하며, 나머지 영역은 4개의 Dimmed 패널이 차단합니다.
4. **순서**: order 값에 따라 튜토리얼이 순차적으로 진행됩니다.
5. **실제 버튼 클릭**:
   - targetButtonPath를 설정하면 해당 버튼이 실제로 클릭됩니다 (버튼의 원래 기능 실행)
   - 클릭 후 자동으로 다음 튜토리얼 단계로 진행
   - 빈 값("")으로 두면 사용자가 해당 영역을 자유롭게 조작 가능
6. **이벤트 핸들러**: `ConnectEvent`로 연결된 핸들러는 튜토리얼이 진행되는 동안 누적될 수 있습니다. 필요시 이벤트 연결 해제 로직 추가 필요.

## 트러블슈팅

### 튜토리얼이 시작되지 않음
- TutorialManager가 전역 변수 `_TutorialManager`로 등록되었는지 확인
- DataSet "Tutorial"이 올바르게 생성되었는지 확인
- 컬럼 이름이 정확한지 확인 (대소문자 구분)
- 콘솔 로그에서 "Tutorial DataSet not found!" 메시지 확인

### 하이라이트 영역이 잘못된 위치에 표시됨
- targetX, targetY 값 확인
- 화면 좌표계가 좌하단(0,0) 기준인지 확인
- screenWidth, screenHeight 값이 실제 해상도와 일치하는지 확인 (기본: 1920x1080)

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

### 튜토리얼이 다음 단계로 넘어가지 않음
- targetButtonPath가 정확한지 확인
- 버튼에 ButtonClickEvent가 제대로 연결되어 있는지 확인
- 콘솔 로그에서 에러 메시지 확인

## 확장 아이디어

1. **애니메이션 효과**: 하이라이트 영역에 펄스 애니메이션 추가
2. **음성/사운드**: 각 단계마다 사운드 재생
3. **분기 튜토리얼**: 사용자 선택에 따라 다른 경로로 진행
4. **진행률 표시**: UI에 "1/5 단계" 같은 진행률 표시
5. **건너뛰기 버튼**: UI에 "건너뛰기" 버튼 추가

## 라이선스
이 튜토리얼 시스템은 MapleWorlds 프로젝트의 일부입니다.
