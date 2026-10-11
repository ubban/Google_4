# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features a player-controlled fighter jet that battles waves of enemies and multi-phase boss battles across 6 themed stages, with difficulty scaling and weapon upgrades.

**Key characteristics:**
- Single-file architecture (index.html contains all HTML, CSS, and JavaScript)
- No build process or external dependencies
- Full WebAudio synthesis for dynamic sound effects
- Touch and keyboard input with mouse steering
- Adaptive performance scaling (quality levels 0-2)
- Persistent state via localStorage
- Accessibility features (screen reader announcements, reduced motion support)

## Running the Game

Simply open `index.html` in any modern browser. The game starts with the title screen where players can select difficulty (Easy/Normal/Hard) and start playing.

**Testing in development:**
```bash
# Python 3 local server (works in any directory containing index.html)
python3 -m http.server 8000

# Node.js http-server
npx http-server
```

Then visit `http://localhost:8000` or `http://localhost:8080` in your browser.

## Architecture Overview

The entire game logic is contained in a single IIFE (Immediately Invoked Function Expression) starting at line 423. The architecture follows these patterns:

### Core Systems

1. **Canvas & Rendering (lines 752-1638)**
   - Canvas dimensions: 480×720 base, scales responsively
   - All drawing uses Canvas 2D API with custom shape functions (`drawPlane`, `drawBossShip`, etc.)
   - Adaptive glow effects controlled by `quality` variable (disabled on low-end devices)

2. **Game Loop (lines 1640-1653)**
   - `requestAnimationFrame`-based loop with frame delta-time tracking
   - `update(dt)` → `draw()` → next frame
   - Performance tracking: frame times are monitored and `quality` is adjusted automatically (lines 473-483)

3. **Input Handling (lines 768-862)**
   - Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse: Position-based steering with firing on click
   - Touch: Two-zone virtual controls (movement zone left, action zone right with FIRE/BOMB buttons)
   - Joystick simulation for touch movement (base + stick)

4. **Game State (lines 864-868)**
   - `player`: position, weapon, level, health, invulnerability timer
   - `enemies`, `enemyBullets`, `bullets`, `particles`, `powerUps`: managed arrays
   - `boss`: active only during stage boss encounter
   - Score, lives, bombs, stage tracking, combo/multiplier system

### Key Game Mechanics

**Weapon System (lines 1124-1146, 1312-1322):**
- Two weapon types: Vulcan (spread bullets) and Laser (piercing, multi-lane)
- Each upgrades in 3 levels, then switches to the other weapon
- Vulcan: spread increases with level; Laser: pierce count and lane count increase
- Cooldown: Vulcan 0.15s, Laser 0.1s (faster fire rate)

**Difficulty Settings (lines 485-489):**
Define spawn rates, enemy speed, shoot frequency, lives, bombs, and boss HP multiplier. Each difficulty has:
- `spawnBase`, `spawnMin`: enemy spawn interval in frames
- `enemySpeed`: movement speed multiplier
- `enemyShootChance`: probability enemy shoots
- `lives`, `bombs`: starting resources
- `bossHpMul`: boss HP scaling factor

**Combo/Multiplier System (lines 1087-1101):**
- Kill streak increases `chain` counter with 2.2s decay timer
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Multiplier breaks when player is hit

**Boss Encounter (lines 1043-1056, 1411-1434):**
- Triggered after 18 kills per stage (`KILLS_PER_STAGE`)
- Each stage cycles through 6 boss designs with unique attack patterns
- Boss has telegraph animation (red glow) before each attack pattern
- Boss health = base HP (70 + stage × 35) × difficulty HP mul × boss design HP mul

**Boss Attack Patterns (lines 1372-1409):**
Six pattern sets, one per boss. Each boss has 3 patterns that cycle. Pattern functions use helper methods:
- `fireRadial(boss, count, speed, [offset])`: bullets in all directions
- `fireAimedSpread(boss, count, speed, spreadStep)`: bullets aimed at player with spread
- `fireColumns(boss, speed)`: vertical bullet columns
- `fireSpiralBurst(boss, speed)`: rotating spiral
- `fireCross(boss, speed)`: 8-direction cross fire
- `fireHomingSeeded(boss, count, speed)`: seeded spread toward player
- `fireDoubleRing(boss, speedA, speedB)`: concentric rings at two speeds

### Theme System (lines 493-500)

Six visual themes cycle through stages. Each defines:
- `sky`: 3-stop gradient colors
- `cloud`, `sun`: RGBA component strings for parallax cloud rendering
- `name`: human-readable theme name

Theme changes every stage, wraps around after stage 6.

## Common Tasks

### Adding a New Enemy Type
1. Define in `ENEMY_TYPES` object (lines 502-506): width, height, HP, speed multiplier, color, score, minimum stage
2. Update spawn logic in `spawnEnemy()` (lines 1022-1041) to include the new type in the weighted random choice
3. Add drawing logic in `draw()` (line 1603) if the shape differs from `drawPlane()`
4. Add explosion sound in `EXPLODE_SFX` mapping (line 678) if custom audio needed

### Adding a New Boss
1. Add entry to `BOSSES` array (lines 508-515): name, hull colors, wing color, core color, HP multiplier
2. Add attack pattern set to `BOSS_PATTERNS` array (lines 1372-1409), indexed by boss position
3. Boss draws via `drawBossShip()` (lines 970-1020) using the design colors

### Adjusting Game Balance
- **Enemy spawn rate**: Modify `spawnBase`/`spawnMin` in `DIFFICULTIES` (lines 485-489)
- **Enemy bullet speed**: Change `speed` parameter in enemy bullet pushes (lines 1219, 1223)
- **Player movement speed**: `player.speed = 260` (line 877)
- **Weapon fire rate**: `fireDelay` values (line 1177)
- **Powerup drop chance**: Rates in `maybeSpawnPowerUp()` (lines 1060-1066)
- **Stage progression**: `KILLS_PER_STAGE = 18` (line 491)

### Audio System
All sound is procedurally generated via WebAudio synth (lines 542-676). No external audio files.
- `beep(freq, duration, type, volume, opts)`: Tone-based sounds (shoot, hit, etc.)
- `noiseBurst(duration, volume, opts)`: Noise-based sounds (explosion, bomb)
- Options: `sweepTo` (frequency sweep), `filterFreq`/`filterType`, `pan` (stereo position), `delay`

Sound effects are muted when `muted` flag is true or stored in localStorage. Master gain is compressed for dynamic range control (lines 549-557).

### Screen Shake & Visual Feedback
- `screenShake(magnitude, duration)`: Sets shake parameters; applied in draw() via canvas translate
- `flashHit()` / `flashBomb()`: CSS animation classes on flash element
- `showComboPop(text)`: Floating combo text animation
- Reduced motion preference is respected: shake reduced to 15% magnitude and capped at 0.1s (line 728)

## Performance & Quality Scaling

The game monitors frame render time and automatically scales visual quality to maintain 60 FPS:
- **Quality 0**: Full effects (glow, shadows, high particle count)
- **Quality 1**: Medium (reduced particle count: 60% density)
- **Quality 2**: Low (minimal effects: 35% particle count, no shadow blur)

Quality adjustment happens every 45 frames (line 479). This keeps the game playable on lower-end devices.

## Accessibility Features

- **Screen Reader Support**: Key game events announced via `announce()` to `#srLive` element (line 468-470)
- **Reduced Motion**: User preference detected via `prefers-reduced-motion` media query; animations disabled and shake/flash reduced
- **Color Contrast**: UI text uses sufficient contrast; game uses high-saturation colors for visibility
- **Pause Functionality**: P key or pause button to pause game and show overlay

## Browser Storage

Uses `localStorage` for persistence (keys defined at lines 463-466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## Debugging Tips

- **View current stage/kills**: Check `stage`, `stageKills`, `kills` in console during gameplay
- **Force boss spawn**: Set `stageKills = 18` in browser console, or manually call `spawnBoss()`
- **Disable quality scaling**: Modify `trackPerf()` to skip quality updates (line 482)
- **Slow motion**: Multiply `dt` in the game loop to see frame-by-frame behavior
- **View collision hitboxes**: Add debug rectangles in `draw()` using `ctx.strokeRect()` on entity bounding boxes
- **Audio diagnostics**: Check `audioCtx.state` (suspended/running) and `masterGain.gain.value` for volume

## Code Style Notes

- No external libraries or build tooling
- Minimal comments; code is self-documenting through clear naming
- Heavy use of inline object literals for configuration (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
- Collision detection uses axis-aligned bounding box (AABB) via `rectsOverlap()` (line 1069-1071)
- Enemy movement: vertical + optional horizontal weave pattern for interceptors
