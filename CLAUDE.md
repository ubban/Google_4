# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application (`index.html`). It features multiple difficulty levels, weapon progression, boss encounters, and supports both desktop and mobile input methods.

## Quick Start

**No build process required.** Open `index.html` directly in a web browser to play the game.

```bash
# On macOS/Linux
open index.html

# On Windows
start index.html

# Or use a simple HTTP server
python3 -m http.server 8000  # then visit http://localhost:8000/index.html
```

## Project Structure

The entire game is contained in a single `index.html` file (1669 lines) with three sections:

1. **HTML Structure** (lines 361–420): Game canvas, HUD elements, overlay UI, touch controls
2. **CSS Styling** (lines 10–359): Dark sci-fi theme with responsive design; handles animations, HUD layout, and reduced-motion preferences
3. **JavaScript Game Engine** (lines 422–1669): Core gameplay loop, entity management, input handling, audio synthesis

**No external dependencies or assets**—all CSS and audio (WebAudio synthesis) are inline.

## Architecture & Key Systems

### Game Loop & Canvas Rendering
- **Canvas setup**: `gameCanvas` element (2D context) with adaptive resolution scaling
- **Main loop**: `requestAnimationFrame`-based loop with delta-time calculation
- **Performance tracking**: Adaptive quality system (`quality` variable) adjusts rendering details based on frame times

### Entity Management System
The game manages several entity types without a formal class structure:
- **Player**: Single aircraft entity with position, velocity, health, weapon state
- **Enemies**: Three types (grunt, interceptor, tank) with properties from `ENEMY_TYPES` config
- **Bosses**: Six unique boss designs from `BOSSES` array with special attack patterns
- **Projectiles**: Player bullets and enemy bullets with simple collision detection
- **Particles**: Explosions, hit effects, visual feedback

### Game State & Configuration

**Difficulty Presets** (`DIFFICULTIES` object):
```javascript
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```
Affects enemy spawn rate, aggression, player lives, bomb count, and boss health scaling.

**Theme System** (`THEMES` array):
Six visual themes affecting sky gradient, clouds, and sun colors. Theme selection per stage adds visual variety.

**Enemy Types** (`ENEMY_TYPES` object):
- Grunt: Basic red fighter (1 HP, low speed)
- Interceptor: Fast orange fighter (1 HP, high speed, weaving pattern)
- Tank: Slow purple tank (4 HP, powerful)

**Boss Designs** (`BOSSES` array):
Six bosses with unique colors, health multipliers, and movement patterns. Players face one randomly-selected boss per stage.

### Input Handling
- **Keyboard**: Arrow keys / WASD (movement), Space / X / Shift (fire / bomb), P / Esc (pause)
- **Mouse**: Position tracks player movement, click fires weapon
- **Touch**: Two-zone control system—left side for movement (with joystick indicator), right side for FIRE/BOMB buttons
- **Detection**: `touchControls` class conditionally enabled on touch devices

### Audio System
- **WebAudio synthesis**: No external sound files—all sounds generated via oscillators and noise bursts
- **Audio effects**: Weapon fire, boss sounds, hits, explosions, UI feedback
- **Master gain & compressor**: Dynamic range control for consistent volume
- **Mute state**: Persisted to localStorage; affects all audio generation

### HUD & UI
- **Real-time displays**: Score, best score, combo multiplier, weapon name/level, health hearts, bomb count
- **Boss health bar**: Appears when boss active, scales width by remaining HP
- **Status announcements**: Screen-reader accessible via `srLive` (aria-live="polite")
- **Overlay menu**: Difficulty selection, game-over screen with new-best highlights

### Game Mechanics

**Weapon Progression**:
- Starts with Vulcan (spread bullets)
- Upgrades to Plasma (fewer, faster shots) and Bomb-Ray (piercing rounds)
- Pickup drops increase weapon level; levels reset on player death

**Score Multiplier**:
- Resets to 1× on hit; increases with consecutive kills
- Displayed in HUD; visual feedback when combo achieved

**Boss Stages**:
- 18 enemy kills per stage triggers boss encounter
- Boss defeated triggers stage clear, spawn rate resets
- Game never ends; stages continue indefinitely

**Health & Bombs**:
- Difficulty determines starting lives (3–4) and bomb count (2–3)
- Heart icons display current lives; bombs show available count
- Death resets weapon level but retains bombs

### Responsive & Accessibility Features
- **clamp() CSS**: Font sizes and spacing adapt from mobile to desktop
- **Safe area insets**: HUD and buttons respect notched devices (iPhone, etc.)
- **Reduced motion**: `prefers-reduced-motion` media query respected; toggle via settings icon
- **Screen reader support**: aria-live region for game announcements
- **Crosshair cursor**: Over game canvas for precision aiming

## Common Development Tasks

### Adding a New Weapon Type
1. Locate the weapon configuration (search `weaponSub` to find weapon definitions within the script)
2. Add bullet pattern and firing logic to the weapon handling section
3. Update weapon names and styling in HUD display logic

### Modifying Boss Difficulty
Edit the `BOSSES` array to adjust `hpMul` (health multiplier) or modify boss movement/attack pattern logic in the boss AI section of the game loop.

### Adjusting Enemy Spawn Rates
Modify `DIFFICULTIES` object properties: `spawnBase` (initial spawn counter) and `spawnMin` (minimum spawn interval).

### Adding a New Theme
Insert a new object into the `THEMES` array with properties: `sky` (gradient colors), `cloud` (RGB string), `sun` (RGB string), and `name`. The game randomly selects a theme per stage.

### Tuning Game Balance
Key variables to adjust:
- `KILLS_PER_STAGE`: Enemy kills required to trigger boss (currently 18)
- `DIFFICULTIES.<level>`: Spawn rates, enemy speed, health, bomb count
- `BOSSES[].hpMul`: Boss health scaling
- Enemy damage, player damage, collision detection in movement/firing loops

## Testing in Browser

The game state persists via localStorage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preferences

**To reset all saved data:**
```javascript
localStorage.removeItem('skyfighter.bestScore');
localStorage.removeItem('skyfighter.muted');
localStorage.removeItem('skyfighter.difficulty');
localStorage.removeItem('skyfighter.reducedMotion');
```

Run this in the browser console while the game page is open.

## Performance Considerations

- **Adaptive quality system**: The game monitors frame timing and reduces quality (fewer particles, simpler rendering) on slower devices
- **Canvas size**: Maintains aspect ratio and scales to fit viewport without exceeding screen dimensions
- **Entity pooling**: Not implemented—consider if frame drops occur with many active entities

## Browser Compatibility

Requires modern browser features:
- Canvas 2D API
- WebAudio API (for sound synthesis)
- localStorage
- Flexible CSS layout (flexbox, clamp(), safe-area-inset)
- Tested on Chrome, Firefox, Safari (macOS/iOS), Edge

## File Locations & Important Code Sections

- **Game initialization**: Lines 517–527 (load saved prefs)
- **Main game loop**: Search for `let gameRunning`, `function step()`, and `requestAnimationFrame`
- **Player movement & firing**: Search for `playerX`, `playerY`, `weapon` variables
- **Collision detection**: Search for distance-based hit checks in entity loops
- **Boss encounters**: Search for `isBoss` flag and boss-specific AI
- **Audio synthesis**: Lines 542–620 (beep, noiseBurst functions and audio context setup)
- **Touch input handling**: Lines 1490–1630 (touch zone setup and joystick logic)

## Notes for Future Development

1. **Single-file structure**: Changes are straightforward but file size grows with features. Consider modular refactoring if complexity increases significantly.
2. **No external assets**: Audio synthesis is lightweight but game SFX may sound synthetic. Real audio files could improve feel.
3. **Input lag on slow devices**: Touch control joystick updates on every move event; consider throttling if performance drops.
4. **Persistent multiplier feedback**: Combo multiplier displays onscreen; ensure visual feedback remains clear with new themes or HUD changes.
