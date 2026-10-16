# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML5 file with embedded CSS and JavaScript. The game is played in a web browser and features:
- Fast-paced arcade gameplay with stage progression
- Weapon upgrade system (Vulcan rapid-fire vs. Laser)
- Smart bomb mechanic with screen-clearing effect
- Multiple difficulty levels (Easy, Normal, Hard)
- Dynamic difficulty scaling with adaptive performance optimization
- Procedurally varied enemy waves and 6 unique boss encounters
- Score multiplier chains and combo system
- Touch controls for mobile and mouse/keyboard for desktop

## Architecture & Core Systems

The entire game runs within a single JavaScript IIFE (Immediately Invoked Function Expression) in `index.html:422-1669`. The architecture is organized into functional modules rather than classes:

### Game State
All game state is stored in module-scoped variables declared at line 863-869:
- `player`: player object with position, velocity, weapon, level, cooldown timers
- `bullets`: array of player projectiles
- `enemies`: array of active enemy ships
- `enemyBullets`: array of enemy projectiles
- `particles`: array of explosion/effect particles
- `powerUps`: array of collectible power-ups
- `boss`: current boss object or null
- Game counters: `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`, etc.

### Key Configuration Objects

**Difficulty Settings** (line 485-489): Each difficulty has spawn rates, enemy speed/AI behavior, starting lives, bombs, and boss HP multiplier.

**Enemy Types** (line 502-506): grunt, interceptor, tank — each with different stats (hp, speed, color, point value, appearance stage).

**Boss Types** (line 508-515): 6 unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with distinct visual colors and HP multipliers.

**Themes** (line 493-500): 6 visual themes with gradient colors for sky/clouds/sun — one theme randomly selected per game.

### Main Game Loop

The game updates via `update(dt)` function (line 1148+) called from the RAF loop. Key subsystems:

1. **Player Movement & Input** (line 1155-1165): Updates player position based on keyboard/mouse/touch input
2. **Weapon Firing** (line 1168-1171): Manages weapon cooldown and automatic fire with pattern variation by weapon type
3. **Enemy Spawning** (line 1175-1189): Time-based enemy spawning with difficulty scaling; triggers boss when kill threshold reached
4. **Physics & Collision** (line 1195-1240): Updates bullets/enemies/particles; detects collisions with enemies, enemy bullets, power-ups; applies damage
5. **Combo Chain** (line 1242-1251): Manages kill chain timer and multiplier decay

### Audio System

WebAudio synthesis only (no external files). The `sfx` object (line 624+) defines sound generation functions:
- `shoot`: rapid bleeps for weapon fire
- `explodeGrunt/Interceptor/Tank`: pitched noise bursts
- `bombShoot`, `bombHit`: bomb activation and detonation sounds
- `levelUp`, `bossDeath`, `gameOver`: major event sounds

The `audioCtx` (created on first user interaction) drives all sound generation. Audio is gated by the `muted` flag from localStorage.

### Rendering Pipeline

The canvas render cycle (RAF at bottom of file, line 1615+):
1. Clear and draw gradient sky background with parallax clouds
2. Draw all game entities (player, enemies, bullets, particles, power-ups)
3. Draw player plane with `drawPlane()` (line 913+) — custom vector graphics
4. Draw boss with `drawBossShip()` (line 970+) — colored hull segments and glowing core
5. Draw UI overlays: HUD stats, boss health bar (when active), stage banner, combo pop-ups

### Performance Optimization

Adaptive quality system (line 473-483) tracks frame times and adjusts rendering detail:
- Quality 0 (full): Normal particle count and effects
- Quality 1 (medium): Reduced particles
- Quality 2 (low): Minimal particles, simplified rendering

### Control Input Handling

**Desktop**: Mouse movement (line 849), mouse click for fire (line 856), keyboard (WASD/arrows for move, Space for fire, X/Shift for bomb, P/Esc for pause).

**Mobile**: Touch drag on move zone (line 785+) updates joystick; tap FIRE/BOMB buttons (line 1547+). Joystick visualization with base circle and stick pointer.

### Power-Up System

Five power-up types (line 1304+):
- `rapid`: Temporary rapid-fire boost to Vulcan weapon
- `laser`: Switch to laser weapon (sustained beam)
- `health`: Restore one life
- `bomb`: Add one smart bomb
- `shield`: Brief invulnerability window

Power-ups spawn randomly from defeated enemies (line 1058-1067).

### Boss Behavior

Each boss has:
- Distinct visual coloring (hull, wings, glowing core)
- Three attack patterns: radial burst, aimed spread, and spiral (selected randomly per boss)
- Health bar display (line 1054+)
- Enrage effect with visual vignette pulse when at critical health

Boss spawning (line 1043-1056) occurs after 18 kills per stage; boss defeat advances to next stage.

## Development & Modification Guide

### Running the Game

Simply open `index.html` in any modern web browser. No build step, server, or dependencies required.

### Common Modifications

**Adjust Difficulty Balance**: Edit the `DIFFICULTIES` object (line 485) to tweak spawn rates, enemy speed, or boss HP.

**Add/Modify Enemy Types**: Extend `ENEMY_TYPES` object (line 502) and update spawn logic in `spawnEnemy()` (line 1022).

**Create New Boss**: Add entry to `BOSSES` array (line 508), then update `spawnBoss()` (line 1043) to select it. Implement new attack pattern in fireRadial/fireAimedSpread/fireColumns/fireSpiralBurst functions.

**New Weapon or Attack Pattern**: Add function like `fireWeapon()` variants (e.g., `fireBeam()` for laser), then integrate into weapon switch logic (line 1124+).

**Tune Audio**: Modify beep frequencies, durations, and volumes in the `sfx` object (line 624). Audio parameters are in Hz (frequency), seconds (duration), and 0-1 (volume).

**Adjust Visual Theme**: Edit `THEMES` array (line 493) to add new color gradients.

### Key Constants & Tuning

- `KILLS_PER_STAGE`: 18 (line 491) — kills required to advance stage
- Player speed: 260 (line 877)
- Spawn base/min timers vary by difficulty (line 485-489)
- Particle and bullet limits are soft (no explicit cap, but adaptive quality reduces count)

### Debugging Tips

- Open browser DevTools (F12) and check the Console for any errors
- The `announce(text)` function (line 468) logs events to the screen-reader live region (useful for testing game state)
- Frame time tracking in `trackPerf()` (line 476) provides FPS diagnostics
- Boss health bar element (`bossBarFill`) is hidden by default; toggled by game logic

## Storage & User Settings

Game state is persisted to `localStorage` with these keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (1/0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1/0)

These are loaded on page load and updated as user changes settings.

## Accessibility

- ARIA live region (`#srLive`) announces major events (stage advance, bomb depletion, etc.)
- Color contrast meets WCAG AA standards
- Reduced motion setting disables screen shake, flash effects, and animations
- Keyboard-only gameplay is fully supported
- Touch controls on mobile with large tap targets
