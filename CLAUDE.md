# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML application. The game features:
- Player-controlled aircraft with weapon upgrades
- Multiple enemy types (Grunt, Interceptor, Tank) with distinct behaviors
- 6 unique boss encounters per stage with pattern-based attacks
- Score multiplier and combo system
- Touch and keyboard/mouse input support
- Procedurally themed stages with different visual themes
- Web Audio API synthesized sound effects (no external audio assets)

## Running the Game

Simply open `index.html` in any modern web browser. No build process, npm packages, or server setup required. The game saves preferences to localStorage (best score, difficulty, mute state, reduced motion preference).

## Game Architecture

The entire game logic is contained in a single IIFE (Immediately Invoked Function Expression) starting at line 422. The architecture consists of several key systems:

### Game State
- **Core entities**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Metrics**: `score`, `lives`, `kills`, `stage`, `chain`, `multiplier`
- **Flags**: `running`, `paused`, `gameOver`

### Main Loop (line 1640)
```
loop() → update(dt) → draw()
```
The frame loop calls `update()` for game state, then `draw()` for rendering. Frame rate is capped at 50ms per frame (line 1644).

### Core Systems

**Input Handling** (lines 767–861)
- Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- Mouse: Pointer position steers the player; click to fire
- Touch: Left area for movement joystick, right buttons for fire and bomb
- Input priority: Keyboard/joystick > mouse (line 1164)

**Physics & Collision** (lines 1188–1295)
- Simple AABB collision detection: `rectsOverlap()` (line 1069)
- Bullet cleanup happens each frame (lines 1189, 1192)
- Collision checks: enemy bullets vs. player, enemies vs. player, bullets vs. enemies/boss, powerups vs. player

**Weapon System** (lines 1124–1145)
- **Vulcan**: Spread shots (1, 2, or 3 bullets depending on level)
- **Laser**: Piercing shots that can hit multiple enemies
- Weapon upgrade powerups cycle through Vulcan levels 1–3, then switch to Laser levels 1–3, then back to Vulcan
- Fire rate: Vulcan 0.15s, Laser 0.1s

**Boss System** (lines 508–515, 1043–1056, 1411–1453)
- 6 boss designs with distinct color schemes and HP multipliers
- Boss patterns: 3 attack patterns per boss defined in `BOSS_PATTERNS` (lines 1372–1409)
- Boss enters stage when 18 enemy kills are reached
- Telegraph phase (22 frames) before each attack pattern fires
- Boss health bar displayed during fights

**Difficulty Settings** (lines 485–489)
Three presets control: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier.

**Audio Synthesis** (lines 542–676)
- Web Audio API with no external assets
- `beep()`: Oscillator-based sounds with optional frequency sweep and filter
- `noiseBurst()`: Procedural noise for explosions
- Sound effects for all game events: shoot, laser, explosions, hits, bombs, boss telegraph, stage clear, game over

**Visual Effects** (lines 726–730, 1073–1085)
- Screen shake: `screenShake()` applies random translation
- Flash overlay: Hit flashes (white) and bomb flashes (brighter white)
- Combo popup: Displays on 10-kill milestones
- Particle system: Explosion particles with velocity and fade

**Difficulty Progression**
- Spawn rate increases over time (line 1201)
- Enemy speed and accuracy scale with difficulty
- Boss difficulty via HP multiplier
- 18 kills per stage required to spawn boss

**Responsive Design** (lines 752–765)
- Canvas scales to fit viewport (max 1.6x)
- Touch controls enabled on touch devices
- Reduced motion preference respected via `prefers-reduced-motion` media query
- Safe area insets applied for notched devices

## Important Implementation Details

### Enemy Spawning (lines 1022–1041)
- Enemies spawn with random horizontal position, above canvas
- Enemy types filtered by `minStage` (e.g., tanks only spawn stage 2+)
- Enemies can weave left/right (sine wave pattern for interceptors)
- Two shoot patterns: spread (3 bullets in arc) or aimed (single bullet at player)

### Power-up System (lines 1058–1067, 1304–1322)
- Life: +1 hp (6% spawn chance)
- Bomb: +1 bomb (8% spawn chance)
- Weapon: Upgrade weapon (20% spawn chance)
- All drop from killed enemies

### Chain & Multiplier (lines 1087–1101)
- Chain increments on every kill, resets if 2.2 seconds elapse
- Multiplier: 1x + floor(chain / 5) (max 10x at 45+ kills)
- Breaks on taking damage

### Theme System (lines 493–500)
- 6 alternating visual themes (sky gradients, cloud colors, sun colors)
- Cycles through one per stage
- Affects only visuals, not gameplay

### Adaptive Quality (lines 474–483)
- Tracks average frame time over 45 frames
- 3 levels: 0 (full quality, with shadow effects), 1 (reduced particle count), 2 (minimal particles, no shadows)
- Automatically scales down if avg frame > 25ms, scales up if < ~18ms

## Rendering Pipeline

1. **Background**: Sky gradient, sun glow, parallax clouds (2 layers)
2. **Player**: Plane sprite with cockpit, wings, flames when firing; laser aura when using laser weapon
3. **Bullets**: Yellow squares (Vulcan) or cyan squares (Laser) with glow
4. **Enemy Bullets**: Red circles with glow
5. **Enemies**: Plane sprites with colors matching type; health bar for tanks
6. **Boss**: Detailed hull/wing/core design with telegraph glow
7. **Power-ups**: Rotating colored squares (red, yellow, or green)
8. **Particles**: Expanding circles with alpha fade
9. **HUD**: Score, best, multiplier, weapon, hearts, bombs (top center)
10. **Boss bar**: Name and health bar (top center when boss active)
11. **UI**: Pause button, mute, motion toggle (top right)

Mouse crosshair shown if mouse is active during gameplay.

## Key Game Constants

- Canvas: 480×720 logical pixels
- Player speed: 260 px/s
- Kills per stage: 18
- Weapon fire rates: Vulcan 0.15s, Laser 0.1s
- Boss pattern telegraph: 22 frames
- Invulnerability after hit: 1.5s
- Chain timeout: 2.2s
- Max lives: 3–4 (depending on difficulty)
- Max bombs: 2–3 (depending on difficulty)

## LocalStorage Keys

- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (0/1)

## Testing & Iteration

Since this is a gameplay-focused project with no external dependencies:
- **Testing**: Open in browser, play through stages, test controls on keyboard and touch
- **Tweaking**: Adjust constants in `DIFFICULTIES`, `BOSS_PATTERNS`, `THEMES`, etc. and reload
- **Performance**: Check adaptive quality scaling; watch browser dev tools for frame times
- **Accessibility**: Verify reduced-motion setting disables animations; test keyboard controls without mouse

## Code Style Notes

- Single-file IIFE pattern to avoid global scope pollution
- Objects for game entities (player, enemies, bullets) with simple property bags
- Array operations (forEach, filter) for entity management
- Canvas 2D API for all rendering
- No external libraries or frameworks
