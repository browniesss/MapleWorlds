# MapleWorlds Project Analysis

## 1. Project Overview
**Project Name**: MapleWorlds 2D Defense Game (Team 옹기종기)
**Platform**: MapleStory Worlds (MSW)
**Language**: mLua (MSW Custom Lua)

This project is a 2D Defense game where players summon units, manage resources (Meso), and defend against waves of enemies. It features normal/hard modes, event modes, multiplayer synchronization, and a PVP system.

## 2. Directory Structure

The project follows the standard MSW structure with an extensive custom script organization.

- **`Environment/`**: Native scripts (`NativeScripts/`) and configuration.
- **`Global/`**: Global game logic, models (`DefaultPlayer`, `Foothold`), and world config.
- **`map/`**: Map files for Chapters, Stages, Events, and Lobby.
- **`RootDesk/MyDesk/`**: User content directory.
    - **`DataSets/`**: Extensive game data (stages, skills, items, shop, synergy, etc.).
    - **`Prefab/`**: Game object templates.
        - **`ItemPrefab/`**: Drop items (Coins, HP/MP packs).
        - **`SkillPrefab/`**: Visual representations of skills (Projectiles, Effects).
        - **`UI/`**: UI Prefabs (Badges, StageSelect, Synergy, etc.).
        - **`UnitPrefab/`**: Character and Enemy units organized by region/type.
    - **`Scripts/`**: Core game logic.
        - **`Analytics/`**: Analytics service and config.
        - **`GameLogic/`**: Central game management (GameManager, Start/Spawn Managers, varied Services).
        - **`Item/`**: Item behavior scripts.
        - **`Lobby/`**: Looby-specific logic (Portals, StageRanking, UserDeckSetting).
        - **`Map/`**: Minimap and map management.
        - **`Meso/`**: Economy logic (Wallet, Income, UI).
        - **`Npc/`**: NPC behaviors.
        - **`Player/`**: Player logic (Deck, SkillDeck, Character).
        - **`PVP/`**: PVP system (Client, Protocol, Server/Ranking).
        - **`Shop/`**: In-game shop logic (Gold, Diamond, Packages).
        - **`Skills/`**: Skill components and logic (Player, Unit, BaseComponents).
        - **`UI/`**: UI Managers (Popup, etc. - Note: specific UI scripts often co-located or in `Scripts/UI` if present).
        - **`CommonProtocol.mlua`**: Network protocol base.
        - **`ProtocolService.mlua`**: Network service.

## 3. Architecture & Key Components

The code is divided into **Logics** (Singleton Managers) and **Components** (Entity behaviors).

### 3.1. Key Logics (Managers)
These are global services accessible via `_ServiceName`.

- **`GameManager` (`_GameManager`)**:
    - **Role**: Central controller for the game loop.
    - **Responsibilities**: State management (Playing, Event, HardMode), Flow control, UI Orchestration, Timer.

- **`SpawnManager` (`_SpawnManager`)**:
    - **Role**: Unit Spawning Factory.
    - **Responsibilities**: Spawning player units/enemies, Synergy checks, Resource validation.

- **`MonsterSpawnService` / `Repo`**:
    - **Description**: More advanced spawning logic located in `Scripts/GameLogic/MonsterSpawn/`, using repositories for patterns and groups (Event vs Normal).

- **`SkillManager` (`_SkillManager`)**:
    - **Role**: Skill Data & Execution Manager.
    - **Responsibilities**: Skill acquisition, leveling, and execution.

- **`PVP Services`**:
    - **Role**: Manages Player vs Player content.
    - **Components**: `PvpRankingService`, `PvpProtocol`.

- **`ShopService` / `BillingService`**:
    - **Role**: Economy management.
    - **Responsibilities**: Handling Coin/Diamond/Package transactions and rewards.

### 3.2. Key Components (Game Objects)

- **`Unit`**: Core component for Characters/Monsters.
    - **Stats**: Synced HP, MP, Attack, Speed.
    - **State Machine**: Stand, Move, Attack, UseSkill, Die.
    - **AI**: Chase and Wander components (Native/Custom).

- **`PlayerCharacter`**:
    - **Role**: Represents the user's avatar.
    - **Features**: Deck management (`UserDeck`), Skill usage (`UserSkillDeck`).

## 4. Systems Breakdown

### 4.1. UI System
UI logic is distributed across directories but centralized in function:
- **Lobby**: `StageSelectorUI`, `UserDeckSetting`, `StageRanking`.
- **Ingame**: `IngameInfoUI`, `ClearRewardUI`.
- **Shop**: `PurchasePopup`, `MushDiaPassUI`, `EventShopPopup`.
- **PVP**: Specific UI for PVP ranking and matchmaking (inferred).

### 4.2. Data Management
Data is driven by **DataSets** (`RootDesk/MyDesk/DataSets/`):
- **`Stage`**: Configs for stages and events.
- **`Synergy`**: Synergy definitions and bonuses.
- **`SpawnPattern`**: Detailed wave definitions (`SpawnGroup`, `SpawnPattern`).
- **`Shop`**: Prices and rewards for various currencies (`coin`, `dia`, `play_coin`).

### 4.3. Networking Flow
Standard MSW Server-Client architecture:
1.  **Action**: User input (Spawn, Skill).
2.  **Request**: `ProtocolService` or specific Managers call Server functions.
3.  **Validation**: Server checks resources/state.
4.  **Execution**: Server modifies `@Sync` properties or spawns entities.
5.  **Replication**: MSW engine handles property sync.

### 4.4. PVP System (`Scripts/PVP/`)
A structured system for competitive play.
- **Architecture**: Separated into `Client`, `Server`, and `Protocol`.
- **Ranking**: `PvpRankingService` handles ELO/Score.
- **Protocol**: dedicated `pvpProtocol` for match state sync.

## 5. Coding Conventions
- **Execution Spaces**:
    - `@ExecSpace("Server")`: Server-authoritative logic.
    - `@ExecSpace("Client")`: Visuals, Input, UI.
- **Sync**: Use `@Sync` for gameplay-critical data.
- **Services**: Singletons prefixed with `_`.
- **Data Loading**: `OnBeginPlay` loads DataSets.
- **Network Parameters**: Avoid `senderUserId` / `targetUserId` in client-to-server calls.

## 7. Development Tools (MCP)
The **msw-api** MCP server provides specialized tools to assist with MapleStory Worlds development.

### 7.1. References & Guidelines
- **`mluaEntry`**: **Start here** when editing mlua files. Provides guidance on tool usage order.
- **`msw_helper`**: Categorized reference guide. Use to find specific component/service docs (e.g., `classes`, `Service`, `Component`, `Annotation`).
- **`guideline`**: **Must read** for core MSW concepts. Covers execution spaces, sync, hierarchy, and resource management.
- **`grammer`**: Mlua syntax guide. Use when unsure about specific language features (properties, events, sync).

### 7.2. Search & Retrieval
- **`API_Retriever`**: Search for specific API specifications (Classes, Functions, Components).
    - *Example*: "Find functions for `UserService`" or "Properties of `TextComponent`".
- **`Document_Retriever`**: Semantic search for implementation concepts and "how-to" guides.
    - *Example*: "How to make a leaderboard" or "Implementing periodic damage".
