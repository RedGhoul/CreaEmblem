# CreaEmblem - Developer Guide

## Table of Contents
- [Introduction](#introduction)
- [Tutorial 1: Adding a New Hero](#tutorial-1-adding-a-new-hero)
- [Tutorial 2: Creating a New Level](#tutorial-2-creating-a-new-level)
- [Tutorial 3: Modifying Hero Stats and Balance](#tutorial-3-modifying-hero-stats-and-balance)
- [Tutorial 4: Adding a New Weapon Type](#tutorial-4-adding-a-new-weapon-type)
- [Tutorial 5: Creating Custom Terrain Types](#tutorial-5-creating-custom-terrain-types)
- [Tutorial 6: Modifying the UI](#tutorial-6-modifying-the-ui)
- [Tutorial 7: Adding Audio Events](#tutorial-7-adding-audio-events)
- [Tutorial 8: Creating New Movement Types](#tutorial-8-creating-new-movement-types)
- [Tutorial 9: Modifying the Battle System](#tutorial-9-modifying-the-battle-system)
- [Tutorial 10: Adding Game Modes](#tutorial-10-adding-game-modes)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)

---

## Introduction

This guide provides **step-by-step tutorials** for modifying core elements of CreaEmblem. Each tutorial is designed to be followed independently, with clear instructions and code examples.

**Prerequisites:**
- You have read `QUICKSTART.md`
- Unity is installed and the project is open
- Basic understanding of Unity Editor
- Basic C# knowledge (for code modifications)

---

## Tutorial 1: Adding a New Hero

**Goal**: Create a brand new playable hero from scratch.

**Time**: 30-45 minutes

### Step 1: Prepare Assets

Create a new folder for your hero:

```
Assets/Data/Heroes/YourHeroName/
```

**Required Assets:**
1. **Character sprite sheet** (e.g., `yourhero_sheet.png`)
   - 4 rows × 3 columns = 12 frames
   - Row 1: Down-facing animation (3 frames)
   - Row 2: Left-facing animation (3 frames)
   - Row 3: Right-facing animation (3 frames)
   - Row 4: Up-facing animation (3 frames)
   - Recommended size: 96×128 pixels per frame

2. **Portrait icon** (e.g., `yourhero_portrait.png`)
   - Square aspect ratio
   - Recommended size: 256×256 pixels

**Import Assets:**
1. Drag both images into the hero folder
2. Select the sprite sheet in Unity
3. In Inspector, set:
   - **Texture Type**: Sprite (2D and UI)
   - **Sprite Mode**: Multiple
   - **Pixels Per Unit**: 32 (or match existing heroes)
   - **Filter Mode**: Point (no filter) for pixel art
   - **Compression**: None
4. Click **Apply**
5. Click **Sprite Editor**
6. Click **Slice** → **Grid By Cell Count**
7. Set **Columns**: 3, **Rows**: 4
8. Click **Slice**, then **Apply**

---

### Step 2: Create SpriteSheet Asset

1. Right-click in the hero folder
2. **Create** → **Data** → **SpriteSheet**
3. Name it: `YourHeroNameSheet`
4. Select the asset
5. In Inspector:
   - **Texture**: Drag your sprite sheet
   - **Sprite Width**: 3
   - **Sprite Height**: 4
   - **FPS**: 6 (animation speed)

---

### Step 3: Create HeroData Asset

1. Right-click in the hero folder
2. **Create** → **Data** → **Hero**
3. Name it: `YourHeroName`
4. Select the asset
5. In Inspector, configure:

```
Display Name: Your Hero Name
Icon: [Drag portrait sprite]
Sprite Sheet: [Drag YourHeroNameSheet asset]
Weapon: [Select from Assets/Data/Weapons/]
  - Sword, Spear, Axe, or Bow
Mount: [Select from Assets/Data/Mounts/]
  - Feet (2 move), Horse (4 move), Pegasus (3 fly), Dragon (3 fly)
Stats: [Size = 6]
  - Element 0: Hp (with curve)
  - Element 1: Atk (with curve)
  - Element 2: Def (with curve)
  - Element 3: Res (with curve)
  - Element 4: Spd (with curve)
  - Element 5: Exp (with curve)
```

**To set stat curves:**
1. Click the **+** button 6 times to create 6 stat entries
2. For each stat:
   - **Stat**: Drag the corresponding StatData from `Assets/Data/Stats/`
   - **Curve**: Click the curve editor
   - Draw a curve from Level 1-40
   - Example: HP might go from 18 → 45 over 40 levels
   - Use existing heroes as reference

---

### Step 4: Build Asset Bundle

**Option A: Manual Asset Bundle Creation** (if Editor script exists)

1. Select the `YourHeroName` asset
2. At the bottom of Inspector, find **AssetBundle**
3. Click dropdown → **New...**
4. Name it: `heroes/yourheroname` (lowercase, no spaces)
5. Click to assign
6. In Unity menu: **Assets** → **Build AssetBundles**
7. Bundles are created in `AssetBundles/` folder
8. Copy `AssetBundles/heroes/yourheroname` to `Assets/StreamingAssets/heroes/`

**Option B: Manual Creation** (if no build script)

Create the asset bundle through code or use existing hero bundles as templates. The bundle should contain:
- HeroData ScriptableObject
- SpriteSheet asset
- All referenced sprites

---

### Step 5: Add Hero to Game

**Method 1: Add to heroes.json (Persistent)**

1. Play the game once (to generate `Data/heroes.json`)
2. Exit Unity Play mode
3. Open `Data/heroes.json` in a text editor
4. Add your hero:

```json
{
  "heroes": [
    {
      "bundleName": "yourheroname",
      "level": 1,
      "exp": 0
    },
    ... other heroes
  ]
}
```

5. Save the file
6. Hero will now appear in team selection

**Method 2: Hardcode for Testing**

Edit `Assets/Scripts/Game/DataManager.cs`:

```csharp
void Start() {
    // ... existing code ...

    // Add your hero to default team
    heroToSpawn.Add(new HeroTeam {
        team = 0,
        info = new HeroInfo {
            bundleName = "yourheroname",
            level = 1,
            exp = 0
        }
    });
}
```

---

### Step 6: Test Your Hero

1. Play the game from Menu scene
2. Go to Team Selection
3. Your hero should appear in the selection grid
4. Select your hero for Player 1 team
5. Choose a level and play
6. Verify:
   - Hero appears on map
   - Sprite animates correctly
   - Stats are correct (check StatsUI panel)
   - Movement range matches mount type
   - Attack range matches weapon type
   - Combat works correctly

---

### Troubleshooting

**Hero doesn't appear in team selection:**
- Check `heroes.json` has correct bundleName
- Verify asset bundle exists in `StreamingAssets/heroes/`
- Check console for loading errors

**Hero appears as blank/missing sprite:**
- Verify SpriteSheet asset is correctly configured
- Check sprite sheet texture is properly sliced
- Ensure HeroData references the SpriteSheet

**Hero stats are wrong:**
- Check stat curves in HeroData
- Verify level is set correctly in HeroInfo
- Use Debug.Log in Hero.cs to print calculated stats

---

## Tutorial 2: Creating a New Level

**Goal**: Design and implement a custom tactical map.

**Time**: 45-60 minutes

### Step 1: Plan Your Level

**Design Considerations:**
- **Size**: Fixed at 26×16 tiles
- **Teams**: 2 players with 1-4 heroes each
- **Terrain**: Ground, Water, Forest, Mountain
- **Balance**: Ensure fair starting positions

**Sketch your map** on paper or use a spreadsheet (26 columns × 16 rows).

---

### Step 2: Create Level Folder

```
Assets/Maps/Levels/YourLevelName/
```

---

### Step 3: Create Map CSV File

1. Create a new file: `yourlevelname_map.csv`
2. Open in a text editor or spreadsheet program
3. Create a 26×16 grid of numbers representing tile types:

**Tile Palette Indices** (based on your palette):
```
0 = Ground
1 = Water
2 = Forest
3 = Mountain
... (check your PaletteData for exact indices)
```

**Example CSV** (simplified 8×4 for demonstration):
```csv
0,0,0,1,1,0,0,0
0,0,2,1,1,2,0,0
0,2,2,0,0,2,2,0
0,0,0,0,0,0,0,0
```

**Full map**: Create 16 rows of 26 comma-separated values.

**Tips:**
- Use 0 (ground) for most tiles
- Add 1 (water) for obstacles (only flyers can cross)
- Use 2 (forest) for cover (aesthetic)
- Use 3 (mountain) for barriers

---

### Step 4: Create Hero Start Positions CSV

1. Create a new file: `yourlevelname_start.csv`
2. Create a 26×16 grid representing hero spawn points:

**Values:**
```
-1 = No hero spawns here
 0 = Player 1 hero can spawn here
 1 = Player 2 hero can spawn here
```

**Example CSV** (simplified 8×4):
```csv
-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1
-1,-1,-1,-1,-1,-1,-1,-1
 0, 0, 0, 0, 1, 1, 1, 1
```

This creates:
- 4 spawn points for Player 1 on the left bottom
- 4 spawn points for Player 2 on the right bottom

**Full map**: Create 16 rows of 26 comma-separated values.

**Tips:**
- Place spawns symmetrically for balance
- Match number of spawns to expected team size (usually 4 per team)
- Don't place spawns on impassable terrain (water, unless flyers)
- Leave space between teams for tactical gameplay

---

### Step 5: Create MapData Asset

1. Right-click in your level folder
2. **Create** → **Data** → **Map**
3. Name it: `YourLevelName`
4. Select the asset
5. In Inspector:

```
Palette: [Drag PaletteData from Assets/Data/Palettes/]
  - Use "DefaultPalette" or create a custom one
Width: 26
Height: 16
```

6. **Load Map Data**:
   - You'll need to populate the `data` array manually or via script
   - See existing MapData assets for reference

**Loading CSV in Code** (if script exists):

If there's a MapData editor script, you might have buttons to:
- **Load Map CSV**: Imports `yourlevelname_map.csv`
- **Load Start CSV**: Imports `yourlevelname_start.csv`

**Manual Approach**:

Create a simple Unity Editor script to load CSV:

```csharp
// Put this in Assets/Scripts/Editor/MapDataEditor.cs
using UnityEngine;
using UnityEditor;
using System.IO;

public class MapDataEditor : MonoBehaviour {
    [MenuItem("Tools/Load Map CSV")]
    static void LoadMapCSV() {
        MapData map = Selection.activeObject as MapData;
        if (map == null) return;

        string path = EditorUtility.OpenFilePanel("Load Map CSV", "", "csv");
        if (string.IsNullOrEmpty(path)) return;

        string[] lines = File.ReadAllLines(path);
        map.data = new int[MapData.height, MapData.width];

        for (int y = 0; y < lines.Length && y < MapData.height; y++) {
            string[] values = lines[y].Split(',');
            for (int x = 0; x < values.Length && x < MapData.width; x++) {
                map.data[y, x] = int.Parse(values[x].Trim());
            }
        }

        EditorUtility.SetDirty(map);
        Debug.Log("Map CSV loaded successfully");
    }
}
```

Usage:
1. Select your MapData asset
2. **Tools** → **Load Map CSV**
3. Select `yourlevelname_map.csv`
4. Repeat for start positions

---

### Step 6: Build Level Asset Bundle

1. Select your `YourLevelName` MapData asset
2. In Inspector (bottom), set **AssetBundle**: `levels/yourlevelname`
3. **Assets** → **Build AssetBundles**
4. Copy `AssetBundles/levels/yourlevelname` to `Assets/StreamingAssets/levels/`

---

### Step 7: Add Level to Game

1. Open `Assets/StreamingAssets/levels.json`
2. Add your level:

```json
{
  "levels": [
    {
      "name": "Your Level Name",
      "path": "yourlevelname",
      "difficulty": 1
    },
    ... other levels
  ]
}
```

3. Save the file

---

### Step 8: Test Your Level

1. Play the game from Menu scene
2. Go through Team Selection
3. Your level should appear in Level Selection
4. Click your level to play
5. Verify:
   - Map tiles render correctly
   - Heroes spawn at correct positions
   - Terrain is walkable/unwalkable as intended
   - Map is balanced and playable

---

### Advanced: Visual Level Editing

For a more visual approach, use Unity's Tilemap:

1. In SampleScene, select the **Tilemap** GameObject
2. Open **Tile Palette** window (**Window** → **2D** → **Tile Palette**)
3. Paint tiles directly on the grid
4. Use a script to export the Tilemap to CSV

**Export Script Example:**

```csharp
// Assets/Scripts/Editor/TilemapToCSV.cs
using UnityEngine;
using UnityEditor;
using UnityEngine.Tilemaps;
using System.IO;
using System.Text;

public class TilemapToCSV : MonoBehaviour {
    [MenuItem("Tools/Export Tilemap to CSV")]
    static void ExportTilemap() {
        Tilemap tilemap = FindObjectOfType<Tilemap>();
        if (tilemap == null) return;

        string path = EditorUtility.SaveFilePanel("Export Map CSV", "", "map.csv", "csv");
        if (string.IsNullOrEmpty(path)) return;

        StringBuilder csv = new StringBuilder();

        for (int y = MapData.height - 1; y >= 0; y--) {
            StringBuilder row = new StringBuilder();
            for (int x = 0; x < MapData.width; x++) {
                Vector3Int pos = new Vector3Int(x, y, 0);
                TileBase tile = tilemap.GetTile(pos);

                // Map tile to palette index (customize based on your palette)
                int index = GetPaletteIndex(tile);

                row.Append(index);
                if (x < MapData.width - 1) row.Append(",");
            }
            csv.AppendLine(row.ToString());
        }

        File.WriteAllText(path, csv.ToString());
        Debug.Log("Tilemap exported to: " + path);
    }

    static int GetPaletteIndex(TileBase tile) {
        if (tile == null) return 0;
        // Map tile types to indices based on your palette
        // This is a simplified example
        return 0;
    }
}
```

---

## Tutorial 3: Modifying Hero Stats and Balance

**Goal**: Adjust hero power levels and progression curves.

**Time**: 15-30 minutes

### Understanding Stat Curves

Each hero has 6 stats, each with an **AnimationCurve** that defines growth from Level 1-40.

**Stat Types:**
1. **HP**: Health points (typical range: 18-45)
2. **ATK**: Attack power (typical range: 7-18)
3. **DEF**: Physical defense (typical range: 6-15)
4. **RES**: Magical resistance (typical range: 6-15)
5. **SPD**: Speed (typical range: 5-12)
6. **EXP**: Experience required for level up (typical: 100 per level)

---

### Step 1: Open Hero Asset

1. Navigate to `Assets/Data/Heroes/<HeroName>/`
2. Select the hero's `.asset` file
3. View the **Stats** list in Inspector

---

### Step 2: Edit Stat Curves

**For each stat:**

1. Click the **Curve** field
2. Curve Editor opens
3. **Horizontal axis**: Level (1-40)
4. **Vertical axis**: Stat value

**Editing the Curve:**

- **Add Key**: Click on the curve line
- **Move Key**: Drag with mouse
- **Delete Key**: Right-click key → Delete
- **Tangent**: Right-click key → change curve type

**Example HP Curve:**
```
Level  1: 18 HP
Level 10: 25 HP
Level 20: 33 HP
Level 30: 39 HP
Level 40: 45 HP
```

**Tips:**
- **HP**: Increase steadily, 0.5-1.0 per level
- **ATK**: Increase slowly, 0.2-0.4 per level
- **DEF/RES**: Increase slowly, 0.2-0.3 per level
- **SPD**: Increase very slowly, 0.1-0.2 per level
- **EXP**: Usually flat at 100

---

### Step 3: Balance Testing

**Test Framework:**

1. Set hero to specific level in `heroes.json`:
```json
{"bundleName": "testhero", "level": 20, "exp": 0}
```

2. Play the game
3. Check stats in StatsUI panel
4. Test combat against enemies

**Balance Targets:**

- **Early Game** (Level 1-10): Weaker, needs careful play
- **Mid Game** (Level 11-25): Balanced, good power
- **Late Game** (Level 26-40): Strong, dominates

**Damage Formula Reference:**
```
Physical: damage = ATK + weaponBonus - enemy.DEF
Magical:  damage = ATK + weaponBonus - enemy.RES
```

**Weapon Triangle Bonus**: ±20% of ATK

---

### Step 4: Quick Balance Changes

**Make a hero stronger:**
- Increase ATK curve by 2-3 points across all levels
- Increase HP curve by 3-5 points
- Leave DEF/RES/SPD unchanged

**Make a hero tankier:**
- Increase HP by 5-8 points
- Increase DEF/RES by 2-3 points
- Reduce ATK slightly for balance

**Make a hero faster:**
- Increase SPD by 2-3 points
- Remember: +5 SPD advantage = double attack

**Make a hero gain levels faster:**
- Reduce EXP curve from 100 to 75-80

---

### Step 5: Rebuild Asset Bundle

After editing:
1. Save the hero asset
2. Rebuild the hero's asset bundle:
   - **Assets** → **Build AssetBundles**
3. Copy updated bundle to `StreamingAssets/heroes/`
4. Or, if not using bundles, changes apply immediately

---

## Tutorial 4: Adding a New Weapon Type

**Goal**: Create a unique weapon with custom properties.

**Time**: 20-30 minutes

### Step 1: Create WeaponData Asset

1. Right-click in `Assets/Data/Weapons/`
2. **Create** → **Data** → **Weapon**
3. Name it: `YourWeaponName` (e.g., "Lance", "Tome", "Dagger")

---

### Step 2: Configure Weapon Properties

Select the weapon asset and configure:

```
Display Name: Your Weapon Name
Range: 1 or 2
  - 1 = Melee (adjacent tiles)
  - 2 = Ranged (2 tiles away)
Damage Type: Physical or Magical
  - Physical: Reduced by DEF
  - Magical: Reduced by RES
Weaknesses: [List of WeaponData]
  - This weapon is weak against these weapons (-20% damage)
Resistances: [List of WeaponData]
  - This weapon is strong against these weapons (+20% damage)
```

**Example - Magic Tome:**
```
Display Name: Tome
Range: 2
Damage Type: Magical
Weaknesses: [Bow]
Resistances: [Sword]
```

**Example - Dagger:**
```
Display Name: Dagger
Range: 2
Damage Type: Physical
Weaknesses: [Sword]
Resistances: [Axe]
```

---

### Step 3: Configure Weapon Triangle

**Current Triangle:**
```
Sword > Axe
Axe > Spear
Spear > Sword
Bow: No relationships
```

**To add your weapon to the triangle:**

**Option 1: Create New Triangle**
```
Tome > Sword (magic beats physical light weapon)
Sword > Dagger (sword beats small blade)
Dagger > Tome (fast blade disrupts casting)
```

1. Select **Tome** weapon
2. **Resistances**: Add "Sword"
3. **Weaknesses**: Add "Dagger"

4. Select **Sword** weapon
5. **Weaknesses**: Add "Tome"
6. **Resistances**: Add "Dagger"

7. Select **Dagger** weapon
8. **Resistances**: Add "Tome"
9. **Weaknesses**: Add "Sword"

**Option 2: Independent Weapon**
- Leave **Weaknesses** and **Resistances** empty
- No triangle advantages/disadvantages

---

### Step 4: Create Weapon Icon

1. Create or import a weapon icon sprite (e.g., `tome_icon.png`)
2. Place in `Assets/Sprites/Weapons/`
3. In Unity, select the sprite
4. Set **Texture Type**: Sprite (2D and UI)
5. **Apply**

---

### Step 5: Add Weapon to Asset Bundle

1. Select weapon asset
2. Set **AssetBundle**: `weapons/yourweaponname`
3. **Assets** → **Build AssetBundles**
4. Copy to `StreamingAssets/weapons/`

---

### Step 6: Update DataManager to Load Weapon

Edit `Assets/Scripts/Game/DataManager.cs`:

```csharp
void Start() {
    // ... existing weapon loading ...
    AssetBundleManager.LoadAsset<WeaponData>("weapons", "yourweaponname");
}
```

Or, if weapons are auto-loaded, just ensure the bundle exists in StreamingAssets.

---

### Step 7: Assign Weapon to Heroes

1. Open hero assets in `Assets/Data/Heroes/`
2. Change **Weapon** field to your new weapon
3. Save and rebuild hero asset bundles

---

### Step 8: Test Weapon

1. Play the game with a hero using the new weapon
2. Verify:
   - Attack range matches weapon range
   - Damage type is correct (check StatsUI)
   - Weapon triangle bonuses apply correctly
   - Combat animations play

---

### Advanced: Custom Weapon Behavior

To add special weapon effects (e.g., lifesteal, poison), modify `Hero.GetDamage()` in `Assets/Scripts/Heroes/Hero.cs`:

```csharp
public int GetDamage(Hero attacker) {
    // ... existing damage calculation ...

    // Add custom effect
    if (attacker.data.weapon.displayName == "Lifesteal Sword") {
        int healing = damage / 2;
        attacker.life = Mathf.Min(attacker.life + healing, attacker.maxLife);
        Debug.Log(attacker.data.displayName + " healed " + healing + " HP");
    }

    return damage;
}
```

---

## Tutorial 5: Creating Custom Terrain Types

**Goal**: Add new terrain with unique properties.

**Time**: 20-30 minutes

### Understanding Terrain System

**Terrain Properties:**
- **Height**: Determines walkability
  - `-1`: Water (flyers only)
  - `0`: Ground (all units)
  - `1`: Forest (all units)
  - `2+`: Mountain (restricted)
- **Visual**: Sprite used for rendering

---

### Step 1: Create ZoneData Asset

1. Right-click in `Assets/Data/Zones/`
2. **Create** → **Data** → **Zone**
3. Name it: `YourTerrainName` (e.g., "Lava", "Ice", "Bridge")

---

### Step 2: Configure Terrain Properties

Select the zone asset:

```
Display Name: Your Terrain Name
Height: -1 to 10
  - -1: Only flyers (like Water)
  -  0: All units (like Ground)
  -  1: All units (like Forest)
  -  2: Some units (like Mountain)
Sprite Sheet: [Your terrain texture]
```

**Example - Lava:**
```
Display Name: Lava
Height: -1 (only flyers can cross)
Sprite Sheet: [Orange/red lava texture]
```

**Example - Bridge:**
```
Display Name: Bridge
Height: 0 (all units can cross, even over water)
Sprite Sheet: [Bridge texture]
```

---

### Step 3: Create Terrain Sprites

1. Create or import terrain tile sprites
2. Place in `Assets/Sprites/Terrain/`
3. Configure as sprites (see Tutorial 1 for import settings)
4. If using tilesets, slice into tiles

---

### Step 4: Create Tile Asset

1. Right-click in `Assets/Maps/Tiles/`
2. **Create** → **2D** → **Tiles** → **Tile**
3. Name it: `YourTerrainTile`
4. Select the tile
5. **Sprite**: Drag your terrain sprite
6. **Collider Type**: None

---

### Step 5: Add to Tile Palette

1. Open **Tile Palette** window (**Window** → **2D** → **Tile Palette**)
2. Select your active palette (or create new)
3. Drag `YourTerrainTile` into the palette
4. Tile is now paintable

---

### Step 6: Update PaletteData

1. Navigate to `Assets/Data/Palettes/`
2. Select your `PaletteData` asset
3. In Inspector, **Tile Infos** list:
   - Increase **Size** by 1
   - New element appears at bottom
4. Configure new element:
   ```
   Tile: [Drag YourTerrainTile]
   Zone Data: [Drag YourTerrainName ZoneData]
   ```

**Remember the index** of this new tile (e.g., if it's the 5th element, index = 4).

---

### Step 7: Use Terrain in Levels

When creating map CSVs (see Tutorial 2), use the new tile's palette index:

```csv
0,0,0,0,0
0,4,4,4,0  ← Your new terrain (index 4)
0,0,0,0,0
```

---

### Step 8: Build Asset Bundle

1. Select your ZoneData asset
2. Set **AssetBundle**: `terrains/zones/yourterrainname`
3. Select your PaletteData (if modified)
4. Set **AssetBundle**: `terrains/palettes/yourpalettename`
5. **Assets** → **Build AssetBundles**
6. Copy to `StreamingAssets/terrains/zones/` and `StreamingAssets/terrains/palettes/`

---

### Step 9: Update DataManager

Edit `Assets/Scripts/Game/DataManager.cs` to load your terrain:

```csharp
void Start() {
    // Load terrain zones
    AssetBundleManager.LoadAsset<ZoneData>("terrains/zones", "yourterrainname");

    // ... existing code ...
}
```

---

### Step 10: Test Terrain

1. Create or modify a level to use your new terrain
2. Play the game
3. Verify:
   - Terrain renders correctly
   - Walkability matches height setting
   - Different mount types interact correctly

---

### Advanced: Terrain Effects

To add terrain effects (e.g., damage, healing), modify `Hero.cs`:

```csharp
void Update() {
    // ... existing code ...

    // Check terrain at hero position
    ZoneData zone = MapManager.GetZoneAtPosition(gridPosition);

    if (zone.displayName == "Lava") {
        // Take 5 damage per turn on lava
        TakeDamage(5);
        Debug.Log(data.displayName + " burned by lava!");
    }

    if (zone.displayName == "Healing Tile") {
        // Heal 10 HP per turn
        life = Mathf.Min(life + 10, maxLife);
    }
}
```

---

## Tutorial 6: Modifying the UI

**Goal**: Customize UI elements and layouts.

**Time**: 20-40 minutes

### Understanding UI Structure

**UI Scripts Location**: `Assets/Scripts/Menu/`

**Key UI Components:**
- **MenuManager**: Main menu navigation
- **TeamSelectionUI**: Hero team composition
- **LevelSelectionUI**: Level selection grid
- **StatsUI**: In-game hero stats panel
- **GameOverUI**: Victory/defeat screen
- **InGameMenu**: Pause menu
- **ExperienceUI**: Level up animations
- **HeroCanvas**: In-game hero health bars

---

### Example 1: Change End Turn Button Text

**Goal**: Rename "End Turn" to "Finish Turn"

1. Open `Assets/Scenes/SampleScene.unity`
2. In Hierarchy, navigate to:
   ```
   Canvas → EndTurnButton → Text
   ```
3. Select **Text** component
4. In Inspector, change **Text** field to: "Finish Turn"
5. Adjust **Font Size** if needed
6. Save scene

---

### Example 2: Modify Hero Stats Panel

**Goal**: Show current EXP in stats panel

1. Open `Assets/Scripts/Menu/StatsUI.cs`
2. Add a new Text field:

```csharp
public class StatsUI : MonoBehaviour {
    // ... existing fields ...
    public Text expText; // Add this line

    public void ShowHero(Hero hero) {
        // ... existing code ...

        // Add EXP display
        if (expText != null) {
            expText.text = "EXP: " + hero.info.exp;
        }
    }
}
```

3. Open `Assets/Scenes/SampleScene.unity`
4. Find **StatsUI** GameObject in Hierarchy
5. Duplicate one of the existing stat Text objects
6. Rename it to "ExpText"
7. Position it below other stats
8. Select **StatsUI** GameObject
9. In Inspector, drag **ExpText** into the new **Exp Text** field
10. Save scene and test

---

### Example 3: Add a Victory Sound

**Goal**: Play sound effect when game ends

1. Open FMOD Studio (if available)
2. Create a new event: `Game/Victory`
3. Add audio file to event
4. Build banks
5. Open `Assets/Scripts/Menu/GameOverUI.cs`:

```csharp
public void ShowWinner(int team) {
    // ... existing code ...

    // Play victory sound
    if (DataManager.instance.audio != null) {
        DataManager.instance.audio.PlayEvent("Game/Victory");
    }
}
```

6. Save and test

---

### Example 4: Change Team Colors

**Goal**: Change Player 1 from blue to green

1. Open `Assets/Scripts/Heroes/HeroCanvas.cs`
2. Find the `SetTeam()` method:

```csharp
void SetTeam(int team) {
    if (team == 0) {
        // Change from blue to green
        image.color = new Color(0f, 1f, 0f); // RGB: Green
    } else {
        image.color = new Color(1f, 0f, 0f); // RGB: Red
    }
}
```

3. Save and test
4. All Player 1 heroes will now have green health bars

---

### Example 5: Add a Turn Counter

**Goal**: Display current turn number

**Step 1: Add Turn Counting to DataManager**

Open `Assets/Scripts/Game/DataManager.cs`:

```csharp
public class DataManager : MonoBehaviour {
    // ... existing fields ...
    public int turnNumber = 1; // Add this

    public void EndTurn() {
        // ... existing code ...

        if (teamPlaying == 1) {
            // Both players went, increment turn
            turnNumber++;
        }

        // ... rest of existing code ...
    }

    public void StartGame() {
        turnNumber = 1; // Reset on new game
        // ... existing code ...
    }
}
```

**Step 2: Add UI Text**

1. Open `Assets/Scenes/SampleScene.unity`
2. Right-click **Canvas** in Hierarchy
3. **UI** → **Text**
4. Name it "TurnCounter"
5. Position at top-center of screen
6. Set text: "Turn: 1"
7. Adjust font size and color

**Step 3: Create UI Controller Script**

Create `Assets/Scripts/Menu/TurnCounterUI.cs`:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class TurnCounterUI : MonoBehaviour {
    public Text turnText;

    void Start() {
        DataManager.instance.onStartTurn.AddListener(UpdateTurnDisplay);
        UpdateTurnDisplay();
    }

    void UpdateTurnDisplay() {
        if (turnText != null) {
            turnText.text = "Turn: " + DataManager.instance.turnNumber;
        }
    }
}
```

**Step 4: Connect Script to UI**

1. Select **TurnCounter** GameObject
2. **Add Component** → **Turn Counter UI**
3. Drag **TurnCounter**'s Text component into **Turn Text** field
4. Save and test

---

### Example 6: Customize Team Selection Grid

**Goal**: Show 16 heroes instead of 12

1. Open `Assets/Scripts/Menu/TeamSelectionUI.cs`
2. Find hero grid generation code
3. Increase grid size from 3×4 to 4×4:

```csharp
void Start() {
    // ... existing code ...

    // Change grid layout
    int columns = 4; // Was 3
    int rows = 4;    // Was 4

    // Generate hero buttons
    for (int i = 0; i < columns * rows; i++) {
        // ... create buttons ...
    }
}
```

4. Adjust UI layout in scene to accommodate extra column
5. Add more heroes to `heroes.json`

---

### UI Best Practices

**1. Use Anchors Properly**
- Set RectTransform anchors for responsive design
- Top-left UI: Anchor to top-left
- Bottom-right UI: Anchor to bottom-right
- Center UI: Anchor to center

**2. Use Canvas Scaler**
- Select Canvas in Hierarchy
- Canvas Scaler component → **Scale With Screen Size**
- Reference Resolution: 1920×1080 (or your target)

**3. Use UI Events**
- Subscribe to DataManager events for reactive UI
- Example: `onStartTurn`, `onEndGame`, `onHeroDeath`

**4. Test Different Resolutions**
- Use Unity Game view to test various screen sizes
- **Game** view → Dropdown → Add custom resolutions

---

## Tutorial 7: Adding Audio Events

**Goal**: Integrate new sound effects and music.

**Time**: 30-45 minutes

**Prerequisite**: FMOD Studio installed

### Step 1: Open FMOD Project

1. Navigate to `Fmod/` folder in project root
2. Open the `.fspro` file in FMOD Studio

---

### Step 2: Create New Event

1. In FMOD Studio, right-click **Events** folder in Browser
2. **New Event** → **2D**
3. Name it: `Game/YourSoundEvent` (e.g., "Game/Heal")
4. Event Editor opens

---

### Step 3: Add Audio to Event

1. Drag audio file into Event Editor timeline
2. Or right-click timeline → **Add Sound**
3. Configure:
   - **Loop**: Enable if looping sound
   - **Volume**: Adjust as needed
   - **Pitch**: Randomize for variation

**For 3D positional audio:**
- Create **3D Event** instead
- Add **Spatializer** effect

---

### Step 4: Set Event Parameters (Optional)

For dynamic audio (e.g., music that changes with intensity):

1. Right-click in **Parameters** section → **Add Parameter**
2. Name: "Intensity" (or similar)
3. Set **Range**: 0.0 to 1.0
4. In timeline, add multiple audio segments
5. Use parameter value to crossfade between segments

---

### Step 5: Build FMOD Banks

1. In FMOD Studio, **File** → **Build All Platforms**
2. Banks are exported to:
   ```
   Assets/StreamingAssets/
   ├── Master.bank
   ├── Master.strings.bank
   └── YourBank.bank
   ```

---

### Step 6: Trigger Event from Code

**Simple Playback:**

Open the script where you want to play audio (e.g., `Hero.cs`):

```csharp
using FMODUnity;

public class Hero : MonoBehaviour {
    public void Heal(int amount) {
        life += amount;

        // Play heal sound
        RuntimeManager.PlayOneShot("event:/Game/Heal");
    }
}
```

**With Parameters:**

```csharp
public void PlayDynamicMusic(float intensity) {
    FMOD.Studio.EventInstance music = RuntimeManager.CreateInstance("event:/Musics/BGM");
    music.setParameterByName("Intensity", intensity);
    music.start();
}
```

**3D Positional Audio:**

```csharp
public void PlayAt3DPosition(Vector3 position) {
    RuntimeManager.PlayOneShot("event:/Game/Explosion", position);
}
```

---

### Step 7: Add Audio Manager Integration

For centralized audio control, use `AudioManager.cs`:

1. Open `Assets/Scripts/Utils/AudioManager.cs`
2. Add new event method:

```csharp
public void PlayHealSound() {
    PlayEvent("Game/Heal");
}
```

3. Call from gameplay code:

```csharp
DataManager.instance.audio.PlayHealSound();
```

---

### Step 8: Volume Control

**Existing Volume Controls:**

The game already has VCA (Virtual Channel Attributes) for:
- **Master**: Overall volume
- **Music**: Background music
- **SFX**: Sound effects
- **UI**: UI sounds

**Assign Event to VCA:**

1. In FMOD Studio, select your event
2. In **Mixer** section, set **Output**: `vca:/SFX` (or appropriate VCA)
3. Event will now respect volume settings

**Code Integration:**

Volume is already managed in `DataManager.cs` and saved to `config.txt`.

---

### Step 9: Test Audio

1. Play the game
2. Trigger the action that plays your sound
3. Verify:
   - Sound plays at correct moment
   - Volume is appropriate
   - No clipping or distortion
   - 3D positioning works (if applicable)

---

### Audio Best Practices

**1. Use VCAs for Volume Categories**
- Group similar sounds (music, SFX, UI)
- Allows user control per category

**2. Randomize Parameters**
- Add slight pitch variation: ±5%
- Prevents repetitive sounds

**3. Use Snapshots for Transitions**
- Create snapshots for different game states
- Smoothly transition between states

**4. Optimize Event Count**
- Don't create hundreds of instances
- Reuse events when possible
- Stop events when no longer needed

**5. Test on Target Platform**
- Audio may sound different on different devices
- Test volume levels on actual hardware

---

## Tutorial 8: Creating New Movement Types

**Goal**: Add custom movement behaviors (e.g., teleporter, swimmer).

**Time**: 25-35 minutes

### Step 1: Create MountData Asset

1. Right-click in `Assets/Data/Mounts/`
2. **Create** → **Data** → **Mount**
3. Name it: `YourMountName` (e.g., "Wyvern", "Boat", "Teleporter")

---

### Step 2: Configure Movement Properties

Select the mount asset:

```
Display Name: Your Mount Name
Move Distance: 1-6 tiles per turn
Min Walkable Height: -1 to 2
Max Walkable Height: -1 to 2
```

**Examples:**

**Wyvern (Super Flyer):**
```
Display Name: Wyvern
Move Distance: 5
Min Walkable Height: -1
Max Walkable Height: 10
```
(Can cross any terrain, long range)

**Boat (Water Mover):**
```
Display Name: Boat
Move Distance: 3
Min Walkable Height: -1
Max Walkable Height: -1
```
(Only moves on water)

**Teleporter (Short-range Teleport):**
```
Display Name: Teleporter
Move Distance: 2
Min Walkable Height: -1
Max Walkable Height: 10
```
(Ignores obstacles, but short range)

---

### Step 3: Build Asset Bundle

1. Select mount asset
2. Set **AssetBundle**: `mounts/yourmountname`
3. **Assets** → **Build AssetBundles**
4. Copy to `StreamingAssets/mounts/`

---

### Step 4: Update DataManager to Load Mount

Edit `Assets/Scripts/Game/DataManager.cs`:

```csharp
void Start() {
    // ... existing mount loading ...
    AssetBundleManager.LoadAsset<MountData>("mounts", "yourmountname");
}
```

---

### Step 5: Assign Mount to Heroes

1. Open hero assets
2. Change **Mount** field to your new mount
3. Rebuild hero asset bundles

---

### Step 6: Test Movement

1. Play the game with a hero using the new mount
2. Verify:
   - Movement range is correct
   - Terrain walkability matches settings
   - Hero can/cannot cross obstacles as intended

---

### Advanced: Custom Movement Logic

For complex movement (e.g., teleportation, phasing), modify `Hero.GetReachableTiles()`:

**Example - Teleporter (Ignores Obstacles):**

Open `Assets/Scripts/Heroes/Hero.cs`:

```csharp
public List<Vector3Int> GetReachableTiles() {
    List<Vector3Int> reachable = new List<Vector3Int>();

    // Special logic for Teleporter mount
    if (data.mount.displayName == "Teleporter") {
        // Can teleport to any tile within range, ignoring obstacles
        int range = data.mount.moveDistance;

        for (int x = gridPosition.x - range; x <= gridPosition.x + range; x++) {
            for (int y = gridPosition.y - range; y <= gridPosition.y + range; y++) {
                Vector3Int tile = new Vector3Int(x, y, 0);
                int distance = Utils.GetDistance(gridPosition, tile);

                if (distance <= range && distance > 0) {
                    // Check if tile is in bounds
                    if (tile.x >= 0 && tile.x < MapData.width &&
                        tile.y >= 0 && tile.y < MapData.height) {
                        // Check if tile is occupied
                        if (MapManager.GetHeroAtTile(tile) == null) {
                            reachable.Add(tile);
                        }
                    }
                }
            }
        }

        return reachable;
    }

    // ... existing BFS pathfinding for normal mounts ...
}
```

**Example - Phasing Mount (Walks Through Enemies):**

```csharp
public List<Vector3Int> GetReachableTiles() {
    // ... existing BFS setup ...

    while (queue.Count > 0) {
        // ... existing code ...

        // Check neighbors
        foreach (Vector3Int neighbor in neighbors) {
            // ... existing checks ...

            // For Phasing mount, ignore enemy occupation
            Hero occupant = MapManager.GetHeroAtTile(neighbor);
            if (data.mount.displayName == "Phasing") {
                // Can walk through enemies (but not end turn on them)
                if (occupant != null && occupant.team == team) {
                    continue; // Still can't walk through allies
                }
            } else {
                // Normal mounts: can't walk through any hero
                if (occupant != null) {
                    continue;
                }
            }

            // ... rest of existing code ...
        }
    }

    // ... return reachable ...
}
```

---

## Tutorial 9: Modifying the Battle System

**Goal**: Add custom battle mechanics (e.g., critical hits, counter prevention).

**Time**: 30-45 minutes

### Understanding Battle Flow

**Current Flow** (in `BattleManager.cs`):
1. Attacker attacks defender
2. Defender counter-attacks (if alive)
3. Check speed for double attacks
4. Award experience

---

### Example 1: Add Critical Hit System

**Goal**: 10% chance for 3x damage

**Step 1: Modify Hero.GetDamage()**

Open `Assets/Scripts/Heroes/Hero.cs`:

```csharp
public int GetDamage(Hero attacker) {
    // ... existing damage calculation ...
    int damage = CalculateBaseDamage(attacker);

    // Critical hit: 10% chance
    if (Random.value < 0.1f) {
        damage *= 3;
        Debug.Log("CRITICAL HIT! " + attacker.data.displayName + " dealt " + damage + " damage!");

        // Play critical sound
        DataManager.instance.audio.PlayEvent("Game/CriticalHit");
    }

    return damage;
}

int CalculateBaseDamage(Hero attacker) {
    // ... move existing damage code here ...
    return damage;
}
```

**Step 2: Add Visual Feedback**

Open `Assets/Scripts/Battle/BattleCutscene.cs`:

Add a critical hit animation or particle effect:

```csharp
public void ShowCriticalHit() {
    // Flash screen
    StartCoroutine(ScreenFlash());
}

IEnumerator ScreenFlash() {
    // Implement screen flash effect
    yield return new WaitForSeconds(0.2f);
}
```

---

### Example 2: Add Weapon Durability

**Goal**: Weapons break after 30 uses

**Step 1: Add Durability to HeroInfo**

Open `Assets/Scripts/Data/HeroData.cs` (or wherever HeroInfo is defined):

```csharp
[System.Serializable]
public struct HeroInfo {
    public string bundleName;
    public int level;
    public int exp;
    public int weaponDurability; // Add this
}
```

**Step 2: Initialize Durability**

Open `Assets/Scripts/Heroes/HeroManager.cs`:

```csharp
void CreateHeroInfo(string bundleName) {
    HeroInfo info = new HeroInfo {
        bundleName = bundleName,
        level = 1,
        exp = 0,
        weaponDurability = 30 // Start with 30 uses
    };
    heroes.Add(info);
}
```

**Step 3: Decrement Durability on Attack**

Open `Assets/Scripts/Battle/BattleManager.cs`:

```csharp
IEnumerator BattleCoroutine(Hero attacker, Hero defender) {
    // ... existing attack code ...

    // Reduce weapon durability
    attacker.info.weaponDurability--;

    if (attacker.info.weaponDurability <= 0) {
        Debug.Log(attacker.data.displayName + "'s weapon broke!");
        // Weapon breaks - implement penalty (e.g., reduced damage)
    }

    // ... rest of battle code ...
}
```

**Step 4: Display Durability in UI**

Add durability display to StatsUI (see Tutorial 6).

---

### Example 3: Prevent Counter-Attacks for Ranged Weapons

**Goal**: Bow users can't be countered if attacking from 2 tiles away

**Modify BattleManager.BattleCoroutine()**:

Open `Assets/Scripts/Battle/BattleManager.cs`:

```csharp
IEnumerator BattleCoroutine(Hero attacker, Hero defender) {
    // Attacker attacks
    yield return AttackSequence(attacker, defender);

    // Counter-attack logic
    if (defender.life > 0) {
        // Check if defender can counter
        bool canCounter = true;

        // If attacker is ranged and defender is melee, no counter
        if (attacker.data.weapon.range > 1 && defender.data.weapon.range == 1) {
            int distance = Utils.GetDistance(attacker.gridPosition, defender.gridPosition);
            if (distance > 1) {
                canCounter = false;
                Debug.Log(defender.data.displayName + " cannot counter from range!");
            }
        }

        if (canCounter) {
            yield return AttackSequence(defender, attacker);
        }
    }

    // ... rest of battle code ...
}
```

---

### Example 4: Add Status Effects (e.g., Stun)

**Goal**: 20% chance to stun enemy, preventing their next action

**Step 1: Add Status to Hero**

Open `Assets/Scripts/Heroes/Hero.cs`:

```csharp
public class Hero : MonoBehaviour {
    // ... existing fields ...
    public bool isStunned = false;

    public void RemoveStun() {
        isStunned = false;
    }
}
```

**Step 2: Apply Stun in Combat**

Open `Assets/Scripts/Battle/BattleManager.cs`:

```csharp
IEnumerator BattleCoroutine(Hero attacker, Hero defender) {
    // ... existing attack code ...

    // 20% chance to stun
    if (Random.value < 0.2f) {
        defender.isStunned = true;
        Debug.Log(defender.data.displayName + " is stunned!");
    }

    // ... rest of battle code ...
}
```

**Step 3: Prevent Stunned Heroes from Acting**

Open `Assets/Scripts/Game/Controller.cs`:

```csharp
void Update() {
    if (selectedHero != null && selectedHero.isStunned) {
        Debug.Log(selectedHero.data.displayName + " is stunned and cannot move!");
        return; // Can't control stunned hero
    }

    // ... existing input code ...
}
```

**Step 4: Remove Stun at Turn End**

Open `Assets/Scripts/Game/DataManager.cs`:

```csharp
public void EndTurn() {
    // ... existing code ...

    // Remove stun from all heroes of the team that just finished
    List<Hero> heroes = MapManager.GetAllHeroes(teamPlaying);
    foreach (Hero hero in heroes) {
        hero.RemoveStun();
    }

    // ... rest of existing code ...
}
```

---

### Example 5: Add Triangle Advantage Indicator

**Goal**: Show visual indicator when attacking with advantage

**Modify Controller.cs**:

```csharp
void ShowAttackPreview(Hero enemy) {
    // ... existing code ...

    // Calculate weapon advantage
    float advantage = selectedHero.data.weapon.GetAdvantage(enemy.data.weapon);

    if (advantage > 0) {
        Debug.Log("Advantage! +" + (advantage * 100) + "% damage");
        // Show green indicator
        ShowAdvantageIndicator(Color.green);
    } else if (advantage < 0) {
        Debug.Log("Disadvantage! " + (advantage * 100) + "% damage");
        // Show red indicator
        ShowAdvantageIndicator(Color.red);
    }
}

void ShowAdvantageIndicator(Color color) {
    // Implement visual indicator (e.g., colored outline around enemy)
}
```

---

## Tutorial 10: Adding Game Modes

**Goal**: Implement alternative victory conditions or rule sets.

**Time**: 45-60 minutes

### Example: Survival Mode

**Goal**: Defend against waves of enemies for 10 turns.

---

### Step 1: Add Game Mode Enum

Create `Assets/Scripts/Data/GameMode.cs`:

```csharp
public enum GameMode {
    Standard,   // Eliminate all enemies
    Survival,   // Survive X turns
    Capture,    // Capture specific tiles
    Escort      // Protect a specific hero
}
```

---

### Step 2: Add Game Mode to DataManager

Open `Assets/Scripts/Game/DataManager.cs`:

```csharp
public class DataManager : MonoBehaviour {
    // ... existing fields ...
    public GameMode gameMode = GameMode.Standard;
    public int survivalTurnsRemaining = 10; // For Survival mode

    // ... existing code ...
}
```

---

### Step 3: Modify Victory Condition

Open `Assets/Scripts/Game/DataManager.cs`:

```csharp
public void CheckVictoryCondition() {
    switch (gameMode) {
        case GameMode.Standard:
            CheckStandardVictory();
            break;
        case GameMode.Survival:
            CheckSurvivalVictory();
            break;
        case GameMode.Capture:
            CheckCaptureVictory();
            break;
        case GameMode.Escort:
            CheckEscortVictory();
            break;
    }
}

void CheckStandardVictory() {
    // Existing victory logic
    List<Hero> team0 = MapManager.GetAllHeroes(0);
    List<Hero> team1 = MapManager.GetAllHeroes(1);

    if (team0.Count == 0) {
        EndGame(1); // Team 1 wins
    } else if (team1.Count == 0) {
        EndGame(0); // Team 0 wins
    }
}

void CheckSurvivalVictory() {
    // Survive X turns
    if (turnNumber >= 10) {
        // Player survived!
        EndGame(0); // Player wins
        return;
    }

    // Check if player lost (all heroes dead)
    List<Hero> playerHeroes = MapManager.GetAllHeroes(0);
    if (playerHeroes.Count == 0) {
        EndGame(1); // Player lost
    }
}

void CheckCaptureVictory() {
    // Check if player captured all objective tiles
    // Implement tile capture logic
}

void CheckEscortVictory() {
    // Check if escort target reached destination
    // Implement escort logic
}
```

---

### Step 4: Call Victory Check in EndTurn

```csharp
public void EndTurn() {
    // ... existing code ...

    // Check victory condition
    CheckVictoryCondition();
}
```

---

### Step 5: Add Game Mode Selection UI

**Create Mode Selection Menu:**

1. Open `Assets/Scenes/Menu.unity`
2. Add UI buttons for each game mode
3. Create script `GameModeSelectionUI.cs`:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class GameModeSelectionUI : MonoBehaviour {
    public Button standardButton;
    public Button survivalButton;
    public Button captureButton;
    public Button escortButton;

    void Start() {
        standardButton.onClick.AddListener(() => SelectMode(GameMode.Standard));
        survivalButton.onClick.AddListener(() => SelectMode(GameMode.Survival));
        captureButton.onClick.AddListener(() => SelectMode(GameMode.Capture));
        escortButton.onClick.AddListener(() => SelectMode(GameMode.Escort));
    }

    void SelectMode(GameMode mode) {
        DataManager.instance.gameMode = mode;
        Debug.Log("Selected game mode: " + mode);
        // Proceed to level selection
    }
}
```

4. Attach script to UI canvas
5. Connect buttons in Inspector

---

### Step 6: Display Mode-Specific UI

**For Survival Mode, show turns remaining:**

1. Add Text to game UI: "Turns Remaining: X"
2. Create script `SurvivalUI.cs`:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class SurvivalUI : MonoBehaviour {
    public Text turnsRemainingText;

    void Start() {
        if (DataManager.instance.gameMode != GameMode.Survival) {
            gameObject.SetActive(false);
            return;
        }

        DataManager.instance.onStartTurn.AddListener(UpdateDisplay);
        UpdateDisplay();
    }

    void UpdateDisplay() {
        int remaining = 10 - DataManager.instance.turnNumber;
        turnsRemainingText.text = "Turns Remaining: " + remaining;
    }
}
```

3. Attach to UI, connect Text field

---

### Step 7: Test Game Mode

1. Play the game
2. Select Survival mode
3. Play for 10 turns
4. Verify victory condition triggers correctly

---

## Best Practices

### Code Organization

**1. Separation of Concerns**
- Keep data (ScriptableObjects) separate from logic (MonoBehaviours)
- UI scripts should only handle display, not game logic
- Game logic should be in manager classes (DataManager, BattleManager, etc.)

**2. Use Events for Communication**
- Decouple systems with UnityEvents
- Example: UI listens to game events, doesn't poll game state

**3. Scriptable Objects for Data**
- Use ScriptableObjects for all game data (heroes, weapons, maps)
- Never hardcode data in scripts

**4. Static vs Singleton**
- Use static for utility classes (MapManager, Utils)
- Use singleton for persistent managers (DataManager)

---

### Performance

**1. Cache References**
```csharp
// Bad: Searches every frame
void Update() {
    Hero hero = FindObjectOfType<Hero>();
}

// Good: Cache once
Hero hero;
void Start() {
    hero = FindObjectOfType<Hero>();
}
```

**2. Avoid Frequent Allocations**
```csharp
// Bad: Creates new list every call
public List<Hero> GetHeroes() {
    return new List<Hero>(heroes);
}

// Good: Return reference
public List<Hero> GetHeroes() {
    return heroes;
}
```

**3. Use Object Pooling** (for frequent instantiation)
- Reuse GameObjects instead of Instantiate/Destroy
- Example: Battle damage numbers, particles

---

### Testing

**1. Unit Testing Setup**
```csharp
// Create test scripts in Assets/Tests/
using NUnit.Framework;

public class HeroTests {
    [Test]
    public void TestDamageCalculation() {
        Hero attacker = CreateTestHero(atk: 10);
        Hero defender = CreateTestHero(def: 5);

        int damage = defender.GetDamage(attacker);

        Assert.AreEqual(5, damage);
    }
}
```

**2. Debug Helpers**
```csharp
// Add debug commands with keyboard shortcuts
void Update() {
    if (Input.GetKeyDown(KeyCode.F1)) {
        Debug.Log("Hero HP: " + selectedHero.life);
    }

    if (Input.GetKeyDown(KeyCode.F2)) {
        selectedHero.life = selectedHero.maxLife; // Instant heal
    }
}
```

---

### Asset Management

**1. Naming Conventions**
- **Scripts**: PascalCase (`HeroManager.cs`)
- **Assets**: lowercase (`lucina.asset`)
- **Scenes**: PascalCase (`MainMenu.unity`)
- **Folders**: PascalCase (`Heroes/`)

**2. Asset Bundle Best Practices**
- Group related assets in same bundle
- Don't create too many small bundles (increases load time)
- Don't create one massive bundle (can't load selectively)

**3. Version Control**
- Commit asset bundles to repo
- Use Git LFS for large binaries
- Don't commit `Library/` folder

---

## Common Patterns

### Pattern 1: ScriptableObject + MonoBehaviour

**Data Definition (ScriptableObject):**
```csharp
[CreateAssetMenu(fileName = "Item", menuName = "Data/Item")]
public class ItemData : ScriptableObject {
    public string displayName;
    public Sprite icon;
    public int value;
}
```

**Runtime Behavior (MonoBehaviour):**
```csharp
public class Item : MonoBehaviour {
    public ItemData data; // References ScriptableObject

    void Use() {
        Debug.Log("Used " + data.displayName);
    }
}
```

---

### Pattern 2: Event-Driven UI Updates

```csharp
public class StatsUI : MonoBehaviour {
    void Start() {
        // Subscribe to events
        DataManager.instance.onHeroSelected.AddListener(ShowHeroStats);
    }

    void ShowHeroStats(Hero hero) {
        // Update UI when event fires
        hpText.text = hero.life + "/" + hero.maxLife;
    }

    void OnDestroy() {
        // Unsubscribe to prevent memory leaks
        DataManager.instance.onHeroSelected.RemoveListener(ShowHeroStats);
    }
}
```

---

### Pattern 3: State Machine

```csharp
public enum HeroState {
    Idle,
    Moving,
    Attacking,
    Dead
}

public class Hero : MonoBehaviour {
    HeroState state = HeroState.Idle;

    void Update() {
        switch (state) {
            case HeroState.Idle:
                // Wait for input
                break;
            case HeroState.Moving:
                // Move towards target
                if (ReachedDestination()) {
                    state = HeroState.Idle;
                }
                break;
            case HeroState.Attacking:
                // Attack animation
                break;
            case HeroState.Dead:
                // Do nothing
                break;
        }
    }
}
```

---

### Pattern 4: Singleton with DontDestroyOnLoad

```csharp
public class DataManager : MonoBehaviour {
    public static DataManager instance;

    void Awake() {
        if (instance == null) {
            instance = this;
            DontDestroyOnLoad(gameObject);
        } else {
            Destroy(gameObject);
        }
    }
}
```

---

### Pattern 5: Coroutine for Sequences

```csharp
void StartBattle() {
    StartCoroutine(BattleSequence());
}

IEnumerator BattleSequence() {
    // Step 1
    PlayAttackAnimation();
    yield return new WaitForSeconds(1f);

    // Step 2
    ApplyDamage();
    yield return new WaitForSeconds(0.5f);

    // Step 3
    PlayHitAnimation();
    yield return new WaitForSeconds(1f);

    // Done
    EndBattle();
}
```

---

## Summary

You now have comprehensive guides for:
- ✅ Adding new heroes with custom stats
- ✅ Creating tactical levels with CSV maps
- ✅ Balancing character progression
- ✅ Designing weapon types with triangle mechanics
- ✅ Creating terrain with walkability rules
- ✅ Customizing UI and user experience
- ✅ Integrating audio with FMOD
- ✅ Implementing movement types
- ✅ Modifying combat mechanics
- ✅ Adding alternative game modes

**Next Steps:**
- Experiment with the tutorials
- Combine concepts to create unique features
- Share your modifications with the team!

For architecture details, see `ARCHITECTURE.md`.
For project setup, see `QUICKSTART.md`.
