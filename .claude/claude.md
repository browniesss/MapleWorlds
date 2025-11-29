# MapleStory Worlds (MSW) 엔진 개발 가이드

## 중요한 MSW 엔진 특성

### Entity Enable 속성
- `Entity.Enable = false`로 설정하면 엔티티가 **완전히 보이지 않음** (invisible)
- UI 버튼을 비활성화 상태로 **보이게** 하려면 `ButtonComponent.Enable = false`를 사용해야 함
- 예시:
  ```lua
  -- ❌ 잘못된 방법: 버튼이 완전히 사라짐
  self.MyButton.Enable = false

  -- ✅ 올바른 방법: 버튼이 비활성화 상태로 보임
  self.MyButton.Enable = true
  if self.MyButton.ButtonComponent ~= nil then
      self.MyButton.ButtonComponent.Enable = false
  end
  ```

### 이벤트 핸들러 타입
- `EventConnection` 타입은 MSW 코드베이스에 존재하지 않음
- 이벤트 핸들러 프로퍼티는 `property any`를 사용
  ```lua
  -- ❌ 잘못된 방법
  property EventConnection buttonHandler = nil

  -- ✅ 올바른 방법
  property any buttonHandler = nil
  ```

### RUID (Resource Unique Identifier)
- 이미지 RUID는 문자열로 직접 할당 가능
- `RUID()` 래퍼 함수 불필요
  ```lua
  -- ✅ 올바른 방법
  self.Image.SpriteGUIRendererComponent.ImageRUID = "3d291a0daa274583a7856b0470793de5"
  ```

## 코드 작성 원칙

### API 사용 제한
- **코드베이스에 존재하지 않는 프로퍼티나 함수 API는 절대 사용하지 말 것**
- 불확실한 API는 먼저 코드베이스를 검색하여 존재 여부 확인
- MSW 네이티브 API만 사용 (예: `_BadgeService`, `_SpawnService` 등)

### 검증된 패턴만 사용
- 기존 코드베이스에서 사용된 패턴을 따를 것
- 새로운 API나 패턴 도입 전 반드시 기존 코드에서 사용 예시 확인
