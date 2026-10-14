# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Players control a fighter aircraft, destroy enemies across stages, defeat bosses, and manage limited resources (lives and bombs). The game features:

- Three difficulty levels (Easy, Normal, Hard) with persistent storage
- 6 distinct themed stages with cycling visual themes
- Multiple enemy types with different behaviors and boss patterns
- Two weapon systems (Vulcan spread-shot and piercing Laser)
- Power-ups for health, bombs, and weapon upgrades
- Score multiplier based on consecutive kills (chain system)
- Procedural sound design with no external audio assets
- Full keyboard, mouse, and touch controls
- Reduced-motion mode for accessibility

## Getting Started

### Running the Game

The game is self-contained in `index.html`. Simply:
- Open the file in any modern web browser
- Game saves progress (best score, difficulty, audio/motion preferences) to localStorage

### No Build System Required

This is a static HTML file with no external dependencies, bundling, or server required. All CSS and JavaScript are inlined.

## Code Architecture

The codebase is organized as a single large HTML file with three main sections:

### 1. Styling (lines 10-358)

Canvas-based rendering with supporting UI overlays:
- Game canvas: responsive 480x720 base size that scales to fit the viewport
- HUD elements: score, multiplier, weapon status, hearts (lives), bombs
- Overlay screens: title/pause menu, game over screen, difficulty selector
- Accessibility: reduced motion preferences, screen reader live region (#srLive)
- Touch controls: virtual joystick for movement, FIRE/BOMB buttons

### 2. HTML Structure (lines 361-420)

Minimal DOM:
- Single `<canvas id="gameCanvas">` for all game rendering
- HUD stats container with real-time score/lives updates
- Boss health bar (shown only during boss phases)
- Combo/stage banners for visual feedback
- Screen flash and vignette effects (CSS-driven, performance-aware)
- Touch zone overlays for mobile steering and action buttons
- Overlay card for menu/pause/game over

### 3. JavaScript Game Loop (lines 422-1666)

#### Initialization & Constants
- **DIFFICULTIES**: 3 difficulty profiles with spawn rates, enemy behavior, boss HP multipliers
- **THEMES**: 6 visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) that cycle per stage
- **ENEMY_TYPES**: Grunt (fast, weak), Interceptor (fast and nimble), Tank (slow, durable)
- **BOSSES**: 6 unique boss designs with distinct attack patterns; cycle based on stage

#### Core Game State (lines 863-912)
```
player: { x, y, w, h, speed, weapon, level, cooldown, invuln, hp }
enemies, bullets, enemyBullets, powerUps, boss
score, multiplier (chain-based), lives, bombCount, stage, kills
```

#### Input Handling (lines 767-861)
- **Keyboard**: Arrow keys or WASD for movement; Space to fire; X/Shift/P for bomb/pause
- **Mouse**: Move to steer; click to fire; aim reticle shown during gameplay
- **Touch**: Virtual joystick (left side) for movement; FIRE/BOMB buttons (right side)
- Input priority: keyboard/joystick > mouse (so they don't fight)
- Audio context is created lazily on first user interaction

#### Audio System (lines 542-676)
Procedural WebAudio synthesis (no external files):
- **beep()**: Oscillator-based tone generation with frequency sweeps, filters, panning
- **noiseBurst()**: Random noise with low-pass filter envelope
- **sfx object**: Predefined sound effects (shoot, laser, explosions, powerup, boss patterns)
- Master gain and dynamics compressor for balanced output
- Mute toggle persisted to localStorage
- All effects are panned spatially based on source position for stereo field

#### Game Mechanics

**Update Loop** (lines 1148-1302):
1. Player movement: keyboard/joystick/mouse input combined; clamped to canvas
2. Weapon firing: cooldown-based; Vulcan has spread patterns per level, Laser pierces enemies
3. Spawning: enemies spawn at difficulty-scaled intervals; boss spawns after `KILLS_PER_STAGE` (18) kills
4. Enemy behavior:
   - Weaving: interceptors oscillate side-to-side
   - Shooting: spread pattern (5 bullets) or aimed at player
   - Movement: constant downward with difficulty-scaled speed
5. Collision detection: axis-aligned rect overlap for all interactions
6. Chain/multiplier: resets if >2.2s without a kill; multiplier = 1 + floor(chain/5) up to 10x
7. Boss phases: entering animation → attack pattern → death sequence

**Boss Behavior** (lines 1324-1434):
- Boss enters from top, settles at y=120
- Bounces horizontally between screen edges
- Each boss has 3 attack patterns (radial bursts, homing, columns, spirals, etc.)
- Attacks are telegraphed: 22-frame warning, then pattern fires, 85-frame cooldown
- Boss HP scales with difficulty and boss design multiplier
- Bullet types: radial (velocity-based), spread (aimed + angular offset), columns, homing

**Scoring & Progression**:
- Stage advances after defeating boss
- Theme changes per stage (cycles through 6 themes)
- Enemy types unlock at specific stages (tank at stage 2+)
- Difficulty multiplier on enemy spawn rates and boss HP

#### Rendering (lines 1520-1638)

**draw()** called every frame:
- Sky gradient and sun radial glow (theme-colored)
- Scrolling cloud layers (two speeds for parallax)
- Player ship with weapon-dependent visuals (laser aura when equipped)
- Mouse aim reticle if mouse is active
- Bullets: glowing yellow (vulcan) or blue (laser)
- Enemy bullets: red circles with glow
- Enemy ships: painted planes with weave animation; tank HP bars
- Boss ship: multi-part design with rotating engine effects and core glow
- Power-ups: rotating colored squares (red=health, yellow=bomb, green=weapon)
- Explosion particles: spawned on kills, fade and expand
- Screen shake effect: applied to canvas transform if active
- Vignette: red pulsing edge when player has 1 life (critical state)
- Flash overlay: white flash on hit, brighter on bomb detonation

**Performance Awareness**:
- `quality` variable (0=full, 1=medium, 2=low) adjusts based on frame time average
- Glow effects (shadowBlur, shadowColor) disabled at low quality
- Particle counts scale with quality
- Frame counting used for cloud animation and boss core effects

#### Input & Event Handlers (lines 768-862)
- Joystick: captures touch start/move/end; displays visual base and stick
- Fire button: toggled on mousedown/touchstart, off on mouseup/touchend
- Mouse move: continuously updates steering target while canvas is active
- Keyboard: space=fire, X/Shift/P=bomb/pause; prevents default for Space

#### UI State Management
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Reduced motion: honors prefers-reduced-motion media query; disables shakes, flashes, animations
- Difficulty buttons: persist selection; locked during gameplay
- Pause: overlays a menu; can only toggle if not game over

## Key Data Structures

- **Sprite Objects**: All have `{ x, y, w, h, ... }` for AABB collision
- **Bullet**: `{ x, y, w, h, speed, vx?, pierce?, laser? }` (radial-only for enemy bullets: `dx, dy`)
- **Enemy**: `{ x, y, w, h, hp, speed, type, color, pattern, weave, shootTimer, ... }`
- **Boss**: `{ x, y, w, h, hp, maxHp, design, telegraph, patternIndex, phaseTimer, ... }`
- **Particle**: `{ x, y, vx, vy, life, maxLife, color }`
- **PowerUp**: `{ x, y, w, h, speed, type ('life'|'bomb'|'weapon') }`

## Modifying the Game

### Adding a New Boss
1. Add entry to `BOSSES` array with name, hull colors, core color, hpMul
2. Add 3 attack patterns to `BOSS_PATTERNS` array (functions receiving boss object, firing enemy bullets)
3. Boss index matches stage modulo 6

### Adjusting Difficulty
Modify `DIFFICULTIES` object:
- `spawnBase/spawnMin`: control enemy spawn frequency
- `enemySpeed`: multiplier on base enemy movement speed
- `enemyShootChance`: probability enemy fires when eligible
- `lives/bombs`: starting resources
- `bossHpMul`: multiplier on calculated boss HP

### Adding Visual Themes
Add to `THEMES` array:
- `sky`: array of 3 color stops for gradient (top to bottom)
- `cloud`: RGB string for cloud color
- `sun`: RGB string for sun glow
- `name`: display name (used in stage clear announcement)

### Weapon Tuning
Modify `fireWeapon()`:
- **Vulcan**: spread angles, projectile speed, fire rate (fireDelay)
- **Laser**: number of lanes, piercing capability, projectile speed

### Sound Customization
Edit `sfx` object (lines 624-676):
- Adjust frequencies, durations, filter cutoffs in beep() calls
- All parameters exposed as named `opts`

## Performance Considerations

- **Adaptive Quality**: Tracks 45-frame rolling average; downgrades on slow frames
- **Collision**: Simple AABB overlap test; no spatial partitioning (canvas is small enough)
- **Particles**: Capped by quality level; old particles culled
- **Canvas**: Cached context; no texture atlasing needed
- **Audio**: Synth reuses single AudioContext; gain nodes created per-note
- **Rendering**: No canvas state save/restore except for transformations; single fillStyle per shape type where possible

## Browser Compatibility

- Requires: Canvas 2D, requestAnimationFrame, localStorage, ES6 (arrow functions, const/let, template strings)
- WebAudio optional but recommended (game works muted if unavailable)
- Touch input: checked via `ontouchstart` or navigator.maxTouchPoints
- Responsive to viewport resize and orientation change
