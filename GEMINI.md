# Gemini: MapleWorld 2D 디펜스 게임 프로젝트

## 1. 프로젝트 개요

이 프로젝트는 **Nexon MapleStory Worlds** 플랫폼을 기반으로 개발된 2D 디펜스 게임입니다. 인기 게임인 '냥코 대전쟁'과 유사한 게임 플레이 방식을 가지고 있으며, '옹기종기' 팀에서 개발을 진행하고 있습니다.

- **플랫폼**: Nexon MapleStory Worlds
- **장르**: 2D 디펜스
- **개발 언어**: mLua (MapleStory Worlds Script)
- **핵심 특징**: 단일 스크립트 파일 내에 서버와 클라이언트 로직이 공존하는 독특한 아키텍처를 가집니다.

## 2. 프로젝트 구조

주요 디렉토리 구조는 다음과 같습니다.

- `map/`: 게임의 다양한 스테이지와 로비 등 맵 파일(.map)이 위치합니다.
- `RootDesk/MyDesk/`: 게임의 핵심 로직을 담고 있는 스크립트(.mlua), 모델(.model), UI 스크립트(.codeblock) 등이 위치하는 메인 개발 디렉토리입니다.
- `ui/`: 게임에 사용되는 각종 UI 요소(.ui)들이 정의되어 있습니다.
- `Environment/`: MapleStory Worlds에서 제공하는 기본 컴포넌트, 서비스, 이벤트 등의 스크립트 인터페이스(.d.mlua)가 정의되어 있습니다. 이 파일들은 코드 어시스트 및 타입 체킹에 사용됩니다.
- `Global/`: 게임 전역에서 사용되는 설정(충돌 그룹, 기본 플레이어 모델 등) 파일이 위치합니다.

## 3. 핵심 아키텍처: 3-Tier 통신 구조 (Client-Protocol-Server)

이 프로젝트는 명확한 역할 분리와 데이터 흐름 관리를 위해 **Client, Protocol, Server**의 3-Tier 아키텍처를 채택하고 있습니다. 이 구조는 `Scripts/Units`와 같은 기능 단위별 폴더에 구현되어 있습니다.

-   **`Server` (서버)**: 실제 로직을 처리하는 두뇌 역할을 합니다.
    -   `@ExecSpace("ServerOnly")`로 지정됩니다.
    -   데이터베이스 조회, 유닛 레벨업, 재화 차감 등 핵심적이고 보안이 필요한 모든 연산을 수행합니다.
    -   클라이언트로부터 직접 호출되지 않으며, 항상 `Protocol`을 통해 통신합니다.
    -   예시: `UserMonsterService.mlua`

-   **`Client` (클라이언트)**: 사용자 인터페이스(UI)와 상호작용하고 사용자 입력을 받습니다.
    -   `@ExecSpace("ClientOnly")`로 지정됩니다.
    -   사용자의 요청이 발생하면, `Protocol`을 통해 서버에 작업을 요청합니다.
    -   서버로부터 `Protocol`을 통해 전달받은 데이터를 UI에 반영하는 역할을 합니다.
    -   예시: `UserUnitStore.mlua`

-   **`Protocol` (프로토콜)**: 서버와 클라이언트 간의 통신을 중개하는 다리 역할을 합니다.
    -   RPC(Remote Procedure Call) 인터페이스를 정의합니다.
    -   클라이언트의 요청을 받아 서버에 전달하는 `...Call` 메소드 (`@ExecSpace("Server")`)를 가집니다.
    -   서버의 처리 결과를 다시 클라이언트에 전달하는 `...Back` 메소드 (`@ExecSpace("Client")`)를 가집니다.
    -   이를 통해 서버와 클라이언트 코드가 직접적으로 서로를 호출하는 것을 방지하고, 통신 창구를 일원화하여 데이터 흐름을 명확하게 만듭니다.
    -   예시: `UserUnitProtocol.mlua`

### 통신 흐름 예시 (유저 유닛 리스트 요청)

1.  **Client (`UserUnitStore`)**: 게임이 시작되면 `OnBeginPlay`에서 `UserUnitProtocol:GetUserUnitListCall()`을 호출하여 서버에 데이터 요청.
2.  **Protocol (`UserUnitProtocol`)**: `GetUserUnitListCall` 메소드가 서버 공간에서 실행됨. `UserMonsterService`를 호출하여 실제 데이터 조회를 지시.
3.  **Server (`UserMonsterService`)**: 데이터베이스에서 유저의 몬스터 데이터를 조회하고 가공함.
4.  **Protocol (`UserUnitProtocol`)**: `UserMonsterService`로부터 받은 결과를 `GetUserUnitListBack(결과)` 메소드를 통해 요청한 클라이언트에 전달.
5.  **Client (`UserUnitStore`)**: `GetUserUnitListBack`으로부터 받은 결과를 `UserUnitMap`에 저장하고 UI를 갱신.

---

## 4. 개발 가이드

이 프로젝트에서 새로운 기능을 개발하거나 기존 코드를 수정할 때는 다음 아키텍처 원칙을 반드시 준수해야 합니다.

1.  **3-Tier 구조 준수**: 모든 클라이언트-서버 간 통신은 `Client`, `Protocol`, `Server` 3-Tier 구조를 따라야 합니다. 기능을 추가할 때 각 역할에 맞는 스크립트를 생성하거나 수정하세요.

2.  **실행 영역 명시**: 모든 메소드에 `@ExecSpace`를 사용하여 실행 환경(서버/클라이언트)을 명확히 지정해야 합니다. 특히 `Protocol` 스크립트의 `...Call`은 `@ExecSpace("Server")`, `...Back`은 `@ExecSpace("Client")`로 지정하는 규칙을 따릅니다.

3.  **신뢰 경계 구분**: 사용자 입력은 항상 `Client`에서 시작되지만, 모든 검증, 연산, 데이터 저장은 `Server`에서 이루어져야 합니다. `Client`에서 보낸 데이터를 절대 신뢰하지 마세요.

### **[중요] 데이터 동기화: `@Sync` 프로퍼티 사용 금지**

이 프로젝트에서는 데이터 흐름의 명확성을 확보하고 잠재적인 성능 문제를 방지하기 위해 **프로퍼티(property)에 `@Sync` 어노테이션을 사용하는 것을 **강력히 금지**합니다.**

-   **이유**: `@Sync`는 사용이 간편하지만, 데이터 변경이 잦을 경우 불필요한 네트워크 트래픽을 유발하여 성능 저하의 원인이 될 수 있습니다. 또한, 데이터가 언제 어떻게 동기화되는지 흐름을 추적하기 어렵게 만듭니다.
-   **대체 방안**: 모든 데이터 조회 및 변경은 상술한 **`Protocol`을 통한 명시적인 `Call/Back` RPC 패턴**을 사용해야 합니다. 클라이언트가 데이터가 필요할 때 `...Call`로 서버에 요청하고, 서버는 처리 후 `...Back`으로 결과를 돌려주는 방식은 데이터 흐름을 명확하고 예측 가능하게 만듭니다.

이 가이드를 통해 프로젝트의 아키텍처 일관성을 유지하고, 안정적이며 효율적인 코드를 작성할 수 있습니다.
