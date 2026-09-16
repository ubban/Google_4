# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sky Fighter: Raiden-Style Shoot-'em-up

Sky Fighter is a browser-based vertical shoot-'em-up game with procedurally generated audio, multiple difficulty levels, and progressive stage-based boss encounters.

## Running the Game

The entire game is a single HTML file (`index.html`). Since WebAudio API and full browser features require HTTPS or `localhost`, serve it via a local HTTP server:

```bash
# Python 3
python -m http.server 8000

# Node.js (npx)
npx http-server

# Ruby
ruby -run -ehttpd . -p8000
```

Then open `http://localhost:8000/index.html` in a browser.

No build step, no dependencies, no tests. The file runs as-is.

## Code Architecture

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) starting at line 422. All code is vanilla JavaScript with no external libraries.

### Core Game Loop (Line 1640)

```javascript
function loop(now) { ... requestAnimationFrame(loop); }
```

The `loop` function runs ~60 FPS via `requestAnimationFrame`. It:
1. Calculates `dt` (delta time in seconds) from wall clock
2. Calls `update(dt)` to advance game state
3. Calls `draw()` to render the frame

Rendering is entirely Canvas 2D (`ctx`).

### State Management

All mutable game state lives in the outer scope (lines 864–869). Key variables:

- **`player`**: Position, speed, weapons, invulnerability timer
- **`enemies`, `boss`**: Array of active enemies + current boss ship
- **`bullets`, `enemyBullets`**: Arrays of projectiles
- **`score`, `lives`, `stage`, `stageKills`**: Progression/scoring
- **`chain`, `multiplier`**: Combo system (increases multiplier every 5 kills, resets after 2.2s no kills)
- **`running`, `paused`, `gameOver`**: State flags

### Enemy Spawning & Waves (Line 1197)

```javascript
if (!boss) {
  spawnTimer -= dt * 60;
  if (spawnTimer <= 0) { spawnEnemy(); spawnTimer = Math.max(d.spawnMin, d.spawnBase - difficultyRamp); }
  if (stageKills >= KILLS_PER_STAGE && !bossWarned) { ... spawnBoss(); }
}
```

- Enemies spawn continuously until 18 kills (`KILLS_PER_STAGE`) are reached
- After 18 kills, boss spawns after a brief delay
- Spawn rate increases over time via `difficultyRamp` (every 300 frames)

### Boss System (Lines 1411–1434, 1372–1409)

Bosses have:
1. **Entry animation**: Fly down from top at 60px/s
2. **Movement pattern**: Bounce left-right at variable speed, ~55px/s horizontal
3. **Attack pattern system**: Each boss (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has 3 attack patterns. They cycle through patterns with a telegraph phase (22 frames before firing) to warn the player
4. **Telegraph & Fire**: `boss.telegraph > 0` means telegraphing; when it reaches 0, fire pattern and reset timer

Each boss has a unique color and damage multiplier (`hpMul`). Patterns vary from radial bursts to homing missiles.

### Weapon System (Lines 1124–1146)

- **Vulcan**: Spread shot. Level 1 (1 bullet), Level 2 (2 bullets), Level 3+ (3 bullets)
- **Laser**: Piercing beam. Level 1 (1 lane), Level 2 (2 lanes), Level 3 (3 lanes with extra pierce)

Power-ups randomly spawn from defeated enemies. Weapon power-ups cycle through levels and switch between Vulcan/Laser.

### Collision Detection (Line 1069)

```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```

Bounding-box collision: compares distance between centers vs. half-widths/heights.

### Difficulty & Progression (Lines 485–489)

Three difficulty levels with scaling:
- `spawnBase` / `spawnMin`: Enemy spawn rate (lower = faster)
- `enemySpeed`: Multiplier on enemy movement
- `enemyShootChance`: Probability enemy fires at player
- `lives`, `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Audio System (Lines 542–676)

**No external assets.** Uses WebAudio API to synthesize all sounds:
- **`beep(freq, dur, type, vol, opts)`**: Sine/square/sawtooth tone with optional frequency sweep, filter, panning
- **`noiseBurst(dur, vol, opts)`**: White noise with low-pass filter
- **`sfx.*`**: Named sound effects (shoot, laser, explode variants, bomb, powerup, etc.)

All audio is gated by the `muted` flag and the audio context's suspended state (requires user interaction to start).

### Visual Systems

#### Canvas Sizing (Lines 752–765)
Base resolution is 480×720 px (portrait). Scales up to fit viewport while maintaining aspect ratio, max 1.6× magnification.

#### Sky Themes (Lines 493–500)
Six parallax sky backgrounds with gradient layers and cloud animation. Themes cycle by stage.

#### Screen Shake (Lines 726–730)
```javascript
function screenShake(mag, dur) {
  shakeMag = reducedMotion ? mag * 0.15 : mag;
  shakeTime = reducedMotion ? Math.min(dur, 0.1) : dur;
}
```

Applied in `draw()` via canvas translation; respects reduced motion preference.

#### Particles (Lines 1073–1085)
Explosions spawn particles with random velocity and 0.5s lifetime. Rendered as circles that fade.

### Performance Adaptation (Lines 473–483)

Tracks average frame delta over 45 frames. If frame time drifts:
- Avg > 1/40th sec (25 FPS) → lower quality (reduce particle/effect count)
- Avg < 1/56th sec (56 FPS) → raise quality

Quality setting affects particle counts, shadow blur, and glow effects.

### Input Handling (Lines 768–861)

**Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)

**Mouse**: Pointer position drives player position smoothly (not instant). Click/hold to fire.

**Touch**: 
- Left 55% of screen: Joystick input (swipe to move)
- Right bottom 45%: Fire and Bomb buttons (tap)
- Joystick base appears under finger when dragging

Joystick uses a 50px radius deadzone (`maxDist`).

### UI & Overlay (Lines 400–419, 1469–1505)

- **HUD** (line 367): Score, best score, multiplier, weapon, lives, bombs (always visible)
- **Boss bar** (lines 376–379): Shows boss name and health when fighting
- **Overlay** (lines 401–419): Menu card for start/pause/end screens
- **Combo pop** (line 381): Displays at kill milestones (10, 20, 30...)
- **Stage banner** (line 382): Shows stage progression

## Key Patterns & Important Notes

1. **No external assets**: All visuals are Canvas 2D paths and rects. All audio is synthesized.
2. **Stateless drawing**: `draw()` is pure—computes everything from `frame` counter and current state.
3. **LocalStorage persistence**: Best score, difficulty, mute state, reduced-motion preference
4. **Reduced motion**: Respects `prefers-reduced-motion` system preference and disables animations/shake when enabled
5. **Single-file**: No imports, no bundler. Minification would reduce ~59KB to ~20KB but isn't needed for localhost play
6. **Frame-based timing**: `frame` counter increments each loop—used for animation frames, visual effects timing
7. **Accessible**: Screen reader announcements (`srLive` element), keyboard controls, reduced motion support

## Modifying the Game

### Adding an Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502–506)
2. Add `minStage` to cap when it appears
3. Update `spawnEnemy()` (line 1026) if new conditions needed
4. Add explosion SFX to `EXPLODE_SFX` map (line 678)

### Adding a Boss
1. Add to `BOSSES` array (line 508–515) with colors and name
2. Add attack patterns to `BOSS_PATTERNS` array (line 1372) as a 3-element array of functions
3. Boss is assigned `(stage - 1) % BOSSES.length`, so order cycles

### Tweaking Difficulty
Edit `DIFFICULTIES` object (line 485). Lower spawn values = more enemies. Adjust `enemySpeed`, `enemyShootChance`, or HP multipliers.

### Adding a Sky Theme
Add to `THEMES` array (line 493). Themes are picked by `(stage - 1) % THEMES.length`.

### Sound Design
Edit or extend the `sfx` object (line 624). Each SFX is a function that calls `beep()` or `noiseBurst()` with parameters. Filter frequencies and sweep targets control timbre.

## Testing / Validation

- **No automated tests.** Play the game: test each difficulty, all weapon combos, boss patterns, edge cases (low HP, bomb spam, rapid stage progression)
- **Performance**: Monitor frame rate in DevTools. Check quality scaling under load
- **Accessibility**: Keyboard controls, screen reader announcements, reduced motion mode
- **Mobile**: Test touch controls on various screen sizes. Joystick and button zones should be comfortable

## Browser Compatibility

Requires:
- Canvas 2D API
- WebAudio API (optional but needed for sound)
- `requestAnimationFrame`
- ES6 (arrow functions, `const`/`let`, template literals)

Works on all modern browsers (Chrome, Firefox, Safari, Edge) desktop & mobile.
