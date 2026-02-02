---
name: mlua-scripting
description: 메이플스토리 월드(MSW)의 mLua 스크립트 작성 및 구조 설계를 위한 심화 지침입니다. 최적화 및 안정성 가이드를 포함합니다.
---

# mLua Scripting (Advanced MSW Guide)

이 스킬은 메이플스토리 월드(MSW) 플랫폼에서 사용하는 **mLua** 언어를 사용하여 성능이 최적화되고 구조적으로 견고한 스크립트를 작성하는 방법을 안내합니다.

## 언제 이 스킬을 사용하는가 (When to use this skill)

- MSW용 스크립트(`.mlua`)를 새로 생성하거나 복잡한 로직을 구현할 때.
- 멀티플레이 환경에서 안정적인 네트워크 통신과 데이터 동기화가 필요할 때.
- `OnUpdate` 등 매 프레임 실행되는 로직의 성능 최적화가 필요할 때.

## 사용 방법 (How to use)

### 1. 스크립트 기본 구조 및 엄격한 제약
- **파일당 단일 스크립트**: 하나의 `.mlua` 파일에는 반드시 하나의 `script` 정의만 존재해야 합니다.
- **이름 일치**: 스크립트 이름(`script Name`)은 파일 이름(예: `Unit.mlua` -> `Unit`)과 완벽히 동일해야 합니다.
- **구조 어노테이션**: 스크립트 최상단에 `@Component`, `@Logic`, `@Struct` 중 하나를 반드시 명시합니다. 상속(`extends`)이 없으면 `@Struct`로 간주됩니다.

```lua
@Component
script UserData extends Component
    Property:
        [Sync]
        number Level = 1
    
    Method:
        void OnBeginPlay()
        {
            log("Unit initialized")
        }
end
```

### 2. 함수 정의 및 타입 명시
- **리턴 타입 필수**: 모든 메서드는 리턴 타입을 명시해야 합니다 (예: `void`, `number`, `Entity` 등).
- **호출 규칙**: 인스턴스 메서드는 `:` (self:Method()), 정적 및 루아 표준 라이브러리는 `.` (table.insert())을 사용하여 호출합니다.

### 3. 안정성 및 최적화 (Effective MSW 지침)
- **유효성 검사**: 엔티티나 컴포넌트 접근 시 `nil` 체크 대신 `isvalid(target)`를 사용하십시오. 이는 삭제 대기 중인 객체에 대한 안전한 접근을 보장합니다.
- **Wait 계열 함수 주의**: `wait()`는 현재 스레드를 중단시키므로 남발을 방지하십시오. 병렬 처리가 필요하다면 `_TimerService`를 활용하십시오.
- **OnUpdate 최적화**: 
    - `OnUpdate` 내에서 동기화 프로퍼티(`[Sync]`)를 직접 수정하거나 실행 제어 호출(Client <-> Server)을 수행하는 것은 엄청난 네트워크 비용을 발생시키므로 지양하십시오.
    - 경과 시간 측정 시 동기화 프로퍼티 대신 `_UtilLogic.ServerElapsedSeconds`를 활용하십시오.

### 4. 네트워크 및 실행 제어
- **특정 유저 응답**: 서버에서 특정 클라이언트로만 메시지를 보낼 때, 실행 공간이 `[Client]`인 함수의 마지막 매개변수로 `UserId`를 전달하여 네트워크 비용을 절감하십시오.
- **Local 처리**: 멀티플레이 환경에서는 항상 `self.Entity == _UserService.LocalPlayer` 체크를 통해 내 캐릭터 전용 로직인지 확인하십시오.

### 5. 문법 확장 및 어노테이션
- **확장 연산자**: `+=`, `-=`, `..=`, `&=`, `|=` 등 복합 할당 연산자와 `continue` 키워드를 적극 활용하십시오.
- **타입 힌트**: `---@type`, `---@param`, `---@return` 을 사용하여 코드 가독성과 안정성을 높이십시오.

## 의사 결정 가이드 (Decision Tree)

1. **객체가 유효한지 확인해야 하는가?**
    - Yes -> `isvalid(target)` 사용. (성능상 `target ~= nil`보다 안전)
2. **리턴값이 있는가?**
    - Yes -> 정확한 타입 명시. (없다면 `void`)
3. **특정 유저에게만 결과를 알려야 하는가?**
    - Yes -> `[Client]` 함수 호출 시 마지막 인자로 `targetUserId` 전달.
4. **매 프레임 실행되는가?**
    - Yes -> 네트워크 통신(`[Sync]` 수정, 공간 이동 호출)이 포함되어 있지 않은지 확인.
    - No -> 일반적인 로직 수행.
5. **시간 지연이 필요한가?**
    - **직렬 처리**(순서 중요) -> `wait()`.
    - **병렬 처리**(다른 로직 동시 수행) -> `_TimerService`.
