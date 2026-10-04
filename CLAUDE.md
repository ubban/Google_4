# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and WebAudio. The entire game (~1,670 lines) is in `index.html`, featuring no external dependencies.

## Running and Testing

### Local Development

The game runs directly in any modern browser:

```bash
# Open in default browser
open index.html

# Or use a local server for better development:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Testing

- **Full gameplay**: Load the page in a browser and play through all difficulties (easy, normal, hard)
- **Specific features**:
  - Weapon switching (cycles through available weapon types)
  - Boss encounters (complete a stage to trigger)
  - Touch controls (use browser DevTools device emulation for mobile)
  - Sound (test with mute toggle in top-right corner)

### Browser Requirements

- Modern browser with:
  - HTML5 Canvas
  - Web Audio API (for in-game sound synthesis)
  - LocalStorage (for persisting high scores and settings)
  - Touch/Pointer events (for mobile support)

## Code Architecture

The codebase follows a **single-file functional structure** with clear logical sections:

### Core Systems

1. **Game State & Initialization** (lines 423-527)
   - Global variables: `score`, `lives`, `running`, `paused`, `gameOver`
   - Difficulty settings stored in LocalStorage
   - Canvas setup and DOM element references

2. **Audio System** (lines 542-676)
   - WebAudio Context initialization (`ensureAudio()`)
   - Beep synthesis for melodic sounds (`beep()`)
   - Noise burst for percussive sounds (`noiseBurst()`)
   - Sound effects object (`sfx`) with methods for each action
   - **Key patterns**: Uses `masterGain` for muting; frequency sweeps for dynamic effects

3. **Input & Controls** (lines 785-871)
   - Keyboard input via `keys` object tracking pressed keys
   - Mouse position tracking for player targeting
   - Virtual joystick for touch devices
   - Auto-firing (continuous with cooldown)

4. **Game Loop** (lines 1520-1665)
   - `update(dt)` – Game state updates each frame
   - `draw()` – Canvas rendering
   - `loop()` – Main animation frame callback

5. **Entity Systems**
   - **Player** (object): `x`, `y`, `w`, `h`, `speed`, `weapon`, `cooldown`, `invuln`
   - **Bullets** (array): Player projectiles, tracked per weapon type
   - **Enemy Bullets** (array): Enemy fire, cleaned up off-screen
   - **Enemies** (array): Spawned based on difficulty; three types (grunt, interceptor, tank)
   - **Boss** (object): Single boss per stage with unique attack patterns
   - **Explosions** (array): Particle effects for visual feedback
   - **Power-ups** (array): Weapon upgrades, health, bombs

6. **Game Mechanics**
   - **Waves**: Enemies spawn continuously; `KILLS_PER_STAGE` (18) enemy kills triggers boss
   - **Difficulty**: Settings object defines enemy stats, lives, bomb count, boss HP multiplier
   - **Themes**: Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) cycle per stage
   - **Scoring**: Base score per enemy type; multiplier for chain (killing without getting hit)
   - **Weapons**: Player cycles through weapon types on pickup; each has different fire rate and pattern

7. **Boss System** (lines 1043, 1411-1440)
   - Six unique boss designs with distinct appearances
   - Pattern-based attacks: radial fire, aimed spread, columns, spirals, homing projectiles
   - Boss telegraph sound before major attacks
   - Boss health bar display with name

8. **Visual & Audio Feedback**
   - **Screen shake** on hit (respects reduced-motion preference)
   - **Flash** effect for impacts
   - **Vignette** red overlay when critical health
   - **Combo pop** text animation for chain multiplier
   - **Stage banner** displayed when entering new stage
   - All animations check `reducedMotion` flag for accessibility

### Key Constants & Configurations

```javascript
DIFFICULTIES          // easy, normal, hard with spawn rates & enemy stats
ENEMY_TYPES          // grunt, interceptor, tank with HP & speeds
BOSSES               // 6 boss definitions with colors & HP multipliers
THEMES               // 6 visual themes (sky gradients, cloud/sun colors)
KILLS_PER_STAGE      // Enemy kills needed to trigger boss (18)
STORAGE_* keys       // LocalStorage keys for persistence
```

## Code Patterns & Guidelines

### Collision Detection

```javascript
function rectsOverlap(a, b) {
  return a.x - a.w/2 < b.x + b.w/2 && a.x + a.w/2 > b.x - b.w/2 &&
         a.y - a.h/2 < b.y + b.h/2 && a.y + b.h/2 > b.y - b.h/2;
}
```

- All game entities use `{ x, y, w, h }` bounds centered at (x, y)

### Weapon System

Weapons are fired via pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, etc.). The `applyPowerUp()` function cycles through them. Current weapon tracked in `player.weapon` string.

### Audio

Always check `muted` before playing. Use `beep()` for pitched sounds (frequency + sweep). Use `noiseBurst()` for percussion. Pan sounds using `panFor(x)` to place them horizontally based on screen position.

### Adaptive Performance

Performance monitoring (`trackPerf()`, `quality` variable) adjusts visual fidelity:
- Quality 0 (full) → 1 (medium) → 2 (low) based on frame time averages
- Currently tracks `frameTimes` but quality setting is not actively used for culling

## Important Files & Sections

- **index.html**: The only file; contains HTML structure, all CSS (embedded), and all JavaScript
- **DOM Structure**: Main elements are `#gameCanvas` (the game), `#overlay` (UI), `#hud` (stats display), `#touchControls` (mobile joystick)

## Development Notes

### Adding New Features

- **New Enemy Type**: Add to `ENEMY_TYPES` object, implement spawning logic in `spawnEnemy()`, add collision handling in `update()`, add death sound in `EXPLODE_SFX`
- **New Weapon**: Create a fire function following the pattern of existing ones (e.g., `fireRadial`), add to `applyPowerUp()` cycle
- **New Boss**: Add to `BOSSES` array, create attack pattern function in `updateBoss()`, update boss drawing if visual is different
- **New Theme**: Add to `THEMES` array; will cycle automatically

### Testing Physics

The game uses simple Euclidean distance and AABB collision. To adjust difficulty:
- Modify `DIFFICULTIES` constants for spawn rates and enemy speeds
- Change `KILLS_PER_STAGE` to adjust stage length
- Adjust boss HP multipliers in `BOSSES` array

### Accessibility

- Reduced motion preference respected via `reducedMotion` flag (toggleable in UI)
- Screen reader announcements via `announce()` → `#srLive`
- Hearts and bomb display icons; stat labels in HUD

### Mobile Considerations

- Virtual joystick uses `moveZone` for movement input
- Fire and bomb buttons overlay canvas
- Viewport fit and safe-area insets used for notched devices
- Touch events prevent default browser behaviors (zoom, etc.)

## Common Tasks

| Task | Location |
|------|----------|
| Adjust difficulty parameters | `DIFFICULTIES` object (line ~486) |
| Change visual theme progression | `THEMES` array (line ~493) |
| Add/modify enemy type | `ENEMY_TYPES` object (line ~502) + `spawnEnemy()` logic |
| Tune boss behavior | `BOSSES` array (line ~508) + `updateBoss()` function (line ~1411) |
| Add sound effect | `sfx` object (line ~624) with `beep()` or `noiseBurst()` calls |
| Change weapon patterns | Fire function in `fireWeapon()` selection (line ~1124) or new pattern function |
| Modify game speed/balancing | `dt` scaling, enemy/bullet/player speeds in respective updates |
