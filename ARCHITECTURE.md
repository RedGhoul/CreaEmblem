# CreaEmblem - Architecture Documentation

## Table of Contents
- [Overview](#overview)
- [Project Structure](#project-structure)
- [Core Systems](#core-systems)
- [Code Flow](#code-flow)
- [Scene Architecture](#scene-architecture)
- [Data Flow](#data-flow)
- [Key Design Patterns](#key-design-patterns)

---

## Overview

CreaEmblem is a **Fire Emblem: Heroes-like tactical RPG** built in Unity. It features turn-based combat, character progression, weapon triangle mechanics, and a modular asset bundle system for content.

**Key Statistics:**
- **Language**: C# with Unity
- **Code Base**: ~3,700 lines across 39 scripts
- **Scenes**: 2 (Menu and Game)
- **Heroes**: 12+ playable characters
- **Levels**: 8 playable maps
- **Grid Size**: 26x16 tiles

---

## Project Structure

```
/CreaEmblem
├── Assets/
│   ├── Scripts/               # All C# game logic
│   │   ├── Game/             # Core game systems
│   │   ├── Battle/           # Combat system
│   │   ├── Heroes/           # Character management
│   │   ├── Map/              # Tilemap and pathfinding
│   │   ├── Menu/             # UI controllers
│   │   ├── Utils/            # Helpers and utilities
│   │   └── Data/             # Data structures (ScriptableObjects)
│   ├── Data/                 # ScriptableObject assets
│   │   ├── Heroes/           # Hero definitions
│   │   ├── Weapons/          # Weapon definitions
│   │   ├── Mounts/           # Movement type definitions
│   │   ├── Stats/            # Stat curve definitions
│   │   └── Zones/            # Terrain type definitions
│   ├── Scenes/               # Unity scenes
│   │   ├── Menu.unity        # Main menu, team/level selection
│   │   └── SampleScene.unity # Game battlefield
│   ├── Maps/                 # Level data and tilemaps
│   ├── Prefabs/              # Reusable game objects
│   ├── Sprites/              # UI and terrain sprites
│   ├── Animations/           # Animation controllers
│   └── StreamingAssets/      # Asset bundles for runtime loading
├── AssetBundles/             # Compiled asset bundles
├── Fmod/                     # FMOD audio project
└── ProjectSettings/          # Unity configuration
```

---

## Core Systems

### 1. Data Management System

**Location**: `Assets/Scripts/Game/DataManager.cs`

**Purpose**: Singleton that manages global game state, asset loading, and persistence.

**Key Responsibilities:**
- Loads all asset bundles at startup
- Manages game state machine
- Handles turn transitions
- Saves/loads configuration and hero data
- Coordinates audio system
- Fires game-wide events

**Game State Machine:**
```csharp
enum GameState {
    None,        // Game not started
    Start,       // Initializing
    Playing,     // Active gameplay
    Pause,       // Menu paused
    Battle,      // Battle animation
    Transition,  // Turn transition
    End,         // Game finished
    NbState
}
```

**Key Events:**
```csharp
onStartTurn    // Fired when a team's turn begins
onEndTurn      // Fired when a team's turn ends
onStartGame    // Game begins
onEndGame      // Game ends, announces winner
onHeroDeath    // Hero dies
```

---

### 2. Battle System

**Location**: `Assets/Scripts/Battle/BattleManager.cs`

**Purpose**: Orchestrates combat sequences between heroes.

**Battle Flow:**
```
1. Attacker attacks defender
2. Wait for animation
3. If defender alive → Defender counter-attacks
4. Wait for animation
5. Check speed stat:
   - If attacker.speed >= defender.speed + 5 → Attacker attacks again
   - If defender.speed >= attacker.speed + 5 → Defender attacks again
6. Both heroes gain 50 EXP per attack
7. Check for level ups
8. Return to gameplay
```

**Key Mechanics:**

**Weapon Triangle:**
```
Sword > Axe > Spear > Sword
  (+20% damage to strong matchup)
  (-20% damage to weak matchup)
Bow: No advantages, 2-tile range
```

**Damage Calculation:**
```csharp
// Physical damage
damage = attacker.ATK + (weaponAdvantage * attacker.ATK) - defender.DEF

// Magical damage
damage = attacker.ATK + (weaponAdvantage * attacker.ATK) - defender.RES
```

**Experience System:**
- Both attacker and defender gain 50 EXP per attack
- Level up when EXP threshold is reached
- Stats increase based on AnimationCurve definitions

---

### 3. Hero System

**Location**: `Assets/Scripts/Heroes/Hero.cs`, `HeroManager.cs`

**Purpose**: Manages individual characters and their progression.

**Hero Components:**

**Hero.cs** - Individual character logic
```csharp
public HeroData data;           // Static character definition
public HeroInfo info;           // Persistent level/exp data
public int team;                // Team affiliation (0 or 1)
public bool canPlay;            // Can act this turn?
public int life;                // Current HP
public Vector3Int gridPosition; // Position on map
```

**Character Stats** (calculated from level):
- **HP**: Health points (maxLife)
- **ATK**: Attack power
- **DEF**: Physical damage reduction
- **RES**: Magical damage reduction
- **SPD**: Determines double-attack eligibility

**HeroManager.cs** - Persistent data management
- Loads all available heroes from asset bundles
- Manages `HeroInfo` persistence (level, EXP)
- Saves/loads hero progression to/from `heroes.json`
- Automatically creates missing hero entries (starts at Level 1)

---

### 4. Map & Movement System

**Location**: `Assets/Scripts/Map/MapManager.cs`, `MapData.cs`

**Purpose**: Manages the tactical grid, pathfinding, and spatial queries.

**MapManager.cs** - Static utility class

Key methods:
```csharp
GetTileUnderMouse()           // Get tile at cursor
GetTileAtPosition(Vector3)    // World → grid position
GetTileCenter(Vector3Int)     // Grid → world position
GetHeroAtTile(Vector3Int)     // Check tile occupancy
GetZoneAtPosition()           // Get terrain type
GetAllHeroes(team=-1)         // Get heroes on map
SetPathTo(Vector3Int)         // Visualize movement path
```

**MapData.cs** - Level definition

```csharp
public static int width = 26;
public static int height = 16;
public PaletteData palette;           // Tile definitions
public int[,] data;                   // Tile grid
public List<HeroStartInfo> heroStarts // Hero spawn positions
```

**Map File Format:**
- `*_map.csv`: Comma-separated tile palette indices (26x16 grid)
- `*_start.csv`: Hero spawn positions (team index per tile)

**Movement & Pathfinding:**
- Uses breadth-first search (BFS) for pathfinding
- `GetReachableTiles()` calculates:
  - Movement range (based on mount type)
  - Attack range (based on weapon type)
  - Walkable terrain (based on mount capabilities)

---

### 5. Control System

**Location**: `Assets/Scripts/Game/Controller.cs`

**Purpose**: Handles player input and translates it to game actions.

**Input Flow:**
```
Click tile with hero → Select hero
  ↓
Click walkable tile → Preview path
  ↓
Click same tile again → Confirm move
  ↓
Click attackable enemy → Preview attack
  ↓
Click enemy again → Execute attack
```

**Visualization:**
- **Blue highlights**: Walkable tiles (opacity = availability)
- **Red highlights**: Attackable tiles (with enemies)
- **Path animation**: Shows movement route

---

### 6. Asset Loading System

**Location**: `Assets/Scripts/Utils/AssetBundleManager.cs`

**Purpose**: Dynamically loads game content from asset bundles.

**Features:**
- Loads asset bundles from `StreamingAssets/`
- Caches loaded bundles to avoid reloading
- Type-filtered asset extraction
- Supports content addition without recompilation

**Bundle Categories:**
```
/heroes/           - Individual hero data
/terrains/zones/   - Terrain type definitions
/terrains/palettes/- Tile palette sets
/stats/            - Character stat curves
/weapons/          - Weapon definitions
/mounts/           - Movement type definitions
/levels/           - Level/map data
```

---

### 7. Audio System

**Location**: `Assets/Scripts/Utils/FMODSoundManager.cs`

**Purpose**: FMOD integration for dynamic audio.

**Key Audio Events:**
```csharp
"Musics/BGM"           // Dynamic music (menu, selection, battle, victory)
"Game/TileClicked"     // UI feedback
"Game/MoveHero"        // Movement confirmation
"Game/AttackClicked"   // Attack selection
"Game/Attack"          // Damage hit
"Game/StartBattle"     // Battle animation start
"Game/ExpGain"         // Experience gain
"Game/LevelUp"         // Level up notification
```

**Volume Control:**
- VCAs: Master, Music, SFX, UI
- Settings persistent in `config.txt`
- Music parameters: Type (Menu/Selection/Battle/Victory)

---

## Code Flow

### Game Initialization Sequence

```
1. Application Start
   ↓
2. DataManager.Awake() [Execution Order: -1000]
   - Create singleton instance
   - Initialize audio manager
   - Load hero manager
   ↓
3. DataManager.Start()
   - Load all asset bundles:
     • Stats (HP, ATK, DEF, RES, SPD, EXP)
     • Weapons (Sword, Spear, Axe, Bow)
     • Mounts (Feet, Horse, Pegasus, Dragon)
     • Terrain zones (Ground, Water, Forest, Mountain)
     • Terrain palettes
   - Load configuration (config.txt)
   - Load hero progression (heroes.json)
   - Default hero team selection
   ↓
4. Menu Scene Loads
   - MenuManager initializes
   - TeamSelectionUI ready
   - LevelSelectionUI ready
```

### Gameplay Loop

```
1. Level Selection
   ↓
2. Load Level Bundle → Extract MapData
   ↓
3. Load Game Scene (SampleScene)
   ↓
4. MapManager.Start() [Execution Order: -100]
   - Load tilemap from MapData
   - Load hero bundles
   - Spawn heroes at start positions
   ↓
5. Hero.Start()
   - Register with MapManager
   - Initialize stats based on level
   ↓
6. Controller.Start() [Execution Order: 100]
   - Register event listeners
   - Initialize input system
   ↓
7. Game Loop Begins
   ↓
8. Player Turn:
   a. onStartTurn event fired
   b. Player selects hero (Controller)
   c. Player clicks destination
   d. Hero moves to position
   e. Player clicks enemy (optional)
   f. Battle sequence (BattleManager)
   g. Experience gained, check level up
   h. Player clicks "End Turn"
   ↓
9. Turn Transition:
   - onEndTurn event fired
   - ChangeTurnUI animation
   - Switch active team
   - Reset hero canPlay flags
   ↓
10. Opponent Turn (repeat step 8)
    ↓
11. Check Victory Condition:
    - If team has no living heroes → Game Over
    - onEndGame event fired
    - GameOverUI displays winner
    - Return to Menu
```

---

## Scene Architecture

### Scene 0: Menu.unity

**Hierarchy:**
```
Canvas (MenuManager)
├── TitleScreen
│   ├── Play Button → Opens TeamSelection
│   ├── Settings Button → Opens GameSettings
│   └── Credits Button → Opens Credits
├── TeamSelection (TeamSelectionUI)
│   ├── Player 1 Team (4 slots)
│   ├── Player 2 Team (4 slots)
│   └── Start Button → Opens LevelSelection
├── LevelSelection (LevelSelectionUI)
│   ├── Level Grid (8 buttons)
│   └── Back Button
└── GameSettings
    ├── Volume Sliders (Master, Music, SFX, UI)
    └── Back Button
```

**Flow:**
```
Title Screen → Team Selection → Level Selection → Load Game Scene
      ↑______________________________________________|
                    (Return on Game Over)
```

---

### Scene 1: SampleScene.unity (Game)

**Hierarchy:**
```
DataManager (Singleton, DontDestroyOnLoad)
├── AudioManager (FMOD)
└── HeroManager

MapManager (Static)
├── Grid
│   ├── Tilemap (Terrain)
│   ├── PathTilemap (Movement visualization)
│   └── GridInformation (ZoneData storage)
└── Heroes (spawned at runtime)

Controller (Input handling)

BattleManager
└── BattleCutscene (AnimatedSprite cutscenes)

UI Canvas
├── StatsUI (Hero info panel)
├── EndTurnButton
├── InGameMenu (Pause menu)
├── ExperienceUI (Level up animations)
├── ChangeTurnUI (Turn transition)
└── GameOverUI (Victory/defeat screen)

Main Camera
```

**Update Loop:**
```
Controller.Update()
  ↓
Check mouse input
  ↓
If tile clicked:
  - Get tile under mouse
  - Check if hero on tile
  - Process selection/movement/attack
  ↓
Update hero animations (AnimatedSprite)
  ↓
Update UI (HeroCanvas, StatsUI)
```

---

## Data Flow

### Hero Selection Flow

```
User clicks hero in TeamSelectionUI
  ↓
Add HeroInfo to DataManager.heroToSpawn list
  ↓
Load level → Game Scene
  ↓
MapManager spawns heroes from heroToSpawn
  ↓
Hero.Start() loads HeroData from asset bundle
  ↓
Hero stats calculated from HeroInfo.level
```

---

### Combat Data Flow

```
Player clicks enemy hero
  ↓
Controller validates attack (range, obstacles)
  ↓
Controller.AttackHero() called
  ↓
BattleManager.StartBattle(attacker, defender)
  ↓
GameState = Battle (locks input)
  ↓
BattleCoroutine:
  1. Play attack animation (BattleCutscene)
  2. Calculate damage (Hero.GetDamage)
  3. Apply damage (Hero.TakeDamage)
  4. Check death (Hero.life <= 0)
  5. Play counter-attack animation
  6. Calculate counter damage
  7. Apply counter damage
  8. Check speed for double attack
  9. Award experience (both heroes)
  10. Check level ups (ExperienceUI)
  ↓
GameState = Playing (unlocks input)
  ↓
Check victory condition (MapManager.GetAllHeroes)
```

---

### Pathfinding Data Flow

```
Player selects hero
  ↓
Hero.GetReachableTiles() called
  ↓
BFS from hero position:
  - Check terrain height (ZoneData)
  - Check mount restrictions (MountData)
  - Check tile occupancy (MapManager)
  - Calculate distance from start
  ↓
Return list of reachable tiles
  ↓
Controller visualizes tiles:
  - Blue = walkable
  - Red = attackable (enemy on tile)
  ↓
Player clicks tile
  ↓
Hero.GetPathTo() called
  ↓
Reconstruct path from BFS parent map
  ↓
MapManager.SetPathTo() visualizes path
  ↓
Player confirms movement
  ↓
Hero moves along path (smooth interpolation)
```

---

### Save/Load Data Flow

**On Game Start:**
```
DataManager.Awake()
  ↓
DataFile.LoadData("config.txt")
  ↓
Parse volume settings (Master, Music, SFX, UI)
  ↓
Apply to AudioManager VCAs
  ↓
HeroManager.LoadHeroes()
  ↓
DataFile.LoadData("heroes.json")
  ↓
Deserialize List<HeroInfo>
  ↓
Create missing hero entries (default Level 1)
```

**On Game Exit:**
```
Application.Quit() / OnApplicationQuit()
  ↓
DataFile.SaveData("config.txt", volume settings)
  ↓
HeroManager.SaveHeroes()
  ↓
DataFile.SaveData("heroes.json", hero progression)
```

---

## Key Design Patterns

### 1. Singleton Pattern
**Used in**: DataManager, MapManager, BattleManager

```csharp
public static DataManager instance;

void Awake() {
    if (instance == null) {
        instance = this;
        DontDestroyOnLoad(gameObject);
    } else {
        Destroy(gameObject);
    }
}
```

**Purpose**: Single global access point for core systems.

---

### 2. Event-Driven Architecture
**Used throughout**: Game state changes, turn transitions, hero death

```csharp
public UnityEvent onStartTurn;
public UnityEvent onEndTurn;
public UnityEvent<int> onEndGame;

// Usage
DataManager.instance.onStartTurn.AddListener(OnTurnStart);
DataManager.instance.onStartTurn.Invoke();
```

**Purpose**: Decouples systems, allows UI to react to game events.

---

### 3. ScriptableObject Pattern
**Used in**: Hero, Weapon, Mount, Stat, Zone, Palette, Map data

```csharp
[CreateAssetMenu(fileName = "Hero", menuName = "Data/Hero")]
public class HeroData : ScriptableObject {
    public string displayName;
    public Sprite icon;
    // ... other data
}
```

**Purpose**: Data-driven design, allows non-programmers to create content.

---

### 4. Asset Bundle Pattern
**Used in**: Dynamic content loading

```csharp
AssetBundle bundle = AssetBundle.LoadFromFile(path);
HeroData hero = bundle.LoadAsset<HeroData>("lucina");
```

**Purpose**: Modular content loading, reduces compile times.

---

### 5. Component Pattern
**Used in**: Hero system (HeroData + HeroInfo + Hero)

```csharp
public class Hero : MonoBehaviour {
    public HeroData data;      // Static definition
    public HeroInfo info;      // Runtime state
    // ... behavior
}
```

**Purpose**: Separates data from behavior, enables persistence.

---

### 6. State Machine Pattern
**Used in**: GameState enum

```csharp
switch (DataManager.instance.gameState) {
    case GameState.Playing:
        // Handle input
        break;
    case GameState.Battle:
        // Input locked during animation
        break;
}
```

**Purpose**: Clear game flow control, prevents invalid actions.

---

### 7. Coroutine Pattern
**Used in**: Battle sequences, animations

```csharp
IEnumerator BattleCoroutine(Hero attacker, Hero defender) {
    yield return new WaitWhile(() => animating);
    // Next step
}
```

**Purpose**: Asynchronous operations without blocking main thread.

---

### 8. Observer Pattern (Unity Events)
**Used in**: UI updates, game state changes

```csharp
public UnityEvent<Hero> onHeroSelected;

// Subscribe
onHeroSelected.AddListener(UpdateUI);

// Notify
onHeroSelected.Invoke(selectedHero);
```

**Purpose**: Loosely coupled communication between systems.

---

## Script Execution Order

Unity's script execution order is configured for proper initialization:

```
-1000: DataManager       (loads data first)
 -100: MapManager        (builds map)
   -1: AnimatedSprite, Controller base
    0: Standard scripts
   50: ChangeTurnUI
  100: Controller, HeroCanvas, TeamSelectionUI, LevelSelectionUI
  200: ExperienceUI
 1000: GameOverUI        (processes last)
```

This ensures:
1. Data is loaded before anything needs it
2. Map exists before heroes spawn
3. UI initializes after game systems

---

## File Naming Conventions

**Scripts**: PascalCase (e.g., `DataManager.cs`, `HeroCanvas.cs`)

**ScriptableObjects**: PascalCase (e.g., `HeroData.cs`, `WeaponData.cs`)

**Asset Files**: lowercase with descriptive names
- Hero: `lucina.asset`, `lucina_sheet.png`, `lucina_portrait.png`
- Map: `simplelevel.asset`, `simplelevel_map.csv`, `simplelevel_start.csv`

**Asset Bundles**: lowercase, no extension (e.g., `heroes/lucina`, `levels/simplelevel`)

---

## Configuration Files

### config.txt
```
master-volume:1.0
music-volume:1.0
sfx-volume:1.0
ui-volume:1.0
```
**Location**: `Data/config.txt`
**Format**: `key:value` pairs

### heroes.json
```json
{
  "heroes": [
    {"bundleName": "lucina", "level": 5, "exp": 25},
    ...
  ]
}
```
**Location**: `Data/heroes.json`
**Format**: JSON array of HeroInfo objects

### levels.json
```json
{
  "levels": [
    {"name": "Tuto", "path": "simplelevel", "difficulty": 0},
    ...
  ]
}
```
**Location**: `StreamingAssets/levels.json`
**Format**: JSON array of LevelInfo objects

---

## Performance Considerations

1. **Asset Bundles**: Cached after first load to avoid I/O overhead
2. **Static MapManager**: Avoids singleton overhead for frequent spatial queries
3. **BFS Pathfinding**: Cached reachable tiles until hero moves
4. **Tilemap**: Unity's built-in grid system for efficient spatial lookups
5. **Object Pooling**: Not implemented (small scale doesn't require it)
6. **Coroutines**: Used for animations to avoid blocking main thread

---

## Extension Points

The architecture makes it easy to extend:

1. **New Heroes**: Create HeroData + sprites, build asset bundle
2. **New Weapons**: Create WeaponData, add to bundle, configure triangle
3. **New Levels**: Create MapData + CSV files, build bundle
4. **New Mounts**: Create MountData, specify movement rules
5. **New Stats**: Create StatData, add AnimationCurve
6. **New Terrain**: Create ZoneData, add to palette
7. **New UI**: Subscribe to DataManager events
8. **New Audio**: Add events in FMOD, trigger from code

---

## Summary

CreaEmblem demonstrates:
- **Modular architecture** with clear separation of concerns
- **Event-driven design** for decoupled systems
- **Data-driven gameplay** using ScriptableObjects
- **Asset bundle system** for runtime content loading
- **Persistent progression** with save/load functionality
- **Professional game flow** with state machines and coroutines

The codebase is clean, maintainable, and extensible—ideal for onboarding new developers or expanding content.
