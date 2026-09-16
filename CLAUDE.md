# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML5 file. The game features player aircraft combat, multiple enemy types, boss battles, weapon upgrades, and procedurally varied enemy attack patterns.

## Running the Game

To play the game, simply open `index.html` in a web browser. The game runs standalone with no external dependencies or build step.

- **Desktop**: Arrow keys or WASD to move, Space/left-click to fire, X or Shift to use bombs, P/Esc to pause
- **Mobile**: Drag left side to fly, tap FIRE and BOMB buttons

## Architecture & Code Structure

The entire game is contained in a single `index.html` file (~1670 lines) with:
- **Embedded CSS** (lines 10–358): Styling for HUD, overlay UI, buttons, animations, and accessibility
- **Embedded JavaScript** (lines 422–1666): Game logic in a single IIFE (immediately invoked function expression)

### Key Game Systems

#### Canvas & Rendering
- **Canvas Setup** (lines 753–765): Base resolution 480x720, scales to fit window while maintaining aspect ratio
- **Draw Loop** (lines 1520–1637): Renders background with animated clouds, player, enemies, boss, bullets, particles using `requestAnimationFrame`
- **Screen Shake** (lines 726–729): Procedural shake effect for impact feedback

#### Game State Management
- **Main State** (lines 864–869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Player State** (line 877): Position, weapon, level, invulnerability frames, cooldown
- **Stage/Difficulty** (lines 485–489): Three difficulty presets (easy/normal/hard) control spawn rates, enemy speed, boss HP multiplier
- **Scoring System** (lines 1087–1101): Score with combo multiplier (increases every 5 kills, resets after 2.2s without a kill)

#### Input Handling (lines 768–861)
- **Keyboard**: Listened on keydown/keyup; Space = fire, arrows/WASD = move, P/Esc = pause, X/Shift = bomb
- **Touch**: Two-zone touch controls on mobile—left 55% for movement (joystick), right 45% for fire/bomb buttons
- **Mouse**: Click to fire, move to steer (only active when hovering canvas)

#### Enemy System (lines 1022–1040, 1210–1227)
- **Enemy Types**: `grunt` (fast, 1 HP), `interceptor` (faster, 1 HP), `tank` (slow, 4 HP)
- **Spawning**: Difficulty-based spawn rate that increases over time; enemies have 50/50 chance of aimed vs. spread shot patterns
- **Behavior**: Simple movement downward; weave pattern on interceptors; periodic shooting at player or in spread pattern

#### Boss System (lines 1043–1055, 1372–1439)
- **Boss Designs**: 6 bosses cycle through stages (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with unique colors and attack pattern sets
- **Attack Patterns**: Each boss has 3 attack patterns (e.g., radial burst, aimed spread, dense walls). Telegraphed before firing.
- **Boss Phases**: Entering animation, then bounces left/right while cycling through pattern attacks

#### Weapon System (lines 1124–1145, 1304–1321)
- **Vulcan Gun**: Rapid fire, bullet spread increases with level (1: single, 2: dual, 3: triple)
- **Laser**: Slower, higher damage, pierces enemies based on level
- **Power-up Transitions**: Collecting weapon power-ups upgrades level; at level 3, next pickup switches to opposite weapon at level 1

#### Audio (lines 542–676)
- **WebAudio Synth**: No external audio files; all effects synthesized using oscillators and noise
- **SFX**: Individual functions for shoot, laser, hit, bomb, powerup, boss telegraph, stage clear, game over, etc.
- **Pan & Filtering**: Sound effects pan and filter based on position on screen or context
- **Mute Storage**: Mute state persists in localStorage

#### UI & HUD
- **Overlay Screens** (lines 401–419): Main menu (difficulty select), pause screen, game-over screen with stats
- **Live HUD** (lines 367–374): Score, best score, multiplier, weapon name/level, hearts, bombs
- **Boss Bar** (lines 376–379): Shows boss name and health during boss fights
- **Accessibility**: Screen reader live region (`#srLive`) announces important game events

#### Visual Theming
- **Sky Themes** (lines 493–500): 6 theme sets cycle through stages, each with unique sky gradient, cloud color, sun color
- **Particles & Effects**: Explosions spawn directional particles; vignette effect on critical health; flash on hits

#### Adaptive Quality (lines 473–482)
- **Performance Tracking**: Measures frame time and adjusts quality (0=full, 1=medium, 2=low)
- **Quality Usage**: High quality adds shadow glows; lower quality reduces particle counts and disables shadows

#### Persistence (lines 517–527, 463–466)
- **localStorage Keys**:
  - `skyfighter.bestScore`: Best score ever achieved
  - `skyfighter.muted`: Mute toggle (0 or 1)
  - `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
  - `skyfighter.reducedMotion`: Respects `prefers-reduced-motion` media query

### Common Modification Points

#### Adjust Difficulty Scaling
**DIFFICULTIES** object (lines 485–489): Tune `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

#### Add/Modify Enemy Types
Add to **ENEMY_TYPES** object (lines 502–506): Define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`

#### Customize Boss Designs
**BOSSES** array (lines 508–515): Each boss has colors, HP multiplier, and name. Attack patterns assigned by index in **BOSS_PATTERNS** (lines 1372–1409)

#### Change Weapon Behavior
**fireWeapon()** function (lines 1124–1145): Modify bullet spread, speed, pierce logic

#### Tweak Power-up Spawn Rates
**maybeSpawnPowerUp()** function (lines 1058–1067): Adjust probability thresholds for life/bomb/weapon drops

#### Modify Visual Effects
- **Screen Shake**: Adjust magnitude in `screenShake()` calls or `shakeMag` multiplier
- **Animations**: CSS `@keyframes` (lines 71–127) for heart beat, combo pop, stage banner, etc.
- **Colors**: Inline `fillStyle` or gradient definitions throughout draw functions

## Code Navigation

| Concept | Lines |
|---------|-------|
| Storage keys | 463–466 |
| Audio init & synth | 542–676 |
| Difficulty/balance data | 485–515 |
| Canvas setup | 753–765 |
| Input handling | 768–861 |
| Game state init | 864–911 |
| Main update loop | 1148–1302 |
| Boss update logic | 1411–1439 |
| Draw/render loop | 1520–1637 |
| Animation frame loop | 1640–1665 |

## Browser Requirements

- Modern browser with Canvas 2D, WebAudio API, `localStorage`
- Touch support detected automatically; joystick UI appears on touch devices
- No polyfills or transpilation needed (ES6+)

## Tips for Development

- Game state is not persisted (only best score, settings, difficulty)—game always starts fresh
- The frame counter (`frame`) is the source of truth for animation timing
- All physics use `dt` (delta time in seconds) for frame-rate-independent movement
- Boss telegraph uses visual effect + sound cue before pattern fires
- The combo chain system incentivizes continuous gameplay (breaks on 2.2s inactivity)
