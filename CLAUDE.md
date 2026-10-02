# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application (index.html). The game features player-controlled aircraft, enemy waves with various types, boss encounters, weapon progression, and procedural difficulty scaling.

## Architecture

The codebase is organized as a single 1670-line HTML file with embedded CSS and JavaScript. All game state, logic, and rendering are contained within an IIFE (Immediately Invoked Function Expression) that starts at line 423.

### Core Systems

**Game Loop & Rendering** (lines 1640–1666)
- Main `loop()` function runs via `requestAnimationFrame` for 60 FPS target
- `update(dt)` handles all game logic
- `draw()` renders to canvas using 2D context
- Adaptive performance tracking adjusts visual quality based on frame times

**Input Handling** (lines 767–862)
- Keyboard: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Mouse: Plane auto-follows cursor when hovering canvas
- Touch: Virtual joystick on left, action buttons on right
- Keyboard input takes priority over mouse (no fighting)

**Entity Management**
- `player`: Position, speed, weapon, level, invunerability frames
- `enemies`: Array with various types (grunt, interceptor, tank) that spawn in waves
- `boss`: Single active boss with health, movement, and attack patterns
- `bullets` / `enemyBullets`: Arrays for projectiles
- `powerUps`: Dropped from enemies (life, bomb, weapon upgrade)
- `particles`: Short-lived explosion effects

**Canvas & Responsiveness** (lines 753–765)
- Base resolution: 480×720
- Scales up to fit window (max 1.6x) while maintaining aspect ratio
- Responsive to resize and orientation change events

### Game State Variables

- `score`, `multiplier`: Score system with combo chains (line 868)
- `lives`, `maxLives`: Player health; loss ends game
- `bombCount`, `maxBombs`: Limited-use smart bombs that damage/clear enemies
- `stage`: Increases after each boss; affects enemy types and theme
- `chain`, `chainTimer`: Tracks consecutive kills for multiplier
- `running`, `paused`, `gameOver`: Game state flags

### Key Game Constants

**Difficulties** (lines 485–489)
- Easy / Normal / Hard adjust spawn rates, enemy speed, shoot chance, lives/bombs, and boss HP
- Stored in `localStorage` under `skyfighter.difficulty`

**Themes** (lines 493–500)
- 6 environment themes with sky gradients, cloud colors, and sun styling
- Cycle through stages: Daylight Front → Sunset Corridor → Night Approach → etc.

**Enemy Types** (lines 502–506)
- `grunt`: Red, slow, 1 HP, score 10 (all stages)
- `interceptor`: Orange, fast weaver, 1 HP, score 15 (all stages)
- `tank`: Purple, slow, 4 HP, score 35 (stage 2+)

**Bosses** (lines 508–515)
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has unique attack patterns; cycle through stages
- HP scales with stage, difficulty, and boss-specific multiplier

### Weapon System

**Vulcan** (lines 1126–1134)
- Spread increases at levels 2 and 3
- Always hits once; no pierce
- Shorter cooldown (0.15s)

**Laser** (lines 1136–1145)
- Lanes widen at level 2 and 3 (can stack with level 3)
- Pierce through enemies up to pierce count
- Longer cooldown (0.1s actually faster, but 60 RPM vs 400 RPM for Vulcan at close range)
- Longer cooldown (0.1s) actually means faster fire rate; ignore line 1177 comment

### Boss Attack Patterns

Each boss has 3 distinct patterns in `BOSS_PATTERNS` (lines 1372–1409), triggered on a telegraph (line 1421). Patterns include:
- **Radial bursts**: 360° bullet rings
- **Aimed spreads**: Cone toward player
- **Columns**: Vertical wall spam
- **Spirals & crosses**: Symmetrical formations
- **Homing volleys**: Bullets that track player
- **Double rings**: Expanding concentric rings
- **Teleport barrages**: Boss reposition before attacking

### Audio System (lines 542–676)

**Web Audio Synthesis**—no external assets:
- `beep()`: Oscillator with frequency sweep and filter
- `noiseBurst()`: Procedural noise with lowpass filtering
- All sound effects defined in `sfx` object
- Muted state stored in `localStorage`
- Master gain routed through dynamics compressor for consistent level

**SFX Events**:
- Weapon fire, hits, explosions (vary by enemy type)
- Bomb, powerup, boss telegraph, boss defeat, stage clear, game over

### Screen Effects

- **Screen shake**: `screenShake(magnitude, duration)` adds random canvas offset
- **Flash**: Hit/bomb animations via CSS, respects reduced motion
- **Vignette**: Red pulsing edge when critical (1 life left)
- **Reduced Motion**: Accessibility toggle respects system preference and user override

### Scoring & Progression

- `addScore(baseScore)` applies multiplier and updates chain
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain breaks if no kills within 2.2 seconds
- `KILLS_PER_STAGE = 18`: Boss spawns after 18 kills
- Each boss kill: +500 points
- Best score stored in `localStorage`

## Development

### Running the Game
- Open `index.html` in any modern browser
- No build step, no dependencies, no server required
- Works offline

### Testing Changes
1. Edit `index.html`
2. Reload browser (Cmd/Ctrl+R)
3. Start a game and verify behavior
4. Use browser DevTools console to inspect game state (all variables accessible via window scope inside IIFE)

### Common Tasks

**Adjust Enemy Difficulty**
- Modify `DIFFICULTIES` object (line 485) to change spawn rates, speed, accuracy
- Adjust `KILLS_PER_STAGE` to tune stage progression pacing

**Add/Tweak Boss Patterns**
- Edit `BOSS_PATTERNS` array (line 1372)
- Each pattern is a function that calls fire helpers: `fireRadial()`, `fireAimedSpread()`, etc.
- Adjust counts and speeds for challenge

**Balance Weapon Progression**
- Weapon level unlock thresholds in `applyPowerUp()` (line 1311)
- Weapon stats: cooldown (line 1177), spread/lanes (lines 1127, 1136), pierce (lines 1141)

**Add Visual Feedback**
- Particle effects via `spawnExplosion()` (line 1073)
- Screen shake via `screenShake()` (line 727)
- UI flashes via `flashHit()` / `flashBomb()` (lines 698, 704)
- Combo pop text via `showComboPop()` (line 712)

**Add Sound Effects**
- New effects in `sfx` object (line 624)
- Use `beep()` for tones, `noiseBurst()` for percussion
- Remember to check `muted` flag

### Accessibility

- Full keyboard + touch support
- Screen reader announcements via `announce()` helper
- Reduced motion toggle disables animations per `prefers-reduced-motion`
- All UI has visible labels and ARIA roles

### Performance

- Adaptive quality system (variable `quality`) scales rendering at 0/1/2 levels
- Disabled shadow blur and particle count on lower quality
- All collision detection is simple AABB (`rectsOverlap()`)
- Canvas resize scales up rendered output without re-rendering

## Browser Compatibility

- Requires ES6 (arrow functions, template literals)
- WebAudio API for sound (gracefully degrades if unavailable)
- Canvas 2D context
- `localStorage` for persistence
- Touch Events API for mobile
- `matchMedia` for reduced-motion detection
