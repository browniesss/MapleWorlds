# MapleWorlds Project Analysis

## 1. Project Overview
**Project Name**: MapleWorlds 2D Defense Game (Team 옹기종기)
**Platform**: MapleStory Worlds (MSW)
**Language**: mLua (MSW Custom Lua)

This project is a 2D Defense game where players summon units, manage resources (Meso), and defend against waves of enemies. It features normal/hard modes, event modes, and multiplayer synchronization.

## 2. Directory Structure

The project follows the standard MSW structure with custom script organization.

- **`RootDesk/`**: The main workspace directory.
    - **`MyDesk/`**: User content directory.
        - **`Scripts/`**: Core game scripts.
            - **`GameLogic/`**: Central game management logic.
            - **`Units/`**: Unit-specific logic (Unit component, UnitShop).
            - **`Skills/`**: Skill logic (though `SkillManager` is in `GameLogic`).
            - **`UI/`**: UI control scripts, organized by scene/function (Lobby, Ingame, Popup).
            - **`CommonProtocol.mlua`**: Network protocol base.
            - **`ProtocolService.mlua`**: Network service.
- **`Environment/`**: Native scripts and configuration.

## 3. Architecture & Key Components

The code is divided into **Logics** (Singleton Managers) and **Components** (Entity behaviors).

### 3.1. Key Logics (Managers)
These are global services accessible via `_ServiceName`.

- **`GameManager` (`_GameManager`)**:
    - **Role**: Central controller for the game loop.
    - **Responsibilities**:
        - Manages State: `isPlaying`, `isEventMode`, `isHardMode`, `playStartTs`.
        - Controls flow: `GameStart`, `GameEnd`, `StageClear`, `Fail`.
        - UI Orchestration: Toggles Lobby/Ingame UI visibility.
        - Timer Management: Handles the limitation timer.

- **`StageService` (`_StageService`)**:
    - **Role**: Static Data Manager for Stages.
    - **Responsibilities**:
        - Loads **`data:stage`** DataSet on init.
        - Provides stage info (`GetStageById`, `GetNextOpenStageId`).
        - Handles Hard Mode level adjustments (`hard_level`).

- **`SpawnManager` (`_SpawnManager`)**:
    - **Role**: Unit Spawning Factory.
    - **Responsibilities**:
        - `Spawn(modelID, price, pos)`: Spawns player units.
        - `SpawnEnemy(modelID, pos)`: Spawns enemies with level scaling.
        - **Synergy System**: Checks `UserSynergyComponent` to apply buffs (HP/MP/Critical/SummonPoint) to spawned units based on Job/Class.
        - Validates resource (Meso) sufficiency (`IsCanSpawn`).

- **`SkillManager` (`_SkillManager`)**:
    - **Role**: Skill Data & Execution Manager.
    - **Responsibilities**:
        - Loads **`data:IngameSkillData`** DataSet.
        - `GetSkill(skillID)`: Handles skill acquisition and leveling up (1 -> Max).
        - `UseSkill`: Spawns skill entities (projectiles/effects).

### 3.2. Key Components (Game Objects)

- **`Unit`**: The core component for Characters, Monsters, and Towers.
    - **Stats**: `hp`, `mp`, `attack`, `moveSpeed`, `attackSpeed` (Synced).
    - **State Machine**:
        - `Stand`: Idle, searching for targets.
        - `Move`: Moving towards `curTarget`.
        - `Attack`: Melee or Ranged attack based on `attackRange`.
        - `UseSkill`: Executes active skills.
        - `Die`: Death processing.
    - **AI**: Simple aggro logic (`FindTarget`) prioritizing closest enemies.

## 4. Systems Breakdown

### 4.1. UI System (`Scripts/UI/`)
UI logic is separated into components attached to UI Entities.
- **Lobby UI**:
    - `LobbyUI`: Root manager.
    - `StageSelectorUI`: Handles Chapter/Stage selection, Hard Mode toggles, and Ranking display.
    - `StartUI`: Game start entry point.
- **Ingame UI**:
    - `IngameInfoUI`: Displays HP, MP, Wave info (inferred).
    - `ClearRewardUI`: Shows rewards (Coins, Meso, Event Tokens) after clearing a stage.
- **Popup UI**:
    - `FailPopup`: Shows restart/quit options on failure.

### 4.2. Data Management
Data is largely driven by MSW **DataSets** accessed via `_DataService`.
- **`stage`**: Stage config (Chapter, Stage, Name, Monster Level, Tower HP).
- **`IngameSkillData`**: Skill properties (ID, Name, CoolTime, Amount).
- **`EventData`**: Event-specific configurations.

### 4.3. Networking Flow
1.  **Action**: User clicks "Spawn" or Unit attacks.
2.  **Request**: Client calls Server method (e.g., `_SpawnManager:Spawn`).
3.  **Validation**: Server checks resources (Meso) and state.
4.  **Execution**: Server spawns Entity, deducts resources, updates Sync properties.
5.  **Replication**: MSW automatically syncs `@Sync` properties (HP, Pos) to all clients.
6.  **Feedback**: Server may call Client callback (e.g., `_GameManager:StageClearClientCallback`) for local UI effects (Victory anim).

## 5. Game Logic Flow (Example: Stage Clear)
1.  **Trigger**: `Unit` logic or `GameManager` detects win condition (Enemy Tower Dead or Wave Clear).
2.  **Client**: `StageClear()` calls `_GameManager:StageClearServer`.
3.  **Server `StageClearServer`**:
    - **Validation**: Checks play time and cheat detection.
    - **Logic**:
        - Updates `UserStage` (unlocks next stage/hard mode).
        - Calculates Rank Score (`_RankingService`).
        - Distributes Rewards (First Clear vs Re-clear).
        - Handles Event Mode specifics.
4.  **Client Callback**:
    - `StageClearClientCallback` triggered.
    - **UI**: Shows `ClearRewardUI` with calculated rewards.
    - **FX**: Plays "Victory" emotion/animation.

## 6. Coding Conventions
- **Execution Spaces**:
    - `@ExecSpace("Server")`: Server-authoritative logic.
    - `@ExecSpace("Client")`: Visuals, Input, UI.
- **Sync**: Use `@Sync` for gameplay-critical data (HP, State).
- **Services**: All Managers are singletons prefixed with `_` (e.g., `_GameManager`).
- **Data Loading**: `OnBeginPlay` in Logic scripts is used to load DataSets into memory tables.
