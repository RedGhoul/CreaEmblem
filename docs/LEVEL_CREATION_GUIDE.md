# Level Creation Guide

This guide will walk you through the process of creating a new level for the game.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Level Structure](#level-structure)
4. [Step-by-Step Guide](#step-by-step-guide)
5. [File Formats](#file-formats)
6. [Adding Level to the Game](#adding-level-to-the-game)
7. [Testing Your Level](#testing-your-level)
8. [Tips and Best Practices](#tips-and-best-practices)

## Overview

Levels in this game are composed of:
- **Map Data**: A 26x16 grid of terrain tiles
- **Spawn Points**: Positions where heroes spawn for each team
- **Unity Assets**: ScriptableObject containers that reference the level data

All levels are compiled into asset bundles for efficient runtime loading.

## Prerequisites

Before creating a new level, you should have:
- Unity Editor installed and project opened
- Basic understanding of CSV file format
- Familiarity with Unity ScriptableObjects
- Knowledge of the game's tile palette system

## Level Structure

Each level consists of the following files:

```
Assets/Maps/Levels/LevelX/
├── LevelX.asset          # Unity ScriptableObject (MapData)
├── LevelX_map.csv        # Terrain tile data (26x16)
└── LevelX_start.csv      # Hero spawn positions (26x16)
```

### Standard Dimensions
- **Map Width**: 26 tiles
- **Map Height**: 16 tiles
- All levels must use these exact dimensions

## Step-by-Step Guide

### Step 1: Create Level Directory

1. Navigate to `Assets/Maps/Levels/`
2. Create a new folder named `LevelX` (where X is your level number)

Example: `Assets/Maps/Levels/Level8/`

### Step 2: Create Map CSV File

Create a file named `LevelX_map.csv` with the following structure:

**Dimensions**: 26 columns × 16 rows

**Tile Values**:
- `-1` = Empty/No tile
- `0` = Walkable terrain (default)
- `1` = Border/Wall tile
- `3` = Forest/Obstacle tile
- Other values may be defined in your PaletteData

**Example** (simplified 26×16 grid):
```csv
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,3,3,0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,0,0,0,0,1
1,0,0,0,3,3,0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,3,3,0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,0,0,0,0,1
1,0,0,0,3,3,0,0,0,0,0,0,0,0,0,0,0,0,0,3,3,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
```

**Tips**:
- Use borders (value `1`) around the edges to prevent units from walking off
- Place obstacles (value `3`) strategically to create interesting tactical situations
- Keep walkable terrain (value `0`) to allow unit movement

### Step 3: Create Start Positions CSV File

Create a file named `LevelX_start.csv` with the following structure:

**Dimensions**: 26 columns × 16 rows

**Spawn Values**:
- `-1` = No spawn point
- `0` = Team 0 (Player) spawn
- `1` = Team 1 (Enemy) spawn

**Example** (simplified 26×16 grid):
```csv
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,0,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,0,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,0,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,0,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1
```

**Tips**:
- Place player spawns (value `0`) on one side of the map
- Place enemy spawns (value `1`) on the opposite side
- You can have multiple spawn points per team (typically 3-4 units per team)
- Ensure spawn points are on walkable terrain in your map CSV

### Step 4: Create Unity MapData Asset

1. In Unity, navigate to `Assets/Maps/Levels/LevelX/`
2. Right-click in the Project window
3. Select `Create > MapData` (or use the appropriate creation menu for ScriptableObjects)
4. Name the asset `LevelX.asset`

### Step 5: Configure the MapData Asset

1. Select the `LevelX.asset` file in Unity
2. In the Inspector, configure the following properties:
   - **Palette**: Assign the PaletteData asset (usually found in `Assets/Data/`)
   - **M_data**: Drag and drop your `LevelX_map.csv` file
   - **M_heroStarts**: Drag and drop your `LevelX_start.csv` file

### Step 6: Build the Asset Bundle

The level needs to be compiled into an asset bundle for runtime loading:

1. Select the `LevelX.asset` file in Unity
2. In the Inspector, look at the bottom panel
3. Set the **AssetBundle** dropdown to `levels/levelX` (lowercase, no spaces)
4. Build the asset bundles for your platform:
   - Go to `Assets > Build AssetBundles` (or the appropriate menu option)
   - This creates files in `AssetBundles/StandaloneWindows/levels/`

**Important**: The asset bundle name should be lowercase and match the path you'll use in `levels.json`.

## Adding Level to the Game

To make your level selectable in the game:

### Step 1: Update levels.json

Edit `Assets/StreamingAssets/levels.json`:

```json
{
  "levels": [
    {
      "name": "Tuto",
      "path": "simplelevel",
      "difficulty": 0
    },
    {
      "name": "First Level",
      "path": "level1",
      "difficulty": 1
    },
    {
      "name": "Your New Level Name",
      "path": "levelX",
      "difficulty": 2
    }
  ]
}
```

**Fields**:
- `name`: Display name shown in the level selection UI
- `path`: Asset bundle path (must match the bundle name you created)
- `difficulty`: Difficulty rating (0 = easiest)

### Step 2: Copy Asset Bundle to StreamingAssets

After building asset bundles:

1. Copy the bundle file from `AssetBundles/StandaloneWindows/levels/levelX`
2. Paste into `Assets/StreamingAssets/levels/levelX`

This ensures the level is included in the build.

## Testing Your Level

### In Unity Editor

1. Open the Game scene
2. Update the level loading code to load your level (or add it to the level selection menu)
3. Press Play
4. Verify:
   - Map tiles render correctly
   - Heroes spawn at the correct positions
   - Terrain is walkable/blocked as expected
   - Teams are assigned correctly

### In Build

1. Build the game for your target platform
2. Launch the game
3. Navigate to level selection
4. Select your new level
5. Test gameplay thoroughly

## File Formats

### Map CSV Format

- **Size**: Must be exactly 26 columns × 16 rows
- **Values**: Integer indices referencing PaletteData tiles
- **Separator**: Comma (`,`)
- **No headers**: First row is tile data, not labels
- **No quotes**: Raw integer values only

**Valid Example**:
```csv
1,1,1,1
0,0,0,0
0,3,3,0
```

**Invalid Example** (has quotes and headers):
```csv
"x","y","tile"
"1","1","1"
```

### Start Positions CSV Format

- **Size**: Must be exactly 26 columns × 16 rows
- **Values**: Team IDs or -1 for empty
  - `-1` = No spawn
  - `0` = Team 0 (Player)
  - `1` = Team 1 (Enemy)
  - Additional teams can use `2`, `3`, etc.
- **Separator**: Comma (`,`)
- **No headers**: First row is spawn data
- **No quotes**: Raw integer values only

## Tips and Best Practices

### Map Design

1. **Use Borders**: Always add wall tiles (value `1`) around the perimeter to prevent units from leaving the map
2. **Balance Teams**: Ensure both teams have equal spawn counts and tactical advantages
3. **Strategic Obstacles**: Place forests/obstacles to create cover and tactical depth
4. **Open Spaces**: Include open areas for movement and tactical maneuvering
5. **Chokepoints**: Consider adding narrow passages for strategic gameplay

### Spawn Points

1. **Separation**: Place opposing teams far enough apart for tactical setup
2. **Safe Spawns**: Ensure spawn points are on walkable terrain, not obstacles
3. **Grouping**: Keep team spawns clustered for coordinated strategy
4. **Equal Numbers**: Balance team sizes (typically 3-4 units per team)
5. **Avoid Overlap**: Never place spawn points on the same tile

### Testing Checklist

- [ ] Map is exactly 26×16 tiles
- [ ] Start positions are exactly 26×16 tiles
- [ ] All tile indices are valid in your PaletteData
- [ ] Both teams have equal spawn counts
- [ ] No spawn points on impassable terrain
- [ ] Map has proper borders to contain units
- [ ] Asset bundle builds without errors
- [ ] Level appears in level selection menu
- [ ] Level loads and renders correctly
- [ ] Heroes spawn at correct positions and teams
- [ ] Gameplay is balanced and fun

### Common Pitfalls

1. **Wrong Dimensions**: Forgetting the 26×16 size requirement causes loading errors
2. **Invalid Tile Indices**: Using tile values not in your PaletteData crashes the renderer
3. **Missing Borders**: Units can walk off the map if borders are incomplete
4. **Spawn on Obstacles**: Heroes stuck on walls/forests if spawn points are misplaced
5. **Bundle Name Mismatch**: Asset bundle name must match the path in `levels.json`
6. **Forgotten StreamingAssets**: Level won't load if bundle isn't copied to StreamingAssets

### Advanced Tips

1. **Prototype in Spreadsheet**: Use Excel/Google Sheets to design your map visually
2. **Symmetrical Design**: Mirror level layout for balanced competitive play
3. **Visual Variety**: Mix different terrain types to create visually interesting maps
4. **Test AI Pathfinding**: Ensure enemy AI can navigate your map effectively
5. **Difficulty Scaling**: Create interesting challenge curves with obstacle placement

## Conclusion

You now have everything you need to create a new level! Start simple with your first level, test thoroughly, and iterate based on gameplay feedback. Happy level designing!

## Further Reading

- See existing levels in `Assets/Maps/Levels/` for reference
- Review `MapData.cs` for data structure details
- Check `MapManager.cs` for level loading logic
- Explore `PaletteData.cs` for tile configuration
