# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML/CSS/JavaScript with no external dependencies. It's a single-file, zero-build project designed for browser play across desktop and mobile devices.

### Running the Game

Simply open `index.html` in any modern web browser. No build process, server, or dependencies required. The game includes:
- Responsive layout that scales to device screen
- Touch controls for mobile (joystick + fire/bomb buttons)
- Keyboard controls for desktop (arrow keys, WASD, Space to fire, X/Shift for bombs)
- Mouse steering on desktop (move pointer to aim, click to fire)

## Architecture & Design

The entire game logic is contained in one `<script>` block (lines 422–1666) within `index.html`. The architecture follows this pattern:

### Initialization & State
- **Lines 517–527**: Load saved progress from localStorage (best score, difficulty, audio/motion preferences)
- **Global game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` (lines 864–868)
- **Game flow flags**: `running`, `paused`, `gameOver` control the main loop state

### Core Game Loop
- **`loop(now)`** (lines 1640–1653): requestAnimationFrame callback that drives the game
  - Calls `update(dt)` to process game logic
  - Calls `draw()` to render the frame
  - Tracks performance and adapts quality if needed

### Key Systems

#### 1. **Input Handling** (lines 768–861)
- Keyboard: arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: movement to steer, click to fire (desktop primary control)
- Touch: left zone for movement (joystick), right zone for fire/bomb buttons
- **Input priority**: Keyboard/joystick > mouse steering (prevents fighting)

#### 2. **Weapon System** (lines 871–1146)
- Two weapon types: **Vulcan** (spread bullets, short cooldown ~150ms) and **Laser** (piercing, longer cooldown ~100ms)
- Weapons have 3 levels; at level 3, picking up weapon power-up switches to the other type
- `fireWeapon()` creates bullet objects with different spread patterns based on level
- Weapons stored in `player.weapon` and `player.level`

#### 3. **Enemy System** (lines 502–506, 1022–1041, 1210–1228)
- Three enemy types defined in `ENEMY_TYPES`: grunt (red), interceptor (fast, weaving), tank (slow, tanky)
- Enemies spawn based on difficulty settings and spawn timers
- Enemy behavior:
  - Movement: follow base Y trajectory, some weave side-to-side
  - Shooting: random intervals, either spread pattern or aimed at player
  - HP: grunt/interceptor have 1 HP, tanks have 4 HP

#### 4. **Boss System** (lines 508–515, 1043–1056, 1411–1439, 1372–1409)
- **Six boss types** (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage
- Each boss has unique visual design (colors, glow) and attack patterns
- **Boss lifecycle**:
  1. Entering phase: slides down from top
  2. Active phase: moves left/right within bounds, fires attack patterns
  3. Telegraph system: 22-frame warning before each attack pattern
- Attack patterns are stored in `BOSS_PATTERNS` array; each boss has 3 unique patterns that cycle
- Boss HP scales with stage and difficulty multiplier

#### 5. **Combo/Chain System** (lines 1087–1101)
- Killing enemies within 2.2 seconds maintains a chain
- Chain counter increases score multiplier (1x base → up to 10x at 50+ kills)
- Multiplier resets on damage taken or timer expiry
- Visual feedback: combo counter displayed when player reaches 10, 20, 30 kills

#### 6. **Difficulty System** (lines 485–489, 741–750)
- Three presets: Easy (4 lives, 3 bombs, slower enemies), Normal (3 lives, 2 bombs), Hard (3 lives, 2 bombs, faster)
- Affects: enemy spawn rate, enemy speed/accuracy, boss HP multiplier
- Persistent: saved to localStorage after selection

#### 7. **Audio (Procedural Synthesis)** (lines 542–676)
- Uses WebAudio API (OscillatorNode, BiquadFilter, GainNode, StereoPanner)
- No external audio files—all sound effects are synthesized in real-time
- Sfx pool includes: shoot, laser, explode (type-specific), hit, bomb, powerup, boss patterns, UI sounds
- Spatial audio: panning based on X position of sound origin

#### 8. **Visual Effects & Performance Adaptation** (lines 473–483, 1073–1085)
- Adaptive quality system monitors frame times and adjusts rendering quality (full → medium → low)
- Screen shake on damage/bombs (reduced or suppressed if `reducedMotion` enabled)
- Particle system for explosions (density scales with quality)
- Vignette/flash effects for critical health and bomb use
- Reduced motion support respects browser `prefers-reduced-motion` preference

#### 9. **Scoring & Progression** (lines 1087–1101, 1446, 1441–1454)
- Kill base score varies by enemy type (10–35 points)
- Score multiplied by chain multiplier (up to 10x)
- Boss kill awards 500 points
- Stages progress every 18 kills; stage counter resets kill count per boss
- Theme rotates per stage (6 sky themes, cycling)

### Key Constants & Data Structures

- **DIFFICULTIES**: difficulty preset objects with spawn rates, enemy speeds, boss HP multipliers
- **ENEMY_TYPES**: object mapping type keys to properties (width, height, HP, speed multiplier, score value, min stage)
- **BOSSES**: array of 6 boss designs with visual colors, HP multipliers
- **BOSS_PATTERNS**: array of pattern functions for each boss (e.g., radial bursts, aimed spreads, spirals)
- **THEMES**: sky gradient colors and cloud rendering for each stage
- **STORAGE_*** constants: localStorage keys for persistence (best score, muted, difficulty, reduced motion)

## Making Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502–506)
2. Set `minStage` property to control when it spawns
3. Add explosion sound to `EXPLODE_SFX` map if needed (line 678)
4. Update spawn probability in `spawnEnemy()` (lines 1029–1030)

### Adding a New Boss
1. Push new boss design object to `BOSSES` array (lines 508–515)
2. Add corresponding attack pattern function in `BOSS_PATTERNS` (lines 1372–1409)
3. Boss picks patterns based on its index in the BOSSES array

### Tweaking Game Balance
- Enemy spawn timers: `DIFFICULTIES[difficulty].spawnBase/spawnMin` (line 485)
- Enemy damage output: `fireWeapon()` spread count and spacing (lines 1127–1145)
- Boss difficulty: adjust `baseHp` calculation in `spawnBoss()` (line 1046)
- Player speed: `player.speed = 260` (line 877)

### Performance Optimization
- Quality setting degrades particle count, shadow effects, and rendering detail
- If adding new visuals, check `quality` variable (0 = full, 1 = medium, 2 = low) and adjust glow/shadow accordingly

## Storage & Persistence

Game state is ephemeral during play but these are persisted to localStorage:
- `skyfighter.bestScore`: highest score achieved
- `skyfighter.muted`: audio mute state (0/1)
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: accessibility preference (0/1)

## Common Tasks

**To pause/resume**: Press P or Esc, or tap pause button
**To use a bomb**: Press X, Shift, or tap BOMB button (clears all enemy bullets, damages boss)
**To switch weapons**: Collect weapon power-up and cycle through levels
**To view accessibility options**: Click the 🎆 button to toggle reduced motion (disables screen shake, flash, animations)
