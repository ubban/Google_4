# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. The entire application is contained in `index.html`, which includes embedded CSS styling and JavaScript game logic. The game runs in any modern browser with Canvas and WebAudio API support.

## Development

### Running the Game

Since this is a pure client-side HTML file with no build process:

1. **Open in browser**: Simply open `index.html` in a web browser (modern Chrome, Firefox, Safari, or Edge)
2. **Local development**: Use a simple HTTP server to avoid CORS issues:
   - Python 3: `python -m http.server 8000` then visit `http://localhost:8000/index.html`
   - Node.js: `npx http-server` then visit the provided localhost URL
3. **Testing changes**: Edit `index.html` and refresh the browser (Ctrl+F5 for hard refresh to avoid caching)

### No Build System

There is no package.json, build step, or external dependencies. All code is self-contained in the HTML file. The game uses:
- Native Canvas API for rendering
- Web Audio API for sound synthesis (no external audio files)
- Vanilla JavaScript (ES6+, no frameworks)
- CSS3 for UI and animations
- LocalStorage for persistent state (best scores, settings)

## Architecture

### High-Level Structure

The JavaScript game logic follows this pattern:

```
INITIALIZATION PHASE
├── Canvas setup & DOM element caching
├── Settings (difficulty, volume, motion preferences)
├── Event listeners (keyboard, mouse, touch)
└── Game loop via requestAnimationFrame()

GAME LOOP (per frame)
├── Input handling (keyboard, mouse, joystick)
├── Update phase (movement, collision, spawning)
└── Draw phase (render canvas)

GAME STATE
├── Player state (position, weapon, health)
├── Bullets (player and enemy)
├── Enemies (spawn, move, shoot)
├── Boss (special arena encounter)
├── Particles (explosions, visual effects)
├── Score & progression (stage, kills, multiplier)
└── UI state (HUD, overlay, pause menu)
```

### Core Game Systems

#### 1. **Game State Management** (lines 863-870)
- `player`: Position, health (lives), weapon system, invulnerability
- `enemies`: Array of spawned enemies with different types
- `boss`: Special boss encounter (one per stage)
- `bullets` / `enemyBullets`: Projectiles from both sides
- `powerUps`: Temporary items (life, bomb, weapon upgrades)
- `particles`: Explosion and visual effects
- Progression: `stage`, `kills`, `stageKills`, `multiplier`, `chain`

#### 2. **Rendering Pipeline** (lines 1520-1638, 913-1020)
- Canvas is 480×720 (BASE_W × BASE_H)
- Auto-scales to fit window, respects device orientation
- Renders in order: background → clouds → player → bullets → enemies → boss → particles → UI (HUD not on canvas)
- Uses themes: 6 different sky color schemes applied per-stage
- Screen shake effect for impacts (reduced if motion preference enabled)

#### 3. **Input System** (lines 767-862)
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Left side drag for movement (joystick UI), right side tap buttons (FIRE/BOMB)
- **Priority**: Keyboard/joystick > mouse (mouse doesn't override active keyboard)
- Input drives a movement vector `(mvx, mvy)` that constrains player within bounds

#### 4. **Combat System**
- **Player weapons** (lines 1124-1146, 871-873):
  - Vulcan: Fast spread bullets, levels 1-3 increase spread
  - Laser: Piercing beams, levels 1-3 add lanes
  - Weapon pickups toggle between types or increase level
  - Cooldown: 0.15s for Vulcan, 0.1s for Laser
- **Collision detection** (line 1069-1071): Rectangle overlap for all entities
- **Damage**: Player bullets deal 1 HP to enemies (3 HP to boss), enemy bullets one-shot player (with invuln after hit)
- **Boss attacks** (lines 1372-1409): 6 different boss types with unique attack patterns (radial, homing, columns, spirals, etc.)

#### 5. **Enemy System** (lines 1022-1041, 1210-1227)
- **Types** (ENEMY_TYPES object):
  - Grunt: Basic, 1 HP, slow, 10 points
  - Interceptor: Fast, 1 HP, high agility, 15 points (weaves side-to-side)
  - Tank: Slow, 4 HP, armored, 35 points
- **Spawn system**: Difficulty affects spawn rate (spawnBase/spawnMin decrease on harder difficulties)
- **Behavior**: Move down screen, shoot at player (aimed or spread pattern)
- **Progression**: Tank appears in stage 2+; difficulty ramps spawn rate over time

#### 6. **Stage/Boss System** (lines 1043-1056, 1441-1454)
- **Progression**: Kill 18 enemies (KILLS_PER_STAGE) per stage, then boss encounter
- **Bosses**: 6 designs cycle through stages, each with distinct appearance and attack patterns
- **Boss HP**: Scales with difficulty multiplier and boss design multiplier
- **Stage themes**: Sky colors, cloud appearance, sun glow change per stage (cycles through THEMES array)
- **Victory**: Boss defeated → stage banner → next stage spawns normal enemies

#### 7. **Scoring & Multiplier** (lines 1087-1101, 1304-1322)
- Base score = enemy type score × difficulty multiplier
- **Chain system**: Kill streak increases multiplier (1 + floor(chain/5), capped at 10x)
- **Chain timer**: 2.2s between kills to maintain chain; breaking resets multiplier
- **Power-ups**: Life (6% drop), bomb (8% drop), weapon (20% drop)
- Best score persisted to localStorage

#### 8. **Audio System** (lines 542-676)
- **WebAudio synth**: No audio files, all sounds synthesized at runtime
- **Techniques**:
  - `beep()`: Oscillator-based tones with optional sweep, filter, panning
  - `noiseBurst()`: Filtered white noise for percussion effects
- **SFX categories**: Shooting, explosions (by enemy type), boss hit, bomb, power-up, UI clicks
- **Mute toggle**: Persisted to localStorage

#### 9. **Difficulty System** (lines 485-489)
- Three presets: Easy, Normal (default), Hard
- Affects: Spawn rate, enemy speed, shoot frequency, lives, bombs, boss HP
- Selectable from start menu, persisted to localStorage

#### 10. **Accessibility & Motion** (lines 529-540, 349-358)
- **Reduced motion**: Disables animations, screen shake, vignette pulse (respects system preference)
- **Screen reader support**: Lives, score updates announced via ARIA live region (`srLive`)
- Toggleable via button in top-right

### Data Flow Example: Enemy Destroyed
1. **Collision detection** (line 1232-1252): Bullet overlaps enemy
2. **Damage & cleanup**: Enemy HP decreases, bullet removed or pierced
3. **Spawn effects**: Explosion particles at enemy position
4. **Scoring**: `addScore()` increments kills and multiplier
5. **SFX**: Type-specific explosion sound
6. **UI update**: Score/multiplier text updated, HUD re-rendered
7. **Stage check**: If kills reach KILLS_PER_STAGE, boss scheduled after 1.2s

## Key Implementation Details

### Constants & Configuration (lines 463-515)
- Storage keys for localStorage (best score, mute, difficulty, reduced motion)
- Difficulty presets (spawn rates, enemy stats, HP multipliers)
- Theme colors (sky gradients, clouds, sun hues)
- Enemy type stats (size, HP, speed multipliers, score values, stage unlock)
- Boss designs (hull colors, core color, HP multipliers, names)

### Performance Optimization (lines 474-483)
- **Adaptive quality**: Tracks frame times and scales particle counts, shadow blur on slower devices
- Quality levels: 0 (full), 1 (medium particles/shadows), 2 (low)
- Runs on 45-frame rolling average; adjusts up/down based on FPS

### Canvas Sizing (lines 752-765)
- Base resolution: 480×720 (portrait, matches mobile)
- Scales up to fit window, capped at 1.6× base size
- Responds to window resize and orientation change events

### Pause/Resume (lines 1491-1504)
- Overlay modal shows pause state, overlaid on game
- Game loop continues but `update()` skipped if paused
- Resume button in overlay re-hides it

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506) with stats
2. Update `minStage` if it should unlock later
3. Add spawn logic in `spawnEnemy()` if conditional rules needed (line 1025-1030)
4. Add explosion sound in `EXPLODE_SFX` mapping (line 678) if unique sound desired

### Adding a New Boss
1. Add to `BOSSES` array (line 508-515) with hull colors, core color, name, HP multiplier
2. Add attack patterns to `BOSS_PATTERNS` array (line 1372-1409) matching the new boss index
3. Boss selection cycles automatically via `(stage - 1) % BOSSES.length`

### Adjusting Difficulty
Edit `DIFFICULTIES` object (line 485-489) to tune spawn rate, enemy behavior, lives, bombs, or boss HP.

### Tuning Audio
- Modify `sfx` object (line 624-676) to change frequency, duration, or filter parameters
- Examples: `beep(freq, duration, type, volume, options)` or `noiseBurst(duration, volume, options)`

### Theming
- Edit `THEMES` array (line 493-500) to add or modify sky colors, cloud colors, sun glow
- Also tweak CSS gradient colors in `<style>` (e.g., `#gameCanvas` background, `#overlay` colors)

## Testing Notes

- **No unit tests** — this is a self-contained game, best validated through play-testing
- **Manual testing**: Different difficulties, all three weapon types, boss patterns, mobile touch controls
- **Performance testing**: Check frame rate on lower-end devices (adaptive quality should engage)
- **Accessibility**: Test with reduced motion enabled, keyboard-only input, screen reader

## Persistence

Game state is not persisted between sessions except:
- **Best score**: `localStorage['skyfighter.bestScore']`
- **Mute toggle**: `localStorage['skyfighter.muted']`
- **Difficulty**: `localStorage['skyfighter.difficulty']`
- **Reduced motion**: `localStorage['skyfighter.reducedMotion']`

In-game state (current score, stage, lives) is lost on page reload.
