# CreaEmblem - Quick Start Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Opening the Project](#opening-the-project)
- [Project Overview](#project-overview)
- [Running the Game](#running-the-game)
- [Basic Gameplay](#basic-gameplay)
- [Folder Structure Quick Reference](#folder-structure-quick-reference)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Software
- **Unity 2020.3 LTS or later** (the project was built with Unity 2020.3+)
- **Git** (for version control)
- **Visual Studio** or **Rider** (for C# code editing)
- **FMOD Studio** (optional, only if you want to edit audio)

### System Requirements
- **OS**: Windows, macOS, or Linux
- **RAM**: 8GB minimum (16GB recommended)
- **Storage**: 2GB free space
- **Graphics**: Any GPU with Unity support

---

## Project Setup

### 1. Clone the Repository

```bash
git clone <repository-url>
cd CreaEmblem
```

### 2. Install Unity

If Unity is not installed:

1. Download **Unity Hub** from https://unity.com/download
2. Install Unity Hub
3. Open Unity Hub → **Installs** → **Install Editor**
4. Choose **Unity 2020.3 LTS** (or compatible version)
5. Include the following modules:
   - **Windows/Mac/Linux Build Support** (based on your OS)
   - **Visual Studio Community** (if you don't have an IDE)

---

## Opening the Project

### Method 1: Using Unity Hub (Recommended)

1. Open **Unity Hub**
2. Click **Add** → **Add project from disk**
3. Navigate to the `CreaEmblem` folder
4. Select the folder (should see Assets, ProjectSettings, etc.)
5. Click **Select Folder**
6. Unity Hub will detect the Unity version
7. Click on the project to open it

### Method 2: Direct Open

1. Open Unity
2. Click **Open**
3. Navigate to the `CreaEmblem` folder
4. Click **Open**

**First-time opening may take 5-10 minutes** as Unity imports all assets.

---

## Project Overview

### Main Scenes

CreaEmblem has **2 primary scenes**:

1. **Menu.unity** (`Assets/Scenes/Menu.unity`)
   - Main menu
   - Team selection (choose 4 heroes per player)
   - Level selection (choose from 8 maps)
   - Settings (audio volume controls)

2. **SampleScene.unity** (`Assets/Scenes/SampleScene.unity`)
   - The actual game battlefield
   - Turn-based tactical combat
   - Hero movement and attacks
   - Battle animations

### Key Folders

```
Assets/
├── Scripts/        → All C# game logic
├── Data/           → Hero, weapon, mount, stat definitions
├── Scenes/         → Unity scenes (Menu + Game)
├── Maps/           → Level data (CSV files + MapData)
├── Prefabs/        → Reusable game objects
├── Sprites/        → UI and terrain graphics
├── StreamingAssets/ → Asset bundles (runtime loaded content)
```

---

## Running the Game

### Play from Menu Scene (Recommended)

1. In Unity, open `Assets/Scenes/Menu.unity`
2. Click the **Play** button (▶) at the top
3. The game will start in Play mode

**Expected Flow:**
```
Title Screen → Team Selection → Level Selection → Game
```

### Play from Game Scene (For Testing)

1. Open `Assets/Scenes/SampleScene.unity`
2. Click **Play**
3. This skips the menu and goes directly to the game
4. **Note**: This uses default heroes/level from DataManager

---

## Basic Gameplay

### Controls

**Mouse Only:**
- **Left Click**: Select hero, move hero, attack enemy
- **UI Buttons**: End Turn, Pause Menu

### Turn-Based Flow

1. **Your Turn Starts**
   - Blue highlights appear on walkable tiles
   - Red highlights show attackable enemies

2. **Select a Hero**
   - Click on one of your heroes (blue team)
   - Movement range is highlighted in blue
   - Attack range is highlighted in red (if enemies present)

3. **Move the Hero**
   - Click on a blue tile to preview path
   - Click again to confirm movement
   - Hero moves to the destination

4. **Attack an Enemy (Optional)**
   - After moving (or without moving), click an enemy in range
   - Click again to confirm attack
   - Battle animation plays
   - Both heroes gain experience

5. **End Your Turn**
   - Click the "End Turn" button
   - Opponent's turn begins (red team)

6. **Win Condition**
   - Eliminate all enemy heroes
   - Game over screen appears
   - Return to main menu

### Combat Mechanics

**Weapon Triangle:**
```
Sword beats Axe
Axe beats Spear
Spear beats Sword
Bow has no advantages (but 2-tile range)
```

**Stats:**
- **HP**: Health points
- **ATK**: Attack power
- **DEF**: Physical damage reduction
- **RES**: Magical damage reduction
- **SPD**: If +5 faster, attacks twice

**Experience & Leveling:**
- Gain 50 EXP per attack (attacker or defender)
- Level up when EXP threshold reached
- Stats increase on level up
- Progress saved in `Data/heroes.json`

---

## Folder Structure Quick Reference

### Scripts Organization

```
Assets/Scripts/
├── Game/
│   ├── DataManager.cs       # Global game state, save/load
│   └── Controller.cs        # Player input handling
├── Battle/
│   ├── BattleManager.cs     # Combat orchestration
│   └── BattleCutscene.cs    # Battle animations
├── Heroes/
│   ├── Hero.cs              # Individual character logic
│   ├── HeroManager.cs       # Hero progression persistence
│   └── HeroCanvas.cs        # Hero UI display
├── Map/
│   ├── MapManager.cs        # Tilemap management, pathfinding
│   └── MapData.cs           # Level definition
├── Menu/
│   ├── MenuManager.cs       # Main menu navigation
│   ├── TeamSelectionUI.cs  # Hero team selection
│   ├── LevelSelectionUI.cs # Level selection
│   ├── StatsUI.cs           # Hero stat display
│   ├── GameOverUI.cs        # Victory/defeat screen
│   └── InGameMenu.cs        # Pause menu
├── Utils/
│   ├── AssetBundleManager.cs # Asset bundle loading
│   ├── AudioManager.cs       # FMOD integration
│   ├── DataFile.cs           # Config/save file I/O
│   └── Utils.cs              # Pathfinding helpers
└── Data/
    ├── HeroData.cs          # Hero ScriptableObject
    ├── WeaponData.cs        # Weapon definition
    ├── MountData.cs         # Movement type definition
    ├── StatData.cs          # Stat curve definition
    ├── ZoneData.cs          # Terrain type definition
    └── PaletteData.cs       # Tileset definition
```

### Data Assets

```
Assets/Data/
├── Heroes/
│   ├── Lucina/              # Example hero
│   │   ├── Lucina.asset            # HeroData
│   │   ├── LucinaSheet.asset       # SpriteSheet
│   │   ├── lucina_sheet.png        # Animation texture
│   │   └── lucina_portrait.png     # Icon
│   └── ... (11 more heroes)
├── Weapons/
│   ├── Sword.asset
│   ├── Spear.asset
│   ├── Axe.asset
│   └── Bow.asset
├── Mounts/
│   ├── Feet.asset           # 2-tile movement
│   ├── Horse.asset          # 4-tile movement
│   ├── Pegasus.asset        # 3-tile flying
│   └── Dragon.asset         # 3-tile flying
├── Stats/
│   ├── Hp.asset             # Health curve
│   ├── Atk.asset            # Attack curve
│   ├── Def.asset            # Defense curve
│   ├── Res.asset            # Resistance curve
│   ├── Spd.asset            # Speed curve
│   └── Exp.asset            # Experience curve
└── Zones/
    ├── Ground.asset         # Height 0
    ├── Water.asset          # Height -1 (flyable only)
    ├── Forest.asset         # Height 1
    └── Mountain.asset       # Height 2+
```

### Maps

```
Assets/Maps/
└── Levels/
    ├── SimpleLevel/
    │   ├── SimpleLevel.asset       # MapData ScriptableObject
    │   ├── simplelevel_map.csv     # 26x16 tile grid
    │   └── simplelevel_start.csv   # Hero spawn positions
    └── ... (7 more levels)
```

---

## Common Tasks

### Playing as Different Heroes

1. Run the game (Menu scene)
2. Click **Play**
3. In **Team Selection**, click hero portraits to change selection
4. Click **Start** when ready
5. Choose a level

### Testing a Specific Level

**Method 1: Through Menu**
1. Play from Menu scene
2. Select teams
3. Choose the level you want to test

**Method 2: Direct Load (Code)**
1. Open `Assets/Scripts/Game/DataManager.cs`
2. Find `Start()` method
3. Modify the default level:
```csharp
void Start() {
    // ... existing code ...
    level = AssetBundleManager.LoadAsset<MapData>("levels", "yourlevelname");
}
```

### Viewing Hero Stats in Inspector

1. In Unity, go to `Assets/Data/Heroes/<HeroName>/`
2. Click on the hero's `.asset` file (e.g., `Lucina.asset`)
3. View properties in the **Inspector** panel:
   - Display Name
   - Icon
   - Sprite Sheet
   - Weapon
   - Mount
   - Stats (HP, ATK, DEF, RES, SPD curves)

### Checking Current Hero Levels

**Runtime Check:**
1. Play the game
2. Open `Data/heroes.json` in a text editor
3. View current level and experience for each hero

**Reset Hero Levels:**
Delete `Data/heroes.json` → Will be regenerated with all heroes at Level 1

---

## Troubleshooting

### Issue: Unity Can't Find Scripts

**Symptoms:**
- Missing script references
- Console errors about missing types

**Solution:**
1. Close Unity
2. Delete the `Library` folder in the project root
3. Reopen Unity (will reimport everything)

---

### Issue: Asset Bundles Not Loading

**Symptoms:**
- Black screen on game start
- Console error: "Could not load bundle..."

**Solution:**
1. Check if `Assets/StreamingAssets/` folder exists
2. Rebuild asset bundles:
   - In Unity, go to **Assets** → **Build AssetBundles** (if available)
   - Or check if bundles exist in `AssetBundles/` folder
   - Copy bundles to `Assets/StreamingAssets/`

**Manual Fix:**
If asset bundle menu is missing, the bundles should already be in StreamingAssets. If not:
1. The project may need asset bundle build scripts
2. Check `Assets/Editor/` for build scripts
3. Create bundles for each hero/weapon/mount/stat/zone

---

### Issue: No Audio Playing

**Symptoms:**
- Game runs but no sound effects or music

**Solution:**
1. Check FMOD banks are built:
   - Navigate to `Assets/StreamingAssets/`
   - Ensure `.bank` files exist
2. If missing:
   - Open FMOD Studio project (in `Fmod/` folder)
   - Build banks: **File** → **Build All Platforms**
   - Banks will export to StreamingAssets

---

### Issue: Heroes Not Appearing on Map

**Symptoms:**
- Map loads but no heroes spawn

**Possible Causes:**

**1. No heroes selected in DataManager:**
```csharp
// In DataManager.Start(), check if heroToSpawn is populated
```

**2. Hero start positions missing:**
- Check `*_start.csv` in the level folder
- Should have team indices (0 or 1) for hero spawn locations

**3. Hero bundles not loaded:**
- Check console for bundle loading errors
- Verify hero bundles exist in `StreamingAssets/heroes/`

---

### Issue: Tiles Not Highlighting

**Symptoms:**
- Can't see movement/attack ranges

**Solution:**
1. Check if you're clicking on your own team (Player 1 = blue)
2. Ensure it's your turn (check turn indicator)
3. Verify hero has `canPlay = true`:
   - Select hero in Hierarchy during Play mode
   - Check Inspector → canPlay checkbox

---

### Issue: Battle Animations Not Playing

**Symptoms:**
- Combat happens instantly without animation

**Solution:**
1. Check BattleManager exists in scene
2. Verify BattleCutscene is attached to BattleManager
3. Check animation assets exist in `Assets/Animations/`

---

## Next Steps

### For New Developers

1. **Read the Architecture Documentation**
   - See `ARCHITECTURE.md` for code flow and system design

2. **Read the Developer Guide**
   - See `DEVELOPER_GUIDE.md` for step-by-step modification guides

3. **Explore Key Scripts**
   - Start with `DataManager.cs` (game state)
   - Then `Hero.cs` (character logic)
   - Then `Controller.cs` (input handling)
   - Then `BattleManager.cs` (combat)

### For Content Creators

1. **Learn to Create Heroes**
   - See `DEVELOPER_GUIDE.md` → "Adding a New Hero"

2. **Learn to Create Levels**
   - See `DEVELOPER_GUIDE.md` → "Creating a New Level"

3. **Modify Existing Heroes**
   - Edit stats in `Assets/Data/Heroes/<HeroName>/`

### For Designers

1. **Balance Weapon Triangle**
   - Modify damage multipliers in `Assets/Data/Weapons/`

2. **Adjust Stat Curves**
   - Edit AnimationCurves in `Assets/Data/Stats/`

3. **Design New Levels**
   - Create new CSV files in `Assets/Maps/Levels/`

---

## Additional Resources

### Documentation Files
- **ARCHITECTURE.md** - System design and code flow
- **DEVELOPER_GUIDE.md** - Step-by-step modification guides
- **README.md** - Project overview

### Unity Documentation
- **Tilemaps**: https://docs.unity3d.com/Manual/Tilemap.html
- **ScriptableObjects**: https://docs.unity3d.com/Manual/class-ScriptableObject.html
- **Asset Bundles**: https://docs.unity3d.com/Manual/AssetBundlesIntro.html

### FMOD Documentation
- **FMOD Unity Integration**: https://www.fmod.com/docs/2.02/unity/

---

## Configuration Files

### config.txt
**Location**: `Data/config.txt`

Contains audio volume settings:
```
master-volume:1.0
music-volume:1.0
sfx-volume:1.0
ui-volume:1.0
```
- Values: 0.0 to 1.0
- Created automatically on first run
- Saved on game exit

### heroes.json
**Location**: `Data/heroes.json`

Contains hero progression:
```json
{
  "heroes": [
    {
      "bundleName": "lucina",
      "level": 5,
      "exp": 25
    }
  ]
}
```
- Created automatically on first run
- Saved on game exit
- Persistent across game sessions

### levels.json
**Location**: `Assets/StreamingAssets/levels.json`

Defines available levels:
```json
{
  "levels": [
    {
      "name": "Tuto",
      "path": "simplelevel",
      "difficulty": 0
    }
  ]
}
```
- Edit to add new levels to the level selection menu

---

## Tips for Development

### 1. Use Unity Console
- **Window** → **General** → **Console**
- Watch for errors/warnings during Play mode
- Errors appear in red, warnings in yellow

### 2. Use Debug.Log
Add logging to track game flow:
```csharp
Debug.Log("Hero moved to: " + targetPosition);
```

### 3. Use Breakpoints
In Visual Studio/Rider:
- Click left margin to add breakpoint (red dot)
- Attach Unity debugger
- Game will pause when breakpoint is hit

### 4. Inspector Debugging
During Play mode:
- Select GameObjects in Hierarchy
- Watch values update in real-time in Inspector
- Modify values to test behavior

### 5. Scene View During Play Mode
- Keep Scene view open during Play mode
- See game state visually
- Use **F** to focus on selected object

---

## Quick Reference: Important Files

### Must-Read Scripts
1. `DataManager.cs` - Global game state
2. `Hero.cs` - Character behavior
3. `Controller.cs` - Player input
4. `BattleManager.cs` - Combat system
5. `MapManager.cs` - Tilemap and pathfinding

### Must-Know Assets
1. `HeroData` - Hero definitions (`Assets/Data/Heroes/`)
2. `MapData` - Level definitions (`Assets/Maps/Levels/`)
3. `WeaponData` - Weapon definitions (`Assets/Data/Weapons/`)

### Key Scene Objects
1. **DataManager** - Singleton, persists across scenes
2. **MapManager** - Static, manages tilemap
3. **Controller** - Handles input
4. **BattleManager** - Handles combat

---

## Getting Help

### In-Code Documentation
Most scripts have header comments explaining their purpose.

### Unity Documentation
Press **F1** while selecting a Unity component for documentation.

### Community
- Unity Forums: https://forum.unity.com/
- Unity Learn: https://learn.unity.com/

---

## Summary

You should now be able to:
- ✅ Open the project in Unity
- ✅ Run the game from Menu or Game scene
- ✅ Understand basic gameplay mechanics
- ✅ Navigate the folder structure
- ✅ Locate key scripts and assets
- ✅ Troubleshoot common issues

**Next**: Read `DEVELOPER_GUIDE.md` to learn how to modify the game!
