# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) running in a single self-contained HTML file. No build tools, dependencies, or external assets—just open `index.html` in a browser to play.

## Running the Game

- **Local browser**: Open `index.html` directly in any modern web browser
- **Web server**: Serve via `python3 -m http.server`, `npx http-server`, or equivalent
- No build step, no npm install, no compilation needed

## Code Architecture

The entire game logic lives in a single `<script>` block in `index.html` (~1300 lines). It's organized into logical sections:

### Core Systems

**Rendering Pipeline**
- Canvas (480×720px base, scales to fit window)
- 2D context drawing: planes, bullets, enemies, particles, UI overlays
- Adaptive performance tracking: adjusts particle/effect quality based on frame time (`quality` variable: 0=full, 1=medium, 2=low)

**Game Loop**
- `loop()`: requestAnimationFrame-driven at ~60fps
- `update(dt)`: physics, collision, spawning (delta-time aware)
- `draw()`: render all entities
- Frame counter for animations and timing

**Input Handling**
- Keyboard: arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: move to steer, click to fire; takes priority over keyboard when active
- Touch: left side for movement (virtual joystick), right side for action buttons
- Joystick state: `joyActive`, `joyVec` track active touch input

**Audio (Web Audio API Synthesis)**
- No external sound files; all SFX generated with oscillators, noise, filters
- `beep()`: tone generation with frequency sweeps and filter modulation
- `noiseBurst()`: white/filtered noise for explosions and impacts
- `sfx` object: named sound effects (shoot, laser, explode, hit, bomb, etc.)
- Master gain + dynamics compressor for limiting

### Game State

**Player**
- Position, speed (260 px/s), size (40×40)
- Weapon system: vulcan (spread shot) vs laser (pierce shot), 3 levels each
- Lives, invulnerability timer, cooldown between shots

**Enemies**
- 3 types: grunt (red), interceptor (orange, fast, weaving), tank (purple, tanky)
- Spawn based on difficulty with adaptive throttling
- Stage-gated: tanks don't appear until stage 2+
- AI: can weave, shoot aimed or spread patterns
- Hull HP, color, scoring value per type

**Bosses**
- 6 unique designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has 3 unique attack patterns (radial bursts, aimed spreads, telegraphed attacks)
- Telegraph mechanic: 22-frame warning before each attack volley
- HP scales with difficulty and design multiplier

**Scoring & Progression**
- Chain multiplier: increases for consecutive kills within 2.2 seconds
- Stage progression: 18 kills per stage → boss spawn
- Difficulty: easy (4 lives, 3 bombs), normal (3 lives, 2 bombs), hard (3 lives, 2 bombs)
- Enemy spawn rate throttles down as frames pass (ramps difficulty mid-stage)

**Visual Effects**
- Screen shake: magnitude + duration, reduced to 15% if reduced-motion enabled
- Flash overlay: brief white flash on player hit or bomb detonation
- Vignette: red pulse when player at 1 HP
- Particles: explosion clouds with physics (gravity-free arc trajectories)
- Combo pop-up, stage banner, boss health bar

### Power-Ups

- **Life** (red): Restore 1 HP (6% drop chance)
- **Bomb** (yellow): +1 smart bomb stock (8% drop chance)
- **Weapon** (green): Level up current weapon or switch (20% drop chance)
  - Vulcan: level 1 (center), level 2 (side), level 3 (triple spread)
  - Laser: level 1 (center), level 2 (dual), level 3 (triple + pierces)

### Collision Detection

- `rectsOverlap()`: AABB collision for all entities
- Player-bullet → enemy: reduce HP, spawn explosion, award score
- Enemy bullet → player: deal damage, trigger invulnerability iframe
- Bomb → all enemies + boss: instant kill with damage multiplier
- Power-up → player: immediate effect

### Theme & Aesthetics

- 6 sky themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) rotate by stage
- Planes drawn procedurally with 2D path + fill operations (no sprites)
- Boss designs use gradient fills and animations
- Cloud parallax background with two layers at different speeds
- HUD uses glass-morphism style (backdrop blur, semi-transparent panels)

### Persistence (localStorage)

- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio on/off toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User motion preference override

## Key Development Patterns

**Difficulty Balancing**
- `DIFFICULTIES` object: tweaks spawn rate, enemy speed, shoot frequency, lives, bombs, boss HP
- Spawn throttling: `spawnTimer = Math.max(d.spawnMin, d.spawnBase - difficultyRamp)`
- Boss patterns are difficulty-agnostic; HP multiplier is the main lever

**Boss AI**
- Telegraph → attack pattern → wait → repeat
- Patterns are pure functions: `BOSS_PATTERNS[designIndex][patternIndex](boss)`
- Movement: horizontal back-and-forth, bounces at edges
- Bullet patterns: radial, aimed spread, homing, columns, spirals, crosses

**Quality Adaptation**
- `trackPerf(dt)`: rolling average of last 45 frame deltas
- If avg > 25ms: reduce quality (skip particles, dim shadows)
- If avg < 18ms: restore quality
- Used in particle spawning and shadow rendering

**Accessibility**
- Reduced motion: disables animations, reduces screen shake to 15%, removes pulse effects
- Screen reader live region for score updates and events
- High contrast HUD with text shadows for readability

## Common Modifications

**Adjust Difficulty**
- Edit `DIFFICULTIES` constants for spawn timings, speeds, lives, bombs, boss HP multipliers
- Boss patterns are in `BOSS_PATTERNS` array (one sub-array per boss)

**Add a New Enemy Type**
- Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
- Optional `weave: true` for sinusoidal horizontal movement
- Update spawn logic in `spawnEnemy()` if needed

**Add a New Boss**
- Add object to `BOSSES` array with colors and HP multiplier
- Add attack pattern array to `BOSS_PATTERNS` (same index)
- Name appears automatically in stage banner

**Tweak Weapon Balance**
- Vulcan: adjust spread angles in `fireWeapon()`, bullet speed/pierce
- Laser: pierce count, damage per shot (currently 3 per hit vs vulcan 1)
- Power-up drop rates in `maybeSpawnPowerUp()`

**Sound Design**
- Edit `sfx` object: adjust frequencies, durations, filter types/cutoffs
- `beep()` opts: `sweepTo` (pitch slide), `filterFreq`, `filterType`, `pan`
- `noiseBurst()` opts: `filterFreq` (higher = brighter), `pan`

**Visuals**
- Plane drawing: modify `drawPlane()` path commands
- Boss drawing: modify `drawBossShip()` gradient + ellipse positions
- Background themes: edit `THEMES` array (sky colors, cloud/sun colors)
- HUD styling: CSS in `<style>` block uses CSS variables and animations

## File Structure

- **index.html** (single file)
  - `<head>`: metadata, favicon (inline SVG), styles (dark theme, animations, responsive)
  - `<body>`: canvas container, HUD, UI overlays (start menu, pause, game over)
  - `<script>`: entire game engine (input, physics, rendering, audio, AI)

No separate asset files; all content is self-contained.

## Browser Compatibility

- Requires: Canvas 2D, Web Audio API, localStorage, requestAnimationFrame
- Tested on modern Chrome, Firefox, Safari, Edge
- Mobile: touch controls enable automatically if `ontouchstart` detected
- Reduced motion: respects `prefers-reduced-motion` media query
