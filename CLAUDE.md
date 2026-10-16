# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file. The game features procedurally generated sound effects via WebAudio, a progressive difficulty system with bosses, multiple weapon types, and adaptive performance scaling.

## Architecture

The entire game is contained in `index.html` (1670 lines) with three integrated layers:

### HTML Structure
- Game canvas (480×720 logical resolution, scales to fit viewport)
- Heads-up display (HUD) showing score, lives, bombs, weapon status
- Overlay modal for menus (start screen, pause, game over)
- Touch controls overlay (virtual joystick and buttons for mobile)
- Accessibility elements (screen reader live region)

### CSS Styling (lines 10–358)
- Dark theme with glassmorphism effects (backdrop-filter)
- Responsive typography using clamp() for all sizes
- Animations for UI feedback: combo pop, stage banner, card entrance, heartbeat (critical health)
- Reduced-motion preference respected; animations disabled when `body.reduced-motion` is set
- Safe area insets for notched devices

### JavaScript Game Engine (lines 422–1666)

**Core Game Loop** (line 1640–1652)
- `requestAnimationFrame(loop)` drives the render at monitor refresh rate
- `update(dt)` processes input, entity movement, collision detection, spawning
- `draw()` renders the scene with screen shake, parallax clouds, and glow effects

**State Management**
- `player`: position, dimensions, weapon type/level, invulnerability cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: entity arrays
- `score`, `lives`, `chain`, `multiplier`: gameplay metrics
- `boss`: null until a stage progresses through `KILLS_PER_STAGE` (18 kills)
- Difficulty settings stored in localStorage; three difficulty tiers affect spawn rates, enemy speed, and boss HP

**Core Mechanics**

1. **Weapon System** (lines 1124–1146)
   - Vulcan: spread bullets (1, 2, or 3 depending on level)
   - Laser: piercing beams (1, 2, or 3 lanes depending on level)
   - Power-ups cycle through levels 1→3, then toggle weapon type
   - Weapon cooldowns: vulcan 0.15s, laser 0.1s

2. **Enemy Spawning** (lines 1022–1041)
   - Three enemy types: grunt (weak, fast spawning), interceptor (fast, weaves), tank (slow, high HP)
   - Types unlock progressively by stage
   - Spawn rate and enemy stats scale with difficulty
   - Enemies fire in two patterns: spread (arc) or aimed (at player)

3. **Boss System** (lines 1043–1056, 1411–1439)
   - One boss per stage (cycles through 6 designs with unique attack patterns)
   - Boss enters from top, then phases between telegraph and attack states
   - Telegraph period (22 frames) gives player warning; 6 different attack pattern functions per boss
   - Attack patterns: radial bursts, aimed spreads, columns, spirals, crosses, homing shots
   - Boss HP scales with stage, difficulty, and boss type multiplier

4. **Chain/Multiplier System** (lines 1087–1101)
   - Each kill increments chain; chainTimer resets to 2.2s
   - Multiplier = 1 + floor(chain / 5), capped at 10×
   - Chain breaks if no kills land within 2.2s; visual feedback with combo popups every 10 kills

5. **Collision Detection** (line 1069–1071)
   - AABB (axis-aligned bounding box) overlap: `abs(dx) < (w1 + w2) / 2 && abs(dy) < (h1 + h2) / 2`
   - Player bullets vs enemies: damage, pierce, knockback particles
   - Enemy bullets vs player: damage, invulnerability period
   - Enemies/boss vs player: contact damage

6. **Input Handling**
   - Keyboard: Arrow keys or WASD to move, Space to fire, X/Shift/P/Esc for bomb/pause
   - Mouse: movement aims player (glides toward pointer); click to fire
   - Touch: left zone for movement (virtual joystick), right zone has FIRE/BOMB buttons
   - Input priority: keyboard/joystick > mouse (no interference between modes)

7. **Performance Adaptation** (lines 474–483)
   - Tracks frame time over 45 frames; adaptive quality: 0 (full glow), 1 (medium), 2 (low)
   - Particle count, shadow blur, and effects scale with quality
   - User can manually enable reduced motion (toggles animations, reduces shake intensity)

8. **Audio** (lines 542–676)
   - WebAudio context created on first user interaction (gesture requirement)
   - No external audio files; all sound effects procedurally generated using oscillators and noise
   - SFX bank: shoot, laser, explode (grunt/interceptor/tank), hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, uiClick, gameover
   - Mute toggle persisted in localStorage

**Rendering Pipeline** (lines 1520–1638)
- Sky gradient and sun glow (varies by stage theme; 6 themes cycle)
- Parallax clouds at two depths
- Player plane with weapon glow (laser-mode blue aura)
- Mouse reticle when moving mouse over canvas
- Glowing bullets (yellow vulcan, cyan laser)
- Enemy planes (colored by type)
- Boss ship (gradient hull, core glow with telegraph flash, engine lights)
- Power-up icons rotating
- Explosion particles with decay
- Vignette pulse during critical health (red, reduced on reduced-motion)
- Screen shake (random offset, magnitude/duration scale with quality)
- Flash (white overlay on hit, brightness on bomb)

## Development Notes

### No Build or Test Process
- This is a standalone HTML file; open `index.html` directly in a browser to run.
- No npm, webpack, or build tools required.
- No external dependencies (uses browser APIs only).

### Code Organization (by line range)
- **10–358**: CSS (layout, animations, dark theme)
- **463–466**: Storage keys (best score, mute, difficulty, reduced motion)
- **473–483**: Adaptive quality tracking
- **485–516**: Game configuration (DIFFICULTIES, KILLS_PER_STAGE, THEMES, ENEMY_TYPES, BOSSES, BOSS_PATTERNS)
- **542–676**: Audio system (WebAudio context, beep/noiseBurst, SFX bank)
- **680–710**: HUD rendering (hearts, bombs, flash feedback)
- **726–750**: Pause, mute, motion toggle, difficulty selection UI
- **753–765**: Canvas sizing (BASE_W=480, BASE_H=720)
- **768–861**: Input handling (keyboard, mouse, touch, joystick)
- **864–912**: Game state variables and reset function
- **913–1020**: Rendering functions (drawPlane, drawBossShip, drawClouds)
- **1022–1041**: Enemy spawning logic
- **1043–1056**: Boss spawning and progression
- **1058–1067**: Power-up spawning
- **1069–1101**: Collision, scoring, chain system
- **1103–1122**: Bomb mechanic
- **1124–1146**: Weapon firing
- **1148–1302**: Main update loop (player movement, entity movement, collision resolution, spawn management, boss updates, particle cleanup)
- **1304–1322**: Power-up application (life, bombs, weapon progression)
- **1324–1409**: Boss attack pattern functions (fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, etc.)
- **1411–1439**: Boss AI update loop
- **1441–1454**: Boss defeat and stage progression
- **1456–1489**: Life loss and game over
- **1491–1504**: Pause toggle
- **1507–1637**: Frame-by-frame drawing and screen shake
- **1640–1665**: Animation loop and start button handler

### Key Constants and Tuning

**Gameplay Balance**
- `KILLS_PER_STAGE = 18`: kills needed before boss warning spawns
- Weapon cooldowns: vulcan 0.15s (6.67 RPS), laser 0.1s (10 RPS)
- Boss HP formula: `baseHp * difficultyMul * designMul` (baseHp = 70 + stage × 35)
- Power-up drop rates: life 6%, bomb 8%, weapon 20%
- Difficulty parameters: easy (slowest spawning, weakest enemies), normal (balanced), hard (fastest spawning, strongest enemies)

**Canvas and Scaling**
- Logical resolution: 480×720 (portrait)
- Canvas scales up to 1.6× max; responsive to window size
- Safe area insets for notched devices

**Performance**
- Particle limit: ~60 on quality 0, ~36 on quality 1, ~20 on quality 2
- Screen shake disabled when `reducedMotion` is true
- Glow effects (shadowBlur) disabled on quality > 0

### Editing Guidelines

**Adding a new enemy type:**
1. Add entry to `ENEMY_TYPES` object with shape `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. Optionally add to `EXPLODE_SFX` if it should have custom explosion sound
3. Update `spawnEnemy()` logic to include it in choices based on `minStage`

**Adding a new boss:**
1. Push entry to `BOSSES` array with `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Add corresponding attack pattern array to `BOSS_PATTERNS`
3. Boss selection is `BOSSES[(stage - 1) % BOSSES.length]`

**Adding a new stage theme:**
1. Push entry to `THEMES` with `{ sky: [color1, color2, color3], cloud, sun, name }`
2. Themes cycle: `THEMES[(stage - 1) % THEMES.length]`

**Adding a new SFX:**
1. Define in `sfx` object using `beep()` or `noiseBurst()` helpers
2. `beep(freq, dur, type, vol, opts)` creates pitched sounds; `noiseBurst(dur, vol, opts)` creates noise
3. Options: `{ sweepTo, filterType, filterFreq, pan, delay }`
4. Call `sfx.soundName()` from gameplay code

**Modifying difficulty:**
- Edit `DIFFICULTIES` object: `{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }`
- Higher `spawnBase`/`spawnMin` = slower spawning (more lenient)
- `enemySpeed` and `enemyShootChance` compound with each other

### Testing / Debugging Tips

- Open browser DevTools console; errors log there
- `localStorage.clear()` resets all saved data (best score, difficulty, mute state)
- Modify `DIFFICULTIES` or boss HP directly in browser console to test balance
- Frame rate visible in quality adaptation; observable via particle density and glow effects
- Screen shake test: use bomb (Space/click X)
- Collision debug: keep a hitbox visible by editing `draw()` to render enemy/bullet rects

### Browser Compatibility

- Requires: ES6 (arrow functions, const/let, template literals)
- Canvas 2D context
- WebAudio API (audio feature gracefully degrades if unavailable)
- localStorage (for settings persistence)
- matchMedia (for prefers-reduced-motion)
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)

### Accessibility

- Screen reader live region (`#srLive`) announces game events (lives lost, stage changes, score records)
- Reduced motion preference auto-detected via `prefers-reduced-motion: reduce`; can be toggled manually
- Keyboard and touch controls fully supported
- Color contrast meets WCAG guidelines (dark theme, high-contrast text)
