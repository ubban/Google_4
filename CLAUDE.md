# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (markup, styles, and logic) is contained in `index.html`. Players navigate stages of increasing difficulty, defeat bosses, collect power-ups, and build combo multipliers.

## Running the Game

The game runs in any modern web browser:
- **Local testing**: Open `index.html` directly in a browser, or serve with a simple HTTP server:
  ```bash
  python3 -m http.server 8000
  # Then visit http://localhost:8000
  ```
- **Mobile**: Works on touch devices with an adaptive joystick and action buttons
- **Keyboard**: Arrow keys or WASD to move, Space/Shift+X to fire/bomb, P/Esc to pause

## Code Architecture

The game is structured as a single vanilla JavaScript IIFE (Immediately Invoked Function Expression) with the following logical sections:

### 1. **HTML Structure & Styling** (lines 1–359)
- Game canvas (480x720px, responsive scaling)
- HUD elements: score, hearts, bombs, weapon status, boss health bar
- Overlay system for title screen, pause, and game-over screens
- Touch controls: joystick zone and action buttons
- Screen effects: vignette (low health), flash (hit/bomb), combo pop, stage banner

### 2. **Game State** (lines 863–870)
Globals tracking:
- Player, bullets, enemies, enemy bullets, particles, power-ups, boss
- Score, lives, stage, combo chain, multiplier, spawn timer
- Game state flags: `running`, `paused`, `gameOver`

### 3. **Difficulty & Content Configuration** (lines 485–515)
- `DIFFICULTIES`: Four difficulty tiers with spawn rates, enemy speed, and boss HP multipliers
- `THEMES`: Six sky themes with gradient colors and cloud rendering
- `ENEMY_TYPES`: Three enemy archetypes (grunt, interceptor, tank) with HP, score, stage unlock
- `BOSSES`: Six boss designs with unique attack patterns and color schemes
- `BOSS_PATTERNS`: Pattern arrays defining each boss's attack sequences

### 4. **Audio System** (lines 542–676)
Web Audio API synthesizer generating all sound effects procedurally (no external files):
- `beep()`: Tone generation with frequency sweep, filtering, and panning
- `noiseBurst()`: Noise-based effects (explosions, bomb blasts)
- `sfx` object: Named effects called throughout gameplay (shoot, explode, hit, bomb, etc.)
- Muting is localStorage-persistent

### 5. **Input System** (lines 767–861)
- Keyboard events (arrow keys, WASD, Space, X/Shift for bomb, P/Esc for pause)
- Mouse/touch controls: Movement via mouse pointer or touch joystick
- Touch-specific handlers for multi-touch joystick on mobile
- Firing state managed by `firing` flag (mouse down, touch button, or Space held)

### 6. **Game Loop & Rendering** (lines 1520–1665)
- `draw()`: Canvas rendering of sky gradient, clouds, entities, particles
- `update(dt)`: Physics and game state advancement
- `loop(now)`: RAF-driven main loop with delta-time tracking and performance monitoring

### 7. **Entity Systems**
- **Player** (lines 1148–1182): Movement, firing cooldown, weapon level, invulnerability
- **Bullets** (lines 1188–1189): Player bullets with optional pierce (laser only)
- **Enemies** (lines 1210–1228): Spawn waves, movement (including weaving patterns), AI shooting
- **Enemy Bullets** (lines 1191–1192): Projectiles in radial or aimed patterns
- **Power-ups** (lines 1194–1195): Life, bomb, weapon upgrades
- **Boss** (lines 1299–1301): Multi-pattern attack sequences, telegraph before attacks, phased movement

### 8. **Collision & Scoring**
- `rectsOverlap()` (line 1069): AABB collision detection
- `addScore()` (line 1087): Adds combo chain logic and multiplier calculation
- `breakChain()` (line 1096): Resets combo when player takes damage
- Boss patterns spawn specific bullet formations (radial, aimed spread, columns, homing, etc.)

### 9. **Stage Progression**
- Stages unlock new enemy types at `minStage`
- After 18 kills (`KILLS_PER_STAGE`), a boss appears
- Boss defeat advances stage and increases base enemy HP multiplier
- Themes cycle across 6 variants

### 10. **Performance Adaptation** (lines 473–483)
Quality levels (0 = full, 1 = medium, 2 = low) adjust:
- Particle spawn count
- Glow/shadow effects (shadowBlur on canvas)
- Automatically downgrade if frame time exceeds 1/40s, upgrade if below 1/56s

### 11. **State Persistence** (lines 517–523)
localStorage keys for:
- `skyfighter.bestScore`: High score (never resets on page reload)
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference for motion animations

## Key Technical Patterns

### Weapon Systems
Two weapons with level progression:
- **Vulcan**: Bullet spread increases at levels 2 & 3 (1 → 2 → 3 shots)
- **Laser**: Fire rate faster (0.1s vs 0.15s), pierce enemies, lane configuration at levels 2 & 3

Power-up collection switches weapons and resets level to 1.

### Boss Attack Patterns
Each boss has three unique attack patterns; the pattern index cycles on each telegraph phase. Patterns use helper functions:
- `fireRadial()`: Bullets radiating equally in all directions
- `fireAimedSpread()`: Fan of bullets aimed at player with spread angle
- `fireColumns()`: Vertical or horizontal columns
- `fireSpiralBurst()`: Rotating circular pattern
- `fireHomingSeeded()`: Aimed bullets with small randomization
- `fireDoubleRing()`: Two concentric rings at different speeds

Boss movement bounces horizontally; telegraph phase (visual telegraphing) precedes attack.

### Canvas Rendering Optimization
- Conditional glow/shadow rendering based on quality level
- Particle alpha blended out as they expire
- Enemy health bars only for tank type
- Invulnerability blink on player during cooldown

### Accessibility Features
- Screen reader live region (`#srLive`) for game announcements
- Reduced motion toggle respects `prefers-reduced-motion` and disables animations/screenshake
- Touch controls automatically enabled on touch devices

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` filter to include new type in stage-appropriate conditions
3. Add explosion SFX entry in `EXPLODE_SFX` if custom sound desired
4. No rendering changes needed—`drawPlane()` works for all types

### Adding a New Boss
1. Add boss entry to `BOSSES` array with `name`, hull/wing/core colors, `hpMul`
2. Define three attack patterns in a new sub-array of `BOSS_PATTERNS` (same index order)
3. Patterns receive boss object and call `fireRadial()`, `fireAimedSpread()`, etc. as needed
4. Stage progression automatically cycles bosses via modulo

### Adding a Theme
Add to `THEMES` array with `sky` (3 gradient stops), `cloud` and `sun` (RGB strings), and `name`. Themes cycle automatically by stage.

### Testing Difficulty Balance
Modify `DIFFICULTIES` entries to adjust spawn rates, enemy speed, or boss HP. Test against each difficulty to ensure progression feels right at spawn rate and enemy damage.

### Audio Adjustments
Modify `beep()` or `noiseBurst()` calls in `sfx` object. Parameters:
- `freq`: Oscillator frequency (Hz); use sweep for pitch slide effects
- `vol`: Gain amplitude (0–0.3 typical)
- `opts.sweepTo`: Target frequency for exponential ramp
- `opts.filterFreq`: Cutoff for lowpass/highpass filter
- `opts.pan`: Stereo pan (-1 left to +1 right)

## Notes on Code Style

- Heavy use of object literals for configuration (difficulty, enemy types, themes, boss patterns)
- Entity state managed as plain objects, no classes—collision detection uses AABB via `rectsOverlap()`
- Drawing uses Canvas 2D API directly; no libraries or frameworks
- All assets (sounds, particles, graphics) generated at runtime—no external files required
- Event listeners added inline; no event delegation or manager pattern
- Screen shake applied to canvas via `ctx.translate()`; visual effects avoid excessive DOM manipulation
