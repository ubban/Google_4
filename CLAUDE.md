# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. The game runs entirely in the browser with no external dependencies—all code, styling, and audio synthesis are embedded in `index.html`.

## Architecture

### Single-File Structure
The entire game is contained in `index.html` (1,669 lines):
- Lines 1–359: HTML markup and CSS styling
- Lines 422–1669: JavaScript game logic within a single IIFE (Immediately Invoked Function Expression)

### Core Game Systems

#### 1. **Configuration Constants** (lines 485–515)
- `DIFFICULTIES`: Three difficulty modes (easy, normal, hard) with spawn rates, enemy stats, and boss HP multipliers
- `THEMES`: Six visual themes with sky gradients and color schemes
- `ENEMY_TYPES`: Three enemy classes (grunt, interceptor, tank) with varying speed, HP, and scoring
- `BOSSES`: Six boss variants with different color schemes and HP multipliers
- `KILLS_PER_STAGE`: Number of enemy kills required to advance to the next stage

#### 2. **Audio System** (lines 542–676)
- **WebAudio synthesis**: No external audio files—all sounds are generated via WebAudio API
- `ensureAudio()`: Lazily initializes the AudioContext with compressor chain
- `beep(freq, dur, type, vol, opts)`: Generates simple tones with optional frequency sweeps and filters
- `noiseBurst(dur, vol, opts)`: Creates white-noise based sounds for explosions and impacts
- `sfx` object: Contains 12 named sound effect functions (shoot, laser, explode variants, bomb, powerup, etc.)
- Audio respects user mute preference and reduced-motion settings

#### 3. **Input Handling** (lines 768–837)
- **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Move player to cursor position; click to fire
- **Touch**: Virtual joystick on left; buttons on right (FIRE and BOMB)
- Dynamic touch controls adapt to device capabilities (only shown on touch devices)

#### 4. **Game State & Persistence** (lines 517–527, 463–466)
- Uses `localStorage` for best score, mute state, difficulty, and reduced-motion preference
- Four storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

#### 5. **Rendering Pipeline** (lines 1520–1639)
- Uses Canvas 2D rendering at 480×720 logical resolution
- `draw()`: Master render function that clears, draws backgrounds, entities, UI, and effects
- Theme-based sky gradients and parallax cloud layers
- HUD updates for score, lives, bombs, weapon level, and boss health bar

#### 6. **Game Loop** (lines 1640–1669)
- `loop(now)`: requestAnimationFrame-based game loop
- Calls `update(dt)` for logic and `draw()` for rendering
- Includes adaptive performance tracking (`trackPerf()`) to reduce render quality on slower devices

#### 7. **Entity Management**
- **Player** (`player` object): Position, velocity, health, weapon state, bomb count
- **Bullets** (`bullets` array): Stores both player and enemy projectiles
- **Enemies** (`enemies` array): Dynamic array of active enemies
- **Boss** (`boss` object): Active boss ship (null when inactive)
- **Explosions** (`explosions` array): Particle effects
- **Power-ups** (`powerups` array): Collectible weapon upgrades

#### 8. **Weapon System** (lines 1124–1147, 1304–1371)
- Three weapon types: Vulcan (spray), Laser (penetrating), Barrage (rapid bursts)
- Each weapon has 3 levels that modify fire rate, spread, and damage
- Weapons upgraded via power-ups collected from defeated enemies
- Boss ships fire unique patterns (lines 1372–1409): Radial, aimed spread, columns, spiral, cross, homing, double-ring

#### 9. **Difficulty & Progression**
- Stages increment after defeating 18 enemies (KILLS_PER_STAGE)
- Each stage spawns a random theme and boss
- Boss difficulty scales with stage progression
- Combo multiplier increases for consecutive hits without missing

#### 10. **Visual Feedback**
- `flashHit()`, `flashBomb()`: Screen flashes on impact
- `screenShake(mag, dur)`: Camera shake effect
- `showComboPop(text)`: Floating combo text
- `showStageBanner(text)`: Stage clear announcement
- Vignette effect when player health is critical (1 HP remaining)

### Performance Optimization
- Adaptive quality system (`quality` variable): Reduces particle count or draw detail on slower devices
- Frame time tracking over 45 frames to detect performance degradation
- Touch controls hidden on non-touch devices to reduce DOM overhead

### Accessibility
- Screen reader support via `#srLive` aria-live region (function `announce()`)
- Reduced-motion option that disables animations and screen shake
- High contrast visuals with drop shadows for readability
- ARIA labels and semantic HTML

## Development Notes

### Key Functions for Modification

**Adding/Modifying Content:**
- `DIFFICULTIES`: Tweak spawn rates, enemy speed, boss HP multipliers
- `ENEMY_TYPES`: Add new enemy classes or modify existing ones
- `BOSSES`: Customize boss appearance (colors) and HP multipliers
- `BOSS_PATTERNS`: Add new attack patterns for bosses (radial spread, aimed, etc.)

**Sound Design:**
- `sfx` object: Add new sound effects using `beep()` or `noiseBurst()`
- Tweak parameters: frequency, duration, filter, pan, volume

**Game Tuning:**
- `KILLS_PER_STAGE`: Controls stage progression speed
- Difficulty settings: Spawn rate (`spawnBase`, `spawnMin`), enemy behavior
- Player speed and hitbox: Search `player.x` and `player.y` movement

**UI/Visual:**
- CSS in `<style>` block: Modify HUD appearance, overlays, animations
- `THEMES`: Add new sky gradient and cloud colors

### Code Patterns

1. **Entity Objects**: Store as objects with properties (x, y, w, h, hp, etc.) and update via reference
2. **Sound Triggers**: Call sfx functions on game events (shoot, hit, etc.)
3. **Canvas Rendering**: All drawing passes through `ctx` (2D context) with manual state management (save/restore)
4. **Input Polling**: `keys` object holds boolean state; read in `update()` to drive player movement
5. **LocalStorage Persistence**: Wrap access with try/catch or ensure it's wrapped (e.g., `localStorage.getItem()`)

### Testing
- Open `index.html` in a browser to test locally
- Test across difficulty modes (Easy, Normal, Hard) using in-game menu
- Verify audio works by toggling mute button (🔊)
- Check touch controls on mobile or DevTools device emulation
- Verify reduced-motion mode disables animations (🎆 button)

### Common Modifications

**Changing Enemy Balance:**
- `ENEMY_TYPES`: Modify `speedMul`, `hp`, `score`, `minStage` properties
- `DIFFICULTIES`: Adjust `enemyShootChance` to control bullet density

**Adding a New Weapon Type:**
1. Create a new firing function (e.g., `fireSpread()`)
2. Add weapon name and properties to weapon logic in `update()` and `fireWeapon()`
3. Assign a color and level display in the HUD update logic

**Boss Customization:**
- Modify `BOSSES` array colors (hullTop, hullBottom, wing, core)
- Add new boss pattern function and reference in `BOSS_PATTERNS` array

## Building and Running

No build step is required. Simply open `index.html` in a modern browser:
```bash
# Direct file open (works from file:// protocol)
open index.html  # macOS
start index.html  # Windows

# Or serve locally for best results
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

The game is fully playable without a server, though local server access is recommended for consistent behavior across browsers.

## Browser Requirements

- Modern Canvas API support (2D context)
- WebAudio API (for sound synthesis)
- localStorage API (for high-score persistence)
- ES6 JavaScript (arrow functions, destructuring, template literals)
- Recommended: Chrome/Edge 90+, Firefox 88+, Safari 14+

## Notable Implementation Details

1. **No External Assets**: All graphics are drawn via Canvas; all audio is synthesized—no images, fonts, or sound files needed.
2. **Mobile-First Touch Handling**: Virtual joystick and buttons adapt to screen size.
3. **Performance Adaptation**: Automatically reduces render quality if frame rate drops below 40 FPS.
4. **Pause State**: Game can pause mid-level without resetting; paused state preserved if switching tabs.
5. **Combo System**: Player score multiplier builds on consecutive enemy hits, resets on player hit or missed shot.
