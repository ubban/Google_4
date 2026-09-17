# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML/CSS/JavaScript application. It runs entirely in the browser with no external dependencies or build process.

**Key Features:**
- Canvas-based 2D graphics rendering
- Multiple enemy types and boss encounters with unique attack patterns
- Dual weapon system (Vulcan guns and laser)
- Touch/keyboard/mouse controls with adaptive support
- Procedural sound effects via Web Audio API
- Difficulty modes with gameplay scaling
- High score tracking via localStorage
- Accessibility: reduced motion preferences, screen reader support

## Architecture

The codebase is organized as a self-contained JavaScript state machine running inside a `requestAnimationFrame` loop.

### Core Game State (`update()` and `draw()`)
- **Main Loop**: `loop(now)` — called via `requestAnimationFrame` at ~60 FPS
  - Calls `update(dt)` for game logic (collisions, entity movement, spawning)
  - Calls `draw()` for canvas rendering
  - Tracks delta time to handle variable frame rates
  
- **Performance Adaptation**: `quality` variable (0=full, 1=medium, 2=low) adjusts:
  - Particle counts in explosions
  - Shadow blur for glows
  - Cloud/visual effects
  
- **Frame Counter**: `frame` increments every update; used for animations, timing, and boss patterns

### Game Entities

**Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- Controlled via keyboard (WASD/arrows), mouse movement, or touch joystick
- Two weapons: `vulcan` (spread shot) and `laser` (piercing beams)
- Weapon levels 1-3 affect spread/pierce; level 4+ switches weapons
- Invulnerability frames after taking damage

**Enemies**: Array of `{ x, y, w, h, hp, speed, shootTimer, pattern, type, ... }`
- Types: `grunt`, `interceptor`, `tank` — defined in `ENEMY_TYPES`
- Behaviors: weaving movement, aimed/spread shooting
- Spawn rate increases with difficulty and frame progression
- Stage gates: `minStage` controls when enemy types appear

**Boss**: Single `{ x, y, hp, maxHp, entering, telegraph, patternIndex, design, ... }`
- Spawns after `KILLS_PER_STAGE` (18) enemies killed
- 6 unique designs (`BOSSES` array) cycle per stage
- Each boss has 3 attack patterns in `BOSS_PATTERNS` array
- Telegraph phase (22 frames) before pattern fires

**Bullets & Enemy Bullets**
- Player bullets: `{ x, y, vx, vy, w, h, speed, pierce, laser }`
  - Laser bullets pierce enemies and can be upgraded
- Enemy bullets: `{ x, y, w, h, speed, vx, vy }` or radial `{ dx, dy }`
  - Radial bullets move via velocity; normal bullets move vertically

**Power-ups**: `{ x, y, w, h, speed, type }` — types: `life`, `bomb`, `weapon`

**Particles**: `{ x, y, vx, vy, life, maxLife, color }` — visual effects for explosions

### Difficulty System

Difficulty profiles in `DIFFICULTIES` scale:
- Enemy spawn frequency
- Enemy movement speed
- Enemy shooting behavior
- Available lives and bombs
- Boss health multiplier

Stored in localStorage; selected before each game via UI buttons.

### Audio (Web Audio Synth)
- **No external assets** — all sounds synthesized with oscillators, filters, noise
- `beep(freq, dur, type, vol, opts)` — sine/square/sawtooth/triangle
- `noiseBurst(dur, vol, opts)` — white noise with filtering
- `sfx` object maps sound names to functions: `shoot`, `laser`, `explode*`, `hit`, `bomb`, `powerup`, `bossTelegraph`, etc.
- Muting stored in localStorage

### Input Handling

**Keyboard/Mouse:**
- WASD/Arrows: move
- Space/Mouse click: fire
- X/Shift: bomb
- P/Esc: pause
- Mouse movement: auto-steer toward pointer

**Touch:**
- Left 55% of screen: joystick for movement (visual base + stick)
- Right 45% of screen: Fire button (large tap zone) and Bomb button
- Joystick constrained to 50px radius

**Priority**: Keyboard/joystick always overrides mouse steering when active

### Canvas Sizing
- Base resolution: 480x720 (portrait)
- Scales to fit window (clamped at 1.6x) while maintaining aspect ratio
- Uses CSS transform for scaling to keep canvas crisp
- `resize()` called on window resize/orientation change

### Visual Themes

Six stages cycle through visual themes defined in `THEMES`:
- Sky gradient (3 color stops)
- Cloud color and sun glow
- Applied dynamically based on `stage`

### Rendering System

**Draw Order** (bottom to top):
1. Sky gradient + sun glow
2. Cloud layers (parallax effect at different speeds)
3. Player ship (with optional laser aura)
4. Player bullets (with glow if `quality` is high)
5. Enemy bullets (red circles)
6. Enemies with optional health bar (tanks)
7. Boss ship (if present)
8. Power-ups (rotating squares)
9. Particles (expanding/fading circles)
10. Screen shake applied via canvas translate

**Screen Effects:**
- `flashHit()` / `flashBomb()` — white flash overlay with CSS animations
- `screenShake(magnitude, duration)` — random canvas translation
- `vignette` — red pulse when critical health (1 life)

## Common Development Tasks

### Running the Game
1. Open `index.html` in any modern browser
2. Click "Start Game" to begin
3. No build step required

### Testing Specific Features

**Test a boss pattern:**
- Edit `spawnBoss()` to force a specific stage: `stage = 3;`
- Or directly set `boss.hp = 1` to make boss easier to test

**Adjust difficulty balance:**
- Modify `DIFFICULTIES` object values
- Test with `KILLS_PER_STAGE` reduced (e.g., 6 instead of 18)

**Add a new enemy type:**
1. Add to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update spawn probabilities in `spawnEnemy()`
3. Add explosion effect to `EXPLODE_SFX` mapping

**Add a new boss pattern:**
1. Add function to BOSS_PATTERNS section (e.g., `fireCustom(b, ...)`)
2. Add to boss design's pattern array in `BOSS_PATTERNS`
3. Patterns fire on telegraph countdown; use helper functions like `fireRadial()`, `fireAimedSpread()`

### Tweaking Weapon Balance

**Vulcan (bullet spray):**
- Fire rate: adjust `fireDelay` constant in `update()`
- Spread: modify `spread` array in `fireWeapon()`
- Damage: modify bullet initialization speed (affects damage on hit)

**Laser (beam weapon):**
- Fire rate: separate `fireDelay` for laser in `update()`
- Pierce: modify `pierce` starting value (3 allows piercing 3 enemies)
- Visual: modify `#40c4ff` colors in draw loop

### Adding Visual Effects

**Glow/bloom for objects:**
- Use `ctx.shadowColor` and `ctx.shadowBlur` (only when `quality === 0`)
- Example: Player ship glows blue around it

**Screen shake calibration:**
- Adjust `screenShake(mag, dur)` calls throughout code
- Bigger values: more violent shake
- Duration: how long shake persists

**Particle effects:**
- `spawnExplosion(x, y, color, count)` creates burst of particles
- Adjust `count` for intensity
- Particles use velocity-based physics (vx, vy each frame)

### Accessibility Considerations

- **Reduced Motion**: CSS animations disabled when `prefers-reduced-motion: reduce` is set
- **Screen Reader Support**: `#srLive` element announces key game events
- **Color**: Avoid red-only feedback (combine with icons, text)
- **Touch**: Virtual controls become visible on touch devices

### Performance Tuning

**Monitor frame time:**
- `trackPerf(dt)` tracks last 45 frame times
- If average > 1/40 (25ms), `quality` increases (fewer effects)
- If average < 1/56 (17.9ms), `quality` decreases (more effects)

**Reduce particle count or disable parallax clouds** if targeting lower-end devices.

## Key Constants & Configuration

- `BASE_W / BASE_H`: Canvas dimensions (480x720)
- `KILLS_PER_STAGE`: Enemies killed before boss spawn (18)
- `STORAGE_*`: localStorage keys for persistence
- Boss HP: `70 + stage * 35` times difficulty/boss multipliers

## File Structure

```
index.html  — Single file containing:
  - HTML structure (canvas, overlay UI, touch controls)
  - CSS (styles, animations, responsive layout)
  - JavaScript (game logic, rendering, audio, input handling)
README.md   — Basic project description
CLAUDE.md   — This file
```

## Known Limitations & Future Improvements

- **No high score global leaderboard** (only localStorage)
- **No pause during boss telegraph** (intentional for tension)
- **Mobile audio** may require user gesture before first sound (Web Audio constraint)
- **No replay/spectate** mode
