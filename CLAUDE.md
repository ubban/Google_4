# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features:
- Progressive difficulty with 3 difficulty levels (Easy, Normal, Hard)
- 6 distinct boss encounters with unique visual designs
- Dynamic enemy types (grunt, interceptor, tank) with varying behaviors
- Weapon upgrade system with multiple weapon types
- Power-up mechanics and bomb drops
- 6 visual themes with different sky gradients and environments
- Adaptive performance quality scaling based on frame rate
- Touch controls for mobile devices and keyboard/mouse for desktop
- Web Audio API for sound effects and procedural audio generation
- Persistent high score tracking via localStorage

## Code Architecture

### Single-File Structure
The entire game is implemented in `index.html` (~1,669 lines), containing:
- **HTML**: Game canvas container, UI overlays, HUD elements, touch control zones
- **CSS**: Responsive layout, animations (keyframes for effects), theming, accessibility
- **JavaScript**: Game logic, rendering, input handling, audio, state management

### Core Game Systems

**Game Loop & State** (lines ~423-520)
- IIFE wrapper for encapsulation
- Canvas initialization and element references
- Difficulty definitions (DIFFICULTIES object)
- Enemy types (ENEMY_TYPES with stats like hp, speed, score)
- Boss definitions (BOSSES array with visual colors and HP multipliers)
- Theme definitions (THEMES with gradient colors for different stages)
- Storage keys for persistence (best score, mute state, difficulty, reduced motion)

**Game Objects & Collections** (referenced throughout)
- `player`: Current player object with position, health, weapon, bomb count
- `enemies`: Array of active enemy objects
- `boss`: Current boss object (null when no boss active)
- `bullets`: Array of player bullets
- `enemyBullets`: Array of enemy/boss bullets
- `explosions`: Array of particle effect objects
- `powerUps`: Array of active power-up objects
- `stage`: Current stage number, `stageKills`: kills toward next stage

**Rendering Pipeline** (update & render cycle)
- `update(dt)`: Game logic (lines ~1148+)
  - Updates player position/state based on input
  - Moves and collides all objects
  - Updates weapon fire rates
  - Spawns new enemies/bosses/power-ups
  - Applies damage/knockback
  - Handles score/multiplier/chain logic
- `render()`: Canvas drawing (implicit in draw functions)
  - `drawPlane()`: Renders player/boss ships with flame effects
  - `drawBossShip()`: Complex multi-part boss visualization
  - Individual drawing for bullets, enemies, explosions, etc.

**Input Handling**
- Keyboard: WASD/Arrows for movement, Space/Z for fire, X for bomb
- Mouse: Click anywhere to move player to mouse position
- Touch: Joystick on left (movement), buttons on right (fire/bomb)
- Touch joystick functions: `joystickStart()`, `joystickMove()`, `joystickEnd()`

**Audio System** (lines ~545-630)
- `ensureAudio()`: Initializes Web Audio context (handles browser autoplay policy)
- `beep()`: Procedural sine/triangle/square wave generation for tones
- `noiseBurst()`: White noise generation for explosions
- `panFor()`: Stereo panning based on screen position
- Audio muting controlled via localStorage with `updateMuteBtn()`

**Weapon System** (lines ~871-875, ~1124+)
- `weaponLabel()`: Maps weapon types to display names
- `fireWeapon()`: Fires based on current weapon type and level
- Weapon types spawn different bullet patterns:
  - Radial: `fireRadial()`
  - Aimed spread: `fireAimedSpread()`
  - Columns: `fireColumns()`
- Weapons upgrade with power-ups; higher levels = more bullets/better patterns

**Power-Up & Bomb System**
- `maybeSpawnPowerUp()`: Random spawn when enemies die
- `applyPowerUp()`: Increases weapon level or refills bomb count
- `useBomb()`: Spawns radial bullet pattern and damage explosion
- Power-ups animate upward and expire after moving off-screen

**Collision & Damage**
- `rectsOverlap()`: Simple AABB collision detection
- Player takes damage on bullet hit; boss/enemies take damage on player bullet hit
- Screen shake and visual flashes on damage
- Health tracking via `renderHearts()` and `renderBombs()`

**Scoring & Progression**
- `addScore()`: Adds score with difficulty multiplier, updates high score
- `breakChain()`: Resets combo multiplier when player is hit
- Chain multiplier increases per enemy killed without taking damage
- Stages progress: 18 kills per stage, each stage changes visual theme and enemy composition
- Bosses appear after completing stages

**Visual Effects**
- `screenShake()`: Temporal shake applied to camera offset
- `flashHit()`: White screen flash on player damage
- `flashBomb()`: Longer flash on bomb detonation
- `showComboPop()`: Floating text animation for combo multiplier
- `showStageBanner()`: Stage transition text animation
- Vignette effect (red pulsing) when player health is low

**Performance Adaptation** (lines ~473-483)
- `trackPerf()`: Measures frame time, maintains 45-frame rolling average
- `quality` variable: 0 = full quality, 1 = medium, 2 = low
- Auto-downgrades quality if avg frame time > 1/40s (25ms)
- Auto-upgrades if consistently under 1/56s (~17.8ms)
- Affects particle count and render complexity where used

### UI & State Management

**Overlays** (game states)
- **Menu**: Shows on game start, displays high score and difficulty selection
- **Game Over**: Shows final score, best score, stage reached
- **HUD**: Always visible during gameplay showing score, multiplier, weapon, health, bombs

**Difficulty System**
- Stored in localStorage; can be changed from menu before each game
- Affects spawn rate, enemy speed, enemy shooting frequency, player lives, bomb count, boss HP
- Difficulty buttons: `.diffBtn` with active state tracking

**Accessibility**
- Screen reader live region: `srLive` (id used for announcements)
- Reduced motion detection via media query
- Motion control button to disable motion effects

## Development Commands

**No build or test infrastructure** — this is a standalone HTML file. To develop:

1. **Run locally**: Open `index.html` in a web browser (file:// protocol works)
2. **Live server** (recommended for development):
   - Using Python 3: `python3 -m http.server 8000` then visit `http://localhost:8000`
   - Using Node.js (with http-server): `npx http-server` then visit the shown URL
   - Using VS Code Live Server extension: Right-click `index.html` → "Open with Live Server"

3. **Testing changes**: Reload browser (Ctrl+R / Cmd+R) to see changes
4. **Clear cached data**: Open DevTools → Application → LocalStorage → Delete `skyfighter.*` keys to reset high score and settings

## Key Modification Points

When adding features, consider:

- **New enemy types**: Add to `ENEMY_TYPES` object (lines ~502-506), set stats and appearance
- **New bosses**: Add to `BOSSES` array (lines ~508-515), define hull/wing/core colors and HP multiplier
- **New weapon patterns**: Add function like `fireRadial()` (lines ~1324+), then call from `fireWeapon()`
- **New visual themes**: Add to `THEMES` array (lines ~493-500) with gradient colors and environment name
- **Game balance**: Adjust `DIFFICULTIES` parameters (lines ~485-489) or `KILLS_PER_STAGE` (line ~491)
- **Audio**: Extend procedures in `beep()` and `noiseBurst()` or add new sound functions
- **UI strings**: Search for hardcoded text in overlays; localization would need extraction

## Browser APIs & Dependencies

- **Canvas API**: 2D rendering context for all visuals
- **Web Audio API**: Sound generation and mixing
- **localStorage**: Persistence of best score, settings
- **requestAnimationFrame**: Game loop timing
- **Touch Events**: Mobile input (touchstart, touchmove, touchend)
- **Keyboard Events**: Desktop input
- **matchMedia**: Reduced motion preference detection

No external libraries or CDN dependencies — fully self-contained.

## Common Tasks

**Debug performance**: Open DevTools → Performance tab → Record a few seconds of gameplay → Check frame rate and bottlenecks. Adjust `quality` thresholds (line ~481-482) if needed.

**Adjust game difficulty**: Edit `DIFFICULTIES` object to tweak spawn rates, enemy behavior, or player resources.

**Test on mobile**: Use browser DevTools device emulation (Ctrl+Shift+M / Cmd+Shift+M) or deploy to a static server and visit from a real device.

**Modify visual theme**: Edit `THEMES` array to add new color schemes; the theme is selected based on `stage % THEMES.length`.

**Add sound**: Create new procedural audio function following `beep()` or `noiseBurst()` pattern, then call during appropriate game events.
