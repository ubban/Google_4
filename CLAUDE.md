# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (HTML, CSS, JavaScript, and assets) is contained in `index.html`. It features:

- Canvas-based 2D graphics rendering
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 stages with boss battles
- Dynamic weapon system (Vulcan guns and Laser)
- Particle effects, explosions, and screen shake
- Procedurally-generated sound effects using WebAudio API
- Touch controls for mobile and keyboard/mouse for desktop
- Adaptive quality scaling based on frame performance
- Local storage persistence for high scores and settings

## Architecture Overview

The game is organized as a single IIFE (Immediately Invoked Function Expression) that manages all game state and logic. The architecture divides into these major systems:

### 1. **Rendering System**
- Canvas setup: Base resolution `480x720`, scalable to screen size (lines ~754-765)
- Custom drawing functions: `drawPlane()`, `drawBossShip()`, render methods for enemies, particles
- Quality levels: 0 (full quality), 1 (medium), 2 (low) - adapted based on frame times (lines ~474-483)
- Theme system: 6 different sky themes that cycle through stages (lines ~493-500)

### 2. **Game State**
Central state variables declared at line ~864:
- `player`: Player object with position, speed, weapons, health
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`: Game entity arrays
- `score`, `lives`, `multiplier`, `chain`: Gameplay metrics
- `stage`, `stageKills`, `kills`: Progression tracking
- `running`, `paused`, `gameOver`: Game lifecycle flags

### 3. **Input System** (lines ~767-861)
- Keyboard: Arrow keys or WASD for movement, Space to fire, X/Shift/P to bomb/pause
- Mouse: Crosshair cursor control when hovering canvas
- Touch: Joystick (left half) + Fire/Bomb buttons (right half)
- All input modes coordinate without conflict; keyboard/joystick priority over mouse

### 4. **Collision & Physics** (lines ~1232-1298)
- `rectsOverlap()` helper (not shown in provided excerpt) used throughout
- Enemies collide with player bullets, player collides with enemies/bullets/boss
- Power-up pickups handled via `applyPowerUp()`
- Projectiles auto-filtered based on bounds

### 5. **Enemy System** (lines ~502-1041)
- **Enemy Types**: Grunt (1 HP), Interceptor (1 HP, fast), Tank (4 HP, slow)
- **Spawning**: `spawnEnemy()` respects stage unlock levels and difficulty-based spawn rates
- **Behavior**: 
  - Enemies move down at difficulty-scaled speeds
  - Interceptors weave side-to-side
  - Enemies shoot either spread patterns or player-aimed bullets
  - Boss summons after 18 kills per stage
- **Stage Progression**: Each stage requires `KILLS_PER_STAGE` (18) enemy kills before boss appears

### 6. **Boss System** (lines ~508-515, 1043-1049)
- 6 unique boss designs that cycle per stage
- Spawned after stage kill quota (via delayed timeout at line ~1206)
- Boss includes entering animation, attack patterns, health bar UI
- Boss patterns include: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`
- Boss defeat triggers stage clear and progression

### 7. **Weapon System** (lines ~871-872, 1177-1181)
- Two weapons: **Vulcan** (rapid-fire bullets) and **Laser** (piercing projectiles)
- Fire rate: Vulcan 0.15s, Laser 0.1s
- Power-up leveling: Level 1→2→3 within weapon, then swap to other weapon at level 1
- Laser bullets pierce enemies (tracked via `pierce` counter)

### 8. **Difficulty System** (lines ~485-489)
- Three presets: Easy, Normal, Hard
- Affects: spawn rates, enemy speed, shoot chance, starting lives/bombs, boss HP
- Persisted to localStorage

### 9. **Audio System** (lines ~543-676)
- WebAudio synthesis (no external assets)
- Master gain + compressor chain
- Sound effects for: shooting, explosions (per enemy type), powerups, boss events, UI clicks
- Stereo panning based on event location
- Mute toggle with localStorage persistence

### 10. **Collision Resolution & Score** 
- Enemy kill awards `score` points (Grunt 10, Interceptor 15, Tank 35)
- Score multiplier increases with kill chain (resets if 3+ seconds without kill)
- Boss defeated increments stage and resets spawn timer
- Player hit: loses 1 life, plays hit SFX/flash effect, brief invuln period

### 11. **UI & HUD** (lines ~368-390)
- Top HUD: Score, Best Score, Multiplier, Weapon, Hearts, Bombs
- Top-right buttons: Mute, Reduced Motion toggle, Pause
- Boss Health Bar: Shows when boss active
- Overlay: Screens for game over, stage transitions, difficulty selection
- Accessibility: Screen reader announcements via `srLive` element

### 12. **Visual Effects**
- Particle system: Explosions, trails (lines ~1296-1297)
- Screen shake: `screenShake(magnitude, duration)` (lines ~727-730)
- Flash overlay: Brief white/colored flash on hit/bomb (lines ~698-709)
- Vignette pulse: Red screen edge warning when health critical (1 HP remaining)

## Key Code Sections

| System | Lines | Key Functions |
|--------|-------|---|
| Game Loop | ~1130-1302 | `update(dt)` - main update function called each frame |
| Rendering | ~1450+  | `render()` - canvas drawing per frame |
| Input | 767-861 | Event listeners for keyboard, mouse, touch |
| Enemies | 1022-1041 | `spawnEnemy()`, enemy type definitions |
| Boss | 1043-1070 | `spawnBoss()`, `updateBoss()` |
| Weapons | 1077-1120 | `fireWeapon()`, weapon-specific logic |
| Collisions | 1232-1295 | Enemy-bullet, boss-bullet, player-enemy checks |
| Score/Combos | 1360+ | `addScore()`, chain multiplier mechanics |
| Audio | 545-676 | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` effects |
| Storage | ~517-527 | localStorage keys: `bestScore`, `muted`, `difficulty`, `reducedMotion` |

## Running the Game

**Desktop (Browser)**
- Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- Controls: WASD or Arrow keys to move, Space to fire, X or Shift for bombs, P/Esc to pause
- Click difficulty buttons before hitting "Start Game"

**Mobile/Touch**
- Open in mobile browser; touch controls auto-enable
- Left zone: drag joystick for movement
- Right zone: Fire and Bomb buttons

**No build step required** — the game is a standalone HTML file with no dependencies.

## Development Notes

### Storage Keys
- `skyFighterBest`: High score persistence
- `skyFighterMute`: Mute state (1 or 0)
- `skyFighterDiff`: Selected difficulty (easy/normal/hard)
- `skyFighterReducedMotion`: Reduced motion preference (1 or 0)

### Constants to Modify
- **Game balance**: `DIFFICULTIES` object (line ~485) controls spawn rates, enemy stats, lives
- **Enemy types**: `ENEMY_TYPES` object (line ~502) defines HP, speed, visuals, scoring
- **Boss designs**: `BOSSES` array (line ~508) for colors and HP multipliers
- **Stages**: `KILLS_PER_STAGE` (line ~491) controls stage length
- **Themes**: `THEMES` array (line ~493) for sky colors and visual style
- **Canvas size**: `BASE_W`, `BASE_H` (line ~753) for game resolution

### Quality Adaptation
Frame time tracking keeps a rolling 45-frame average and adjusts `quality` level. This gates expensive operations:
- Shadow effects, glow, special rendering only at `quality === 0`
- This affects visual polish but maintains playability on slower devices

### Accessibility Features
- Screen reader announcements for game state changes
- Reduced motion mode disables animations/screen shake/flash
- High contrast colors chosen for readability
- Keyboard-only playable (no pointer required)

### Common Tasks

**Adjust enemy balance:**
Edit `DIFFICULTIES` spawn/speed values or `ENEMY_TYPES` HP/score values.

**Add a new enemy type:**
Add entry to `ENEMY_TYPES`, add minStage check, update `spawnEnemy()` selection logic.

**Add a new boss pattern:**
Create new pattern function following the `fireRadial()` / `fireAimedSpread()` style, add to boss update logic.

**Change visual theme:**
Add/edit entries in `THEMES` array (sky gradients, cloud/sun colors).

**Tweak audio:**
Edit `sfx` object frequency, duration, and filter parameters.

## Browser Compatibility

- **Canvas 2D**: Required (all modern browsers)
- **WebAudio**: Optional (falls back gracefully if unavailable)
- **Touch Events**: Auto-detected; controls adapt if present
- **localStorage**: Used for persistence; degrades if unavailable
- **Tested**: Chrome, Firefox, Safari, Edge on desktop and mobile
