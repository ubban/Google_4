# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas and vanilla JavaScript. It's a single-file implementation (`index.html`) with no build process, external dependencies, or server required — simply open the file in a browser to play.

### Key Features
- Three difficulty levels (Easy, Normal, Hard) with progressive difficulty ramping
- Two weapon types: Vulcan (spray) and Laser (piercing)
- Boss encounters with unique bullet patterns per boss
- Smart bomb mechanic that clears the screen
- Combo/chain multiplier for consecutive kills
- WebAudio API-based procedural sound synthesis (no audio files)
- Mobile touch controls with virtual joystick and buttons
- Responsive canvas scaling for any screen size
- Accessibility features: screen reader announcements, reduced motion support
- LocalStorage for high score persistence across sessions

## File Structure

- **index.html** — Complete game implementation (~1670 lines)
  - CSS styling (responsive, dark theme, animations, mobile layout)
  - HTML markup (canvas, HUD, overlay menu, touch controls)
  - JavaScript game logic (everything below)

## Architecture & Game Systems

### Game Loop
- Main loop runs via `requestAnimationFrame(loop)` at ~60fps
- Each frame: `update(dt)` handles game logic, `draw()` renders, `trackPerf()` monitors performance
- Time step capped at 50ms to prevent large jumps on frame misses
- Adaptive quality system: reduces particle count under performance strain

### Core Game Objects

**Player**
- Controlled ship with position (x, y), movement speed (260 px/s)
- Two weapons: `vulcan` (single/spread shots) and `laser` (piercing beams)
- Weapon levels: 1–3 controlling spread/beam count
- Invulnerability timer after taking damage (1.5s)
- Fire cooldown: 150ms (vulcan), 100ms (laser)

**Enemies** (spawned continuously based on difficulty curve)
- **grunt**: Basic enemy, 1 HP, appears from stage 1
- **interceptor**: Fast, 1 HP, weaves across screen, stage 1+
- **tank**: Slow, 4 HP, tougher, stage 2+
- All spread or aim shots at player; weaving interceptors follow a sine wave

**Bosses** (one per stage after 18 enemy kills)
- 6 boss designs with unique visual styles and attack patterns (stored in `BOSSES[]` array)
- Each boss has 3 attack patterns cycling continuously
- Boss patterns fire radial bursts, aimed spreads, homing shots, columns, etc.
- Telegraph phase (22 frames of warning) before attack
- HP scales by difficulty and boss design

**Projectiles**
- Player bullets: track pierce count (laser only), move upward
- Enemy bullets: move downward; radial bullets track dx/dy instead of speed
- Removed when off-screen

**Power-ups** (6% life, 8% bomb, 20% weapon drops)
- Spawn on enemy death, drift downward
- Types: +life, +bomb, weapon upgrade

**Particles** (explosion effects)
- Adaptive count: quality=0 → 14 particles, quality=2 → ~5 particles
- Each tracks position, velocity, life time

### Game State Machine

**States**: `running`, `paused`, `gameOver`
- Game starts in overlay screen (menu/pause overlay)
- Clicking "Start" transitions to `running`
- Escape/P key toggles pause (shows overlay)
- Final screen shows score, stage reached, best score

**Progression**
- `stage` increments after each boss defeat
- `stageKills` counter triggers boss spawn at 18 kills
- Difficulty settings (`DIFFICULTIES` object) control: enemy spawn rate, speed, shoot chance, HP multipliers
- Theme cycles through 6 visual themes (sky gradients, cloud colors)

### Difficulty & Progression

Three preset difficulty levels tweak these parameters:
```
easy:   spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, bossHpMul: 0.8, lives: 4, bombs: 3
normal: spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, bossHpMul: 1.0, lives: 3, bombs: 2
hard:   spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, bossHpMul: 1.3, lives: 3, bombs: 2
```

Spawn rate ramps dynamically: `spawnTimer = max(spawnMin, spawnBase - difficultyRamp)` where `difficultyRamp` increases every 300 frames.

### Scoring & Combo System

- Enemy kills add points: grunt (10), interceptor (15), tank (35), boss (500)
- **Combo**: consecutive kills within 2.2s window increment `chain` counter
- **Multiplier**: `1 + floor(chain / 5)` caps at 10x
- Breaks when player takes damage
- Every 10th kill shows combo pop animation

### Collision Detection

Axis-aligned bounding box (AABB) overlap check:
```
rectsOverlap(a, b) → |a.x - b.x| < (a.w + b.w) / 2 && |a.y - b.y| < (a.h + b.h) / 2
```

Checked each frame for:
- Player bullets ↔ enemies (3 damage per hit, pierce-through for laser)
- Player bullets ↔ boss (3 damage)
- Enemies ↔ player (collision damage + loseLife)
- Enemy bullets ↔ player (loseLife)
- Boss ↔ player (loseLife, only when not entering)
- Power-ups ↔ player (apply bonus)

### Audio System

WebAudio synthesis with no audio files — all procedural:
- **Oscillators** (square, sine, sawtooth, triangle) with frequency sweeps
- **Filters** (lowpass, highpass) for tonal shaping
- **Noise bursts** for impact/explosion sounds
- **Panning** based on on-screen X position for spatial audio
- Master dynamics compressor for consistent volume

Each game event has an `sfx` function: shoot, laser, hit, explode (enemy-type-specific), powerup, boss patterns, game over, etc.

Muting state persists in localStorage; disabled during audio context suspension.

### Input System

**Keyboard**
- Arrow keys / WASD: movement
- Space / click: fire
- X / Shift / click FIRE button: weapon fire
- P / Escape: pause
- Buttons top-right: mute, toggle motion effects, pause

**Mouse** (desktop)
- Move over canvas: ship glides toward cursor
- Click: fire + movement target
- Priority: keyboard/joystick overrides mouse when active

**Touch** (mobile)
- Left 55% of screen: draggable joystick for movement
- Right 45%: FIRE and BOMB buttons
- Joystick displays visual feedback (base + stick)

### Visual Effects

- **Screen shake**: magnitude and duration params; reduced by 85% in reduced-motion mode
- **Flash overlay**: white full-screen fades (0.25s hit, 0.5s bomb) with CSS keyframes
- **Vignette pulse**: red edges pulse when health is critical (1 life)
- **Combo pop**: yellow text animation at 30% screen height
- **Stage banner**: blue glow text at 42% height, 2s display
- **Boss bar**: gradient health bar below top HUD, fades in/out
- **Particle explosions**: burst from impact location, drift and fade
- **Depth layers**: z-index 0–11 manage stacking (game, HUD, overlays, touch controls)

### Canvas Rendering

Base canvas: 480×720px, scaled to fit window (max 1.6x)
- CSS applies `max-width: 100%; max-height: 100%` to maintain aspect ratio
- Safe area insets applied for notches/bezel on mobile
- Crosshair cursor when hovering canvas
- High dpi handled by browser scaling

**Render order**
1. Sky gradient (stage theme)
2. Sun glow overlay
3. Scrolling cloud layers (2 depths, parallax)
4. Player ship (blink during invuln, glow if laser equipped)
5. Mouse steering indicator (if active)
6. Player bullets (with glow on high quality)
7. Enemy bullets (red circles)
8. Enemy ships
9. Boss ship (if active)
10. Power-ups (rotating, color-coded)
11. Particle effects
12. Screen shake offset applied to all above

### Storage & Preferences

LocalStorage keys (no private data, all client-side):
- `skyfighter.bestScore`: best score ever achieved
- `skyfighter.muted`: audio mute state (0 or 1)
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: reduced motion preference (0 or 1)

Best score persists indefinitely; others last per-browser session or until cleared.

## Common Development Tasks

### Testing the Game
1. Open `index.html` directly in a browser (no server needed)
2. Click difficulty buttons to test each level
3. Start game; test player movement with:
   - Mouse: hover and drag pointer
   - Keyboard: arrow keys, WASD
   - Touch: drag left side for joystick
4. Test fire with: spacebar, mouse click, FIRE button
5. Test bomb with: X, Shift, BOMB button
6. Test pause with: P or Escape
7. Verify audio: click mute/unmute button (speaker icon)
8. Test reduced motion: click motion icon (disables shake/flash/animations)

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: w, h, hp, speedMul, color, dark, score, minStage, weave (optional)
2. Update `spawnEnemy()` probability logic to include new type
3. Add explosion sound in `EXPLODE_SFX` if unique
4. Define sfx function in `sfx` object if needed

### Adding a New Boss
1. Add boss config to `BOSSES` array with: name, hullTop/Bottom colors, wing color, core color, hpMul
2. Create a 3-element array of attack pattern functions in `BOSS_PATTERNS`
3. Boss pattern functions call helpers like `fireRadial()`, `fireAimedSpread()`, etc.
4. Rendering in `drawBossShip()` — update if visual is different

### Modifying Difficulty Settings
Edit `DIFFICULTIES` object. Spawn rate is most impactful; tweaking by ±10 is usually enough. Test across a full run (5+ stages) to feel balance.

### Adding a Visual Theme
1. Add entry to `THEMES` array with: sky (3 gradient stops), cloud RGB, sun RGB, name
2. Themes auto-cycle per stage: `THEMES[(stage - 1) % THEMES.length]`
3. Used in `draw()` for gradient, cloud colors, sun glow

### Performance Optimization
- Reduce `frameTimes.length` window if performance tracking is too sensitive (line 478)
- Lower quality threshold: quality rises if `avg > 1/40` (25ms); lower to `1/50` to be stricter
- Particle count scales with quality; adjust multiplier in `spawnExplosion()` (line 1074)

## Key Code Sections & Line Numbers

| Section | Lines | Purpose |
|---------|-------|---------|
| HTML & CSS | 1–359 | UI layout, HUD, overlays, responsive styling |
| Canvas setup & input | 423–862 | Keyboard, mouse, touch, canvas sizing |
| Game state & difficulty | 863–489 | `DIFFICULTIES`, `ENEMIES`, `BOSSES`, `THEMES` |
| Audio synthesis | 542–676 | `beep()`, `noiseBurst()`, `sfx` object |
| UI rendering | 680–730 | Hearts, bombs, flash, combos, stages |
| Reset & main loop | 751–870 | `resetGame()`, `loop()`, frame update |
| Update logic | 1148–1302 | Player movement, collision, spawning |
| Boss logic | 1372–1439 | Boss patterns, telegraph, attack firing |
| Drawing | 1520–1638 | All sprite & effect rendering |

## Debugging Tips

- Open browser DevTools console (F12) to check for errors
- `announce()` function sends screen-reader text — check it for game state messages
- Quality system adjusts automatically; force it: `quality = 0` (high) or `quality = 2` (low) in console
- Pause and step through using `paused = true` in console, then manually call `update(0.016)` per frame
- Inspect game state: log `player`, `enemies`, `boss`, `score`, `multiplier` to trace issues

## Notes for Future Work

- Currently single-file for simplicity; splitting into modules would help for larger expansion
- Boss patterns are hardcoded arrays; data-driven pattern system could ease iteration
- Touch controls are fixed zones; mobile gamepad API could add hardware support
- Particle pool could be pre-allocated for zero-allocation updates
- Canvas rendering is immediate mode; spatial partitioning (quadtree) would help if enemy/bullet counts balloon
