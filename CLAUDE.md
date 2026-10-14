# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is self-contained in a single `index.html` file with embedded HTML, CSS, and JavaScript (no external dependencies). It features:
- Wave-based enemy spawning with 3 enemy types
- Progressive difficulty stages with escalating boss encounters
- Weapon upgrade system (Vulcan and Laser weapons with levels)
- Procedurally varying boss attack patterns
- Full touch and keyboard/mouse controls
- Web Audio API synthesized sound effects
- Adaptive performance quality scaling

## How to Run

The game runs in any modern web browser:
- **Local file**: Open `index.html` directly in a browser (file:// protocol works)
- **Via web server**: Serve the file over HTTP for full Web Audio support in all browsers
- **Live testing**: Use Claude Code's built-in dev server if available

## Key Architecture

The game follows a classic game loop pattern with these major systems:

### Core Game Loop (lines 1640-1653)
- `requestAnimationFrame()` drives the loop at ~60 FPS
- Each frame: `update(dt)` handles logic, `draw()` renders, performance tracking adapts quality
- Delta-time clamped at 50ms to prevent spiral behavior

### Game State (lines 863-869)
- Global state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game flow: `running`, `paused`, `gameOver` flags
- Progression: `stage`, `kills`, `stageKills`, `chain`/`multiplier` for combo scoring

### Major Subsystems

**Input Handling (lines 768-861)**
- Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: Move to steer (takes priority only when no keyboard input active), click to fire
- Touch: Joystick in left 55%, action buttons (FIRE/BOMB) in right 45%
- Desktop vs mobile detected at line 778

**Physics & Collision (lines 1069-1071, 1232-1295)**
- All collision via axis-aligned bounding box overlap test: `rectsOverlap(a, b)`
- Bullets removed when off-screen
- Damage: player bullets 1 HP to enemies, 3 HP to boss; lasers pierce up to 3 hits

**Enemy System (lines 1022-1041, 1210-1228)**
- Three types: grunt (1 HP), interceptor (1 HP, fast, weaving), tank (4 HP, slow, has health bar)
- Types unlocked per stage (minStage property)
- Spawning intervals controlled by difficulty and frame count
- Pattern types: 'spread' (radial) or 'aimed' (targeted at player)

**Difficulty System (lines 485-489)**
- Three difficulties (easy/normal/hard) affect: spawn rates, enemy speed, fire rate, lives, bombs, boss HP
- Difficulty persists in localStorage; picked at game start from overlay

**Weapon System (lines 871-872, 1124-1146, 1304-1322)**
- Two weapons: Vulcan (spread shot) and Laser (piercing, higher fire rate)
- Both have 3 levels; weapon upgrades cycle between the two
- Weapon level affects fire pattern (spread count, number of shots)
- Laser weapons can pierce enemies/bosses

**Boss System (lines 1043-1056, 1324-1408, 1411-1454)**
- Boss triggers after KILLS_PER_STAGE (18) enemy kills
- Six boss types cycle per stage, each with distinct visual design
- Each boss has 3 unique attack patterns defined in BOSS_PATTERNS array (lines 1372-1409):
  - Pattern types: radial burst, aimed spread, columns, spiral, cross, homing
  - Patterns fire on a telegraph/attack loop
- Boss HP scales with difficulty and stage

**Audio System (lines 542-676)**
- No external audio files; all synth generated via Web Audio API
- Oscillators and noise bursts with frequency sweeps, filters, and panning
- Each game event has dedicated SFX function
- Master gain and compressor for dynamics

**Visual Effects**
- Screen shake (lines 726-730) for impacts/bosses
- Flash effects (lines 698-709) for hits and bombs
- Combo popups (lines 711-717) at kill milestones
- Stage banner animations (lines 719-724)
- Particle system (line 1073-1085) for explosions
- Vignette pulse effect (line 687) when health critical

**Performance Adaptation (lines 474-483)**
- Quality setting (0=full, 1=medium, 2=low) auto-adjusted based on frame time
- Low quality reduces particle counts and disables glow shadows
- Reduced motion setting (localStorage) clips animations for accessibility

### Game Balance Constants

- **Stage progression**: KILLS_PER_STAGE = 18 (line 491)
- **Base boss HP**: 70 + stage*35, multiplied by difficulty and boss type (line 1046)
- **Scoring**: Base score per enemy varies (10-35), multiplied by chain bonus (1-10x)
- **Enemy spawning**: Decreases over time via difficultyRamp (line 1201)
- **Weapon fire rates**: Vulcan 0.15s, Laser 0.1s (line 1177)

## Code Navigation

Within the single `index.html` file:

| Section | Lines | Purpose |
|---------|-------|---------|
| HTML structure | 361-420 | Canvas, HUD, overlay, touch controls |
| CSS | 10-359 | Game styling, animations, responsive layout |
| Variables & constants | 422-523 | DOM refs, storage keys, difficulty/theme/enemy configs |
| Audio system | 542-676 | WebAudio initialization and SFX functions |
| Render helpers | 680-709 | Hearts, bombs, flash, combo/stage banners |
| Input handlers | 768-861 | Keyboard, mouse, touch input |
| Game state | 863-869 | Global game variables |
| Core game logic | 871-1302 | Reset, update, collision, scoring |
| Weapon & power-ups | 1124-1322 | Fire, weapon application |
| Boss patterns | 1324-1408 | Attack pattern definitions |
| Boss update | 1411-1454 | Boss behavior and defeat |
| Drawing | 1507-1638 | Canvas rendering (clouds, player, enemies, effects) |
| Main loop | 1640-1665 | Game loop and startup |

## Common Tasks

**Add a new enemy type**: Add entry to ENEMY_TYPES (line 502), set minStage for when it appears, adjust spawning logic.

**Add a new weapon**: Extend the weapon === 'laser' check (line 1549); add new weapon case to fireWeapon() and weaponLabel().

**Adjust difficulty**: Modify DIFFICULTIES object (line 485); the spawnBase/Min control enemy spawn rate, enemySpeed/ShootChance control combat, lives/bombs/bossHpMul control survival.

**Add boss attack pattern**: Extend BOSS_PATTERNS array (line 1372) with new pattern function; the function receives boss `b` and fires into `enemyBullets`.

**Tweak game balance**: Key numbers are stage progression (KILLS_PER_STAGE=18), base boss HP formula (line 1046), enemy type HP values (ENEMY_TYPES), scoring multipliers (addScore logic around line 1087).

**Test a specific stage/boss**: Modify resetGame() to set `stage` to desired value before spawning.

## Input Priority & Control Flow

Keyboard/joystick movement takes priority over mouse steering when active. This prevents fighting between input methods. Fire and bomb commands work from any input source (keyboard, mouse, touch buttons).

## Accessibility

- Reduced motion preference is detected and respected (prefers-reduced-motion media query)
- All major game events announced via screen reader (srLive element)
- Touch controls automatically enable on touch devices
- High contrast colors for visibility

## Browser Compatibility

Requires modern browser with:
- Canvas 2D context
- Web Audio API (for sound; game still runs muted if unavailable)
- Touch Events (for mobile)
- localStorage (for persistence; game still runs without it)
