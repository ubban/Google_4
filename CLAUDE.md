# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single-file HTML5 application. The game is self-contained in `index.html` with no external dependencies—all CSS and JavaScript are inlined.

- **Game Type**: Browser-based arcade shooter with procedural enemy spawning and boss encounters
- **Platform**: Runs in any modern web browser (desktop/mobile)
- **Architecture**: Single HTML file (~1,670 lines) with embedded CSS, Canvas 2D rendering, and Web Audio API synth
- **Input Methods**: Keyboard (WASD/Arrows), mouse, touch (mobile joystick)

## Quick Start

### Running the Game

1. **Desktop/Web**: Open `index.html` in a browser (file:// works locally)
2. **Mobile/Testing**: Serve via `npx http-server` or similar if needed
3. No build step, no server—just open and play

### Testing Game Features

- **Desktop Controls**: Mouse to steer, click to fire, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mobile Controls**: Drag left zone to move, tap FIRE/BOMB buttons
- **Difficulty Modes**: Easy/Normal/Hard (selectable before game start, persisted in localStorage)
- **Audio**: Click sound icon to mute/unmute (persisted)
- **Reduced Motion**: For accessibility; respects `prefers-reduced-motion` media query

## High-Level Code Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) with the following major sections:

### 1. **State Management & Configuration** (~line 863–869)
- Core game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game flags: `running`, `paused`, `gameOver`, `score`, `lives`
- Stage progression and combo/multiplier tracking
- `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` constants define game content

### 2. **Canvas & Rendering** (~line 752–1638)
- **Canvas Setup** (`resize()`): Base 480x720, scaled responsively up to 1.6x
- **Drawing Functions**:
  - `drawPlane()`: Renders player and enemy aircraft
  - `drawBossShip()`: Boss-specific multi-layered rendering
  - `drawClouds()`: Parallax background
  - `draw()`: Main render loop—clears canvas, applies shake, draws all entities
- **Visual Effects**: Screen shake, flash overlays, vignette (critical health), combo pop-ups, stage banners

### 3. **Audio System** (~line 542–676)
- **Web Audio Synth**: No external audio files; all SFX generated via oscillators, noise, and filters
- **Key Functions**:
  - `beep(freq, dur, type, vol, opts)`: Parametric tone generator with sweep and filtering
  - `noiseBurst(dur, vol, opts)`: Filtered noise (explosions, bombs)
  - `sfx` object: Pre-composed sound effects (shoot, laser, explosions, boss patterns, UI clicks)
- Spatial audio: `panFor()` calculates stereo pan based on x-coordinate
- Master gain & compressor for control

### 4. **Game Logic** (~line 1148–1505)
- **`update(dt)`**: Main update loop
  - Player movement (keyboard/joystick/mouse priority)
  - Weapon firing with cooldown
  - Enemy spawning with difficulty ramp
  - Collision detection (bullets vs. enemies, bullets vs. boss, player vs. hazards)
  - Chain/combo system: kills without gaps increase multiplier
  - Difficulty-scaled enemy AI and boss spawning at stage thresholds
- **`fireWeapon()`**: Weapon-level spread (Vulcan: spread bullets; Laser: piercing vertical shots)
- **`applyPowerUp(type)`**: Weapon upgrades, life restoration, bomb refills

### 5. **Boss & Enemy AI** (~line 1022–1439)
- **Enemy Types** (`ENEMY_TYPES`): Grunt, Interceptor, Tank with varying HP, speed, and behavior
  - Weaving enemies use sine-wave lateral movement
  - Shoot patterns: "spread" (wide fan) or "aimed" (homing toward player)
- **Bosses** (`BOSSES`): 6 uniquely named bosses with distinct color schemes and HP multipliers
- **Boss Attack Patterns** (`BOSS_PATTERNS`): 6 separate arrays of attack functions (radial bursts, homing, spirals, etc.)
  - Each boss cycles through 3 unique patterns with telegraph windup
  - Patterns fire at randomized or aimed trajectories

### 6. **Input Handling** (~line 768–861)
- **Keyboard**: Arrow/WASD movement, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Steer to pointer (lower priority than keyboard/joystick), click to fire
- **Touch**: Virtual joystick in left zone, FIRE/BOMB buttons in right zone
- Priority system: Keyboard/joystick > mouse > none (prevents input conflicts)

### 7. **Performance Adaptation** (~line 474–483)
- Adaptive quality system: Monitors frame times; downgrades particle/glow rendering if avg FPS drops below target
- Three quality levels: 0 (full), 1 (medium—60% particles), 2 (low—35% particles)

### 8. **UI & Persistence** (~line 517–526, 732–750, 1491–1504)
- **HUD**: Score, best score, multiplier, weapon level/type, health hearts, bomb count
- **Boss Bar**: Shows boss name and health during boss fights
- **Overlays**: Main menu, pause screen, game-over screen with stats
- **LocalStorage Keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Key Game Systems

### Difficulty Scaling

Difficulties define spawn rate, enemy speed, enemy aggression, starting lives, and boss HP multipliers. Difficulty is selectable pre-game and persisted. Spawn rate also ramps over time (`difficultyRamp = Math.floor(frame / 300) * 4`).

### Combo & Multiplier Mechanic

- Killing enemies within 2.2 seconds extends the "chain"
- Multiplier = 1 + floor(chain / 5), capping at 10x
- Score += kill_value × multiplier
- Breaking the chain (taking damage or timeout) resets

### Stage Progression

- Each stage requires 18 (KILLS_PER_STAGE) enemy kills before boss spawns
- After boss defeat, stage increments; spawn rates reset but enemies become harder
- Themes cycle (6 backgrounds: Daylight, Sunset, Night, Storm, Arctic, Volcanic)

### Power-Up Drop Rates

- **Life**: 6% drop chance (restores 1 HP up to max)
- **Bomb**: 8% drop chance (refills 1 bomb)
- **Weapon**: 20% drop chance (upgrade weapon or cycle weapon type)

### Weapon System

Two weapons with 3 levels each:
- **Vulcan**: Rapid spread bullets (Level 1→spread widens, Level 3→5-way spread)
- **Laser**: Piercing vertical shots (Level 2→dual lanes, Level 3→triple lanes + center)

Weapon switch preserves level; advancing to Level 4 swaps to the other weapon at Level 1.

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optional: Set `weave: true` for sine-wave movement
3. Update the `spawnEnemy()` filter and probability logic if needed
4. Add explosion SFX to `EXPLODE_SFX` mapping
5. Test spawning on relevant stages

### Adding a New Boss

1. Add to `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, and `hpMul`
2. Create a new 3-pattern array in `BOSS_PATTERNS` following the existing attack function signatures
3. Adjust base HP in `spawnBoss()` if needed (currently `70 + stage * 35`)
4. Test at the corresponding stage (Stage 1→BOSSES[0], Stage 2→BOSSES[1], etc.)

### Adding a New Stage Theme

1. Add entry to `THEMES` array with `sky` (3-stop gradient), `cloud`, `sun` RGB values, and descriptive `name`
2. Themes cycle by stage modulo: `(stage - 1) % THEMES.length`
3. Gradient drawn at frame render; adjust colors for visibility

### Tweaking Difficulty or Balance

- `DIFFICULTIES` object controls spawn rate (`spawnBase`, `spawnMin`), enemy speed, shoot chance, lives, bombs, boss HP
- Enemy `hp` in `ENEMY_TYPES` and boss damage per hit (currently 3 for player bullets, 24 for bomb) tune difficulty
- Multiplier formula: `1 + Math.min(9, Math.floor(chain / 5))` caps at 10x; adjust divisor to change pacing

### Audio Tuning

- Frequencies, durations, and envelope shapes in `sfx` functions are tweakable
- `masterGain.gain.value` (currently 0.9) and compressor settings control volume
- Use browser DevTools to visualize frequency response if needed

## Implementation Notes

### Important Quirks & Patterns

1. **Frame-Based Timing**: Game uses `frame` counter (incremented each update) for animations and timing. Not wall-clock time, so animations are frame-locked (e.g., `b.telegraph > 0`).

2. **Collision Detection**: Naive AABB (axis-aligned bounding box) via `rectsOverlap()`. No spatial hashing; adequate for current enemy count but may bottleneck with many entities.

3. **Input Priority**: Movement input hierarchy is Keyboard/Joystick → Mouse, preventing conflicts. Mouse sets `mouseActive` flag on any canvas event; `mouseActive` resets on `mouseleave`.

4. **Canvas Sizing**: CSS scales via `transform: scale()` but logical canvas is always 480×720. Coordinates from mouse/touch are scaled back via `canvasPointFromEvent()`.

5. **Particle System**: Particles are simple, free-flying objects. Quality setting culls count, not visual size.

6. **Boss Telegraph**: Bosses announce attacks with a 22-frame (time-based) visual and audio telegraph before firing; gives player reaction time.

7. **Screen Shake**: Applied via `ctx.translate()` in draw loop; magnitude and duration set by `screenShake()` call. Reduced motion setting caps both.

8. **Reduced Motion**: Respects OS preference and provides UI toggle; disables animations (vignette pulse, combo pop, etc.) and caps screen shake/flash.

### No External Assets or Dependencies

- All visuals drawn procedurally via Canvas 2D API
- All audio synthesized in real-time via Web Audio API
- All fonts use system sans-serif (Segoe UI → Roboto → Arial)
- Gameplay fully deterministic given same input and difficulty seed (no external RNG libraries)

## Testing Checklist

Before shipping changes:

1. **Game Loop**: Frame rate stable? No visual stuttering or audio glitches?
2. **Collision**: Player and enemy bullets collide correctly? Boss hits register?
3. **UI**: Overlay screens (menu, pause, game-over) appear/disappear correctly? Difficulty selection persists?
4. **Audio**: All SFX play without clipping? Mute toggle works? Web Audio context resumes on user interaction?
5. **Mobile**: Touch controls responsive? Joystick moves smoothly? Orientation changes handled?
6. **Accessibility**: Reduced motion toggle works? Screen reader announcements clear?

## Performance Considerations

- Canvas is fixed 480×720; scaling is CSS-only
- Particle count adapts to frame rate (see `quality` variable)
- Boss patterns spawn many bullets; could batch-render for efficiency
- No WebGL or custom shaders; relies on CPU canvas rendering
- Mobile performance varies; test on target devices

## File Structure

- **index.html** (~1,670 lines): Complete game (no separate files)
  - Lines 1–359: HTML + inline CSS
  - Lines 360–420: DOM elements (canvas, UI overlays)
  - Lines 422–1667: Inline `<script>` with game logic

No build step, no compilation needed.
