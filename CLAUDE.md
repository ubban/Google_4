# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. It runs directly in the browser with no build process or dependencies. The game features:

- Three difficulty levels (Easy, Normal, Hard)
- Multiple weapon types with upgrade system
- Six unique boss encounters with distinct attack patterns
- Procedurally spawned enemy waves across 6 environmental themes
- Web Audio API synthesized sound effects
- Responsive input (keyboard, mouse, and touch joystick)
- Score multiplier system for consecutive kills
- Adaptive performance quality scaling

## Running the Game

Simply open `index.html` in a web browser. The game is fully self-contained.

**Local testing:** Use a simple HTTP server:
```bash
python3 -m http.server 8000
# or
npx http-server
```

Then navigate to `http://localhost:8000` and open `index.html`.

## Architecture Overview

The entire game logic is contained in a single ~1670-line HTML file with three major sections:

### 1. **DOM Setup & UI (lines 2–420)**
- Canvas element for game rendering
- HUD elements (score, lives, weapon, multiplier)
- Overlay system for menus, pause screen, game over
- Touch control UI (joystick, action buttons)
- Mute and reduced-motion accessibility controls

### 2. **Core Game Loop & State (lines 422–1670)**

The game runs in a single IIFE with:

- **State Management (lines 863–870):** Player, bullets, enemies, power-ups, boss, score, lives
- **Input Handling (lines 767–861):** Keyboard (WASD/Arrows), mouse steering, touch joystick with dual-zone layout
- **Update Loop (lines 1148–1302):** Handles physics, collisions, spawning, and state changes
- **Render Loop (lines 1520–1638):** Canvas drawing for sky, clouds, entities, particles
- **Game Loop (lines 1640–1666):** RequestAnimationFrame with frame-rate-independent delta-time

### 3. **Key Game Systems**

#### Audio (lines 542–676)
- **No external audio assets** — uses Web Audio API to synthesize all SFX
- `beep()`: Generates tones with optional sweep, filter, and pan
- `noiseBurst()`: Creates noise-based explosion/impact sounds
- `sfx` object: Defines all sound effects (shoot, explode, powerup, boss patterns, etc.)
- Master gain and compressor for consistent volume

#### Difficulty System (lines 485–489)
Three presets control spawn rates, enemy speed, shoot chance, lives, and boss HP scaling.

#### Enemy System (lines 502–1227)
- **Three enemy types** with varying stats and behavior
- Spawn pattern: "spread" (radial fire) vs "aimed" (toward player)
- Weaving interceptors add movement variety
- Tank enemies have visible health bars

#### Boss System (lines 507–1439)
- **Six boss designs** (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has a unique 3-attack pattern sequence
- Attack patterns include radial bursts, homing missiles, beam columns, spiral fire, and more
- Boss enters with animation, telegraphs attacks with core glow effect

#### Weapon System (lines 1124–1146)
- **Vulcan:** Spread shot with 1–3 lanes based on level
- **Laser:** Piercing shots that can hit multiple enemies
- Weapon upgrades cycle between levels 1–3 and then switch weapon type

#### Power-ups (lines 1058–1322)
- Life recovery, bomb pickup, weapon upgrade
- Spawn probability on enemy death

#### Visual Effects (lines 1073–1085, 1628–1637)
- Particle explosions with velocity and fade
- Adaptive quality reduces particle count on low frame rates
- Screen shake for impacts
- Flash overlays for hits and bombs

#### Performance Tracking (lines 474–483)
- Frame times averaged over 45 frames
- Auto-scaling quality (0=full, 2=low) reduces particle counts and glow effects
- Threshold: quality increases if avg frame time > 1/40s

#### Theming (lines 493–500)
- Six sky gradients, cloud colors, and sun glows
- Cycles per stage for visual variety
- Purely cosmetic; no gameplay impact

## Making Common Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502–506) with stats
2. Add kill sound to `EXPLODE_SFX` mapping (line 678)
3. Spawn logic in `spawnEnemy()` (lines 1022–1041) handles chance-based selection

### Adding a New Boss
1. Add to `BOSSES` array (lines 508–515) with colors and HP multiplier
2. Add attack pattern functions like `fireRadial()`, `fireColumns()`, etc. (lines 1324–1370)
3. Add pattern array to `BOSS_PATTERNS` (lines 1372–1409)
- Index must match the boss's position in the BOSSES array

### Adding a New Theme
1. Add object to `THEMES` (line 493) with sky colors, cloud, and sun RGB values
2. Cycles automatically every stage via modulo indexing

### Tuning Game Balance
- **Difficulty parameters:** `DIFFICULTIES` object (line 485)
  - `spawnBase` / `spawnMin`: Enemy spawn rate (lower = faster)
  - `enemySpeed`, `enemyShootChance`: Enemy aggression
  - `lives`, `bombs`, `bossHpMul`: Player resources and boss HP
- **Base stage:** `KILLS_PER_STAGE = 18` (line 491) — kills to trigger boss
- **Weapon tuning:** `fireWeapon()` (lines 1124–1146) for spread patterns and fire rates
- **Player speed:** `player.speed = 260` (line 877)

### Adjusting Audio
- Toggle `muted` flag or `localStorage.setItem(STORAGE_MUTE, '1')`
- Tweak synth parameters in `sfx` object (lines 624–676)
- Adjust `masterGain.gain.value` in `ensureAudio()` (line 556)

## Key Variables & Constants

| Constant | Purpose | Location |
|----------|---------|----------|
| `BASE_W`, `BASE_H` | Canvas logical size (480×720) | Line 753 |
| `KILLS_PER_STAGE` | Enemy kills to spawn boss | Line 491 |
| `DIFFICULTIES` | Per-difficulty tuning | Line 485 |
| `STORAGE_*` | LocalStorage keys for persistence | Lines 463–466 |
| `quality` | Adaptive quality level (0–2) | Line 474 |

## Input Handling

- **Keyboard:** WASD/Arrows move, Space/click fire, X/Shift bomb, P/Esc pause
- **Mouse:** Move pointer to steer, click to fire
- **Touch:** Left half-screen dragging for movement joystick, FIRE/BOMB buttons on right

Priority: Keyboard/joystick input > mouse input for movement conflicts.

## Performance Considerations

1. **Particle system:** `quality` variable reduces count on low FPS (lines 1074–1075)
2. **Glow effects:** Shadow blur disabled at quality 2 (line 1002)
3. **Frame cap:** Delta-time clamped to 0.05s to prevent large jumps (line 1644)
4. **Canvas size:** Logical 480×720 scaled with CSS to prevent large render targets

## Deployment

The game is ready to deploy as-is. Only `index.html` is needed. The file contains:
- All HTML structure
- All CSS styling
- All JavaScript logic
- Embedded favicon (as data URI)

No external assets, APIs, or server-side code required.

## Debugging Tips

- Open browser console (F12) to see no errors in a healthy game
- `quality` variable (logged in code) indicates FPS-based scaling
- `localStorage` persistence:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Mute state
  - `skyfighter.difficulty`: Selected difficulty
  - `skyfighter.reducedMotion`: Accessibility setting

