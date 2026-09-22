# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file with no external dependencies. The game runs in any modern browser and features:

- Player-controlled aircraft defending against enemy squadrons
- 6 boss encounters with unique attack patterns
- 3 difficulty levels (Easy, Normal, Hard)
- Dual weapon system (Vulcan cannons, Laser)
- Smart bomb power-ups
- Stage progression system
- Synthesized sound effects using Web Audio API
- LocalStorage-based high score and settings persistence
- Touch and keyboard/mouse controls

## Development Setup

No build process or external dependencies required. Simply open `index.html` in a web browser to play.

### Running the Game

**Desktop:** Double-click `index.html` or open in browser
**Development server (optional):** `python3 -m http.server 8000` then visit `http://localhost:8000`
**Mobile:** Serve via HTTPS; game requires Web Audio API and touch support

### No Tests or Linting

This is a single-file game with no test suite or linting configuration. Changes are validated by playing through the game manually, checking:
- Player movement and controls work correctly
- Weapons fire and damage registering
- Enemy spawning and difficulty scaling
- Boss patterns executing
- Power-ups collecting and applying
- UI updates (HUD, overlays, pause menu)
- Audio playback (if unmuted)
- Performance on lower-end devices

## Code Architecture

All code exists in a single `<script>` block in `index.html` (lines 422–1666). It follows a game loop pattern with clear responsibility separation:

### Global Initialization (Lines 423–527)

- DOM element references and LocalStorage keys
- Initial state (best score, mute, difficulty, reduced motion preference)
- Audio context setup deferred to first user interaction

### Audio System (Lines 542–676)

**Web Audio Synthesis** — no external audio files. Synth functions:
- `beep(freq, dur, type, vol, opts)` — oscillator-based tones with optional pitch sweep and filtering
- `noiseBurst(dur, vol, opts)` — filtered white noise for explosions
- `sfx` object — predefined sound effects (shoot, explode, bomb, powerup, boss patterns, UI click, etc.)

Uses Web Audio compressor for dynamic range management. Pan effects position sounds left/right based on sprite position on canvas.

### Input Handling (Lines 767–862)

Three input methods with priority order (only one active per frame):
1. **Keyboard** — Arrow keys / WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
2. **Touch joystick** — Left zone for movement, right zone for fire/bomb buttons
3. **Mouse** — Move player toward pointer (on canvas), click to fire

Joystick system uses touch ID tracking to handle multi-touch. Mouse target is smoothly interpolated for responsive steering.

### Game State (Lines 863–911)

Central mutable state:
- `player` — position, weapon, level, invuln timer, cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps` — game objects
- `boss` — current boss (null if none)
- `score`, `lives`, `maxLives`, `stage`, `stageKills`, `kills`
- `chain`, `chainTimer`, `multiplier` — combo system
- `running`, `paused`, `gameOver` — game state flags

### Game Objects & Types (Lines 485–515)

**Difficulties** (3 configurations scaling spawn rates, enemy speed, lives, bombs, HP multipliers)

**Enemy Types:**
- `grunt` — basic fighter, hp: 1
- `interceptor` — fast weaver, hp: 1
- `tank` — heavy armor, hp: 4, shows damage bar

**Bosses** (6 unique designs with color schemes and HP multipliers):
- VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique attack pattern sequence

**Themes** (6 sky gradients + cloud/sun colors, cycle per stage)

### Collision & Physics (Lines 1069–1303)

- `rectsOverlap(a, b)` — AABB collision test for all game objects
- Player health / invulnerability handling
- Bullet-enemy / bullet-boss interactions (hp decrement, pierce tracking for lasers)
- Enemy bullet-player collisions
- Power-up collection and application
- Particle system with velocity decay

### Rendering (Lines 1507–1638)

Canvas rendering pipeline:
1. Clear and apply screen shake offset
2. Draw sky gradient + sun glow + layered clouds
3. Draw player plane (with laser aura if weapon active)
4. Draw player bullets (with glows at quality level 0)
5. Draw enemy bullets
6. Draw enemies (with damage bars for tanks)
7. Draw boss if present
8. Draw power-ups (rotating)
9. Draw particles (fading)

`drawPlane()` and `drawBossShip()` use path-based procedural graphics (no sprite sheets).

### Game Loop (Lines 1640–1665)

`loop(now)` runs via `requestAnimationFrame`. Each frame:
1. Calculate delta time, cap at 50ms
2. Track performance (adaptive quality based on frame times)
3. If running and not paused: call `update(dt)`, apply shakes, call `draw()`
4. Schedule next frame

### Update Logic (Lines 1148–1302)

Per-frame updates happen in `update(dt)`:
- Player movement from keyboard/joystick/mouse input with boundary clamping
- Fire cooldown and weapon timing
- Bullet/enemy movement and cleanup (off-screen)
- Enemy spawning (scaled by frame count and difficulty)
- Enemy weaving behavior (sine wave horizontal offset)
- Enemy shooting (spread or aimed patterns)
- Boss update via `updateBoss(dt)` (movement, telegraph, attack patterns)
- Collision detection (6 separate loops for bullet-enemy, bullet-boss, enemy-player, etc.)
- Combo decay (chainTimer counts down; breaking chain on damage)
- Score/multiplier updates

### Boss Attack Patterns (Lines 1324–1409)

Each boss has 3 attack patterns, cycling through them:
- `fireRadial()` — bullets in full circle
- `fireAimedSpread()` — fan of bullets toward player
- `fireColumns()` — vertical columns
- `fireSpiralBurst()` — rotating spiral
- `fireCross()` — 8-way cross
- `fireHomingSeeded()` — pseudo-homing spray
- `fireDoubleRing()` — concentric rings at different speeds

Boss telegraph system: 22-frame warning before pattern fires, accompanied by audio cue.

### UI & Accessibility (Lines 468–740, 1656–1663)

- HUD display: score, best score, multiplier, weapon level, hearts (lives), bombs
- Overlay card for title screen, pause menu, game over screen
- Difficulty selector (persisted to localStorage)
- Mute and reduced-motion toggles (respects prefers-reduced-motion media query)
- Screen reader announcements via `announce()` for state changes
- ARIA live region for accessibility

### Performance Optimization

Adaptive quality system (lines 473–483):
- Quality level 0 (full) — shadow glows on bullets/powerups, full particle counts
- Quality level 1 (medium) — reduced shadows, 60% particle counts
- Quality level 2 (low) — no shadows, 35% particle counts

Triggered automatically when average frame time exceeds 1/40s (quality up) or drops below 1/56s (quality down).

## Key Data Structures

**Player:**
```
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

**Bullet:**
```
{ x, y, w, h, speed, vx, pierce, laser? }
```

**Enemy:**
```
{ x, y, w, h, speed, hp, type, color, dark, score, weave, weaveSeed, baseX, shootTimer, canShoot, pattern }
```

**Enemy Bullet:**
```
{ x, y, w, h, speed, vx?, radial?, dx?, dy? }  // radial bullets have (dx,dy), others have vx
```

**Boss:**
```
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

**Particle:**
```
{ x, y, vx, vy, life, maxLife, color }
```

## Common Tasks

### Adjusting Game Balance

- **Difficulty scaling:** Edit `DIFFICULTIES` object (lines 485–489) for spawn rates, enemy speed, enemy shoot chance, lives, bombs
- **Enemy stats:** Modify `ENEMY_TYPES` (lines 502–506) for hp, speed multiplier, scoring
- **Boss HP:** Adjust base HP formula (line 1046) and `design.hpMul` in `BOSSES` (lines 508–515)
- **Combo multiplier:** Edit formula in `addScore()` (line 1090) to change how chain length scales multiplier

### Adding/Modifying Enemy Attack Patterns

1. Create a new pattern function in the `fireRadial`/`fireAimedSpread` family (lines 1324–1370)
2. Add it to the appropriate boss's pattern array in `BOSS_PATTERNS` (lines 1372–1409)
3. Patterns append bullets to `enemyBullets` array

### Adding/Modifying Sound Effects

1. Add a new function to the `sfx` object (lines 624–676) using `beep()` and `noiseBurst()`
2. Call `ensureAudio()` first if the sound might fire on user gesture
3. Respect the `muted` flag and `audioCtx` null checks
4. Use `pan` option for positional audio based on `panFor(x)`

### Visual Tweaks

- **Sky themes:** Add/edit entries in `THEMES` array (lines 493–500)
- **Enemy colors:** Modify `color` and `dark` fields in `ENEMY_TYPES`
- **Boss visuals:** Adjust colors and shapes in `drawBossShip()` (lines 970–1019)
- **HUD styling:** Edit CSS `:root`, `.stat`, `#hud`, etc. (lines 10–359)
- **Screen shake intensity:** Adjust magnitude and duration in `screenShake()` calls

### Accessibility & Motion

- `reducedMotion` flag disables animations, screen shake, and vignette pulsing
- CSS media query `(prefers-reduced-motion: reduce)` auto-enables on load
- All animations have reduced-motion overrides in CSS (lines 349–358)
- Keyboard shortcuts: P/Esc pause, X/Shift/Space for bomb/fire

## Browser Compatibility

Requires:
- HTML5 Canvas
- Web Audio API (for sound; game runs silently if unavailable)
- ES6 (arrow functions, const/let, template literals)
- LocalStorage
- requestAnimationFrame

Tested on Chrome, Firefox, Safari, Edge (desktop and mobile).

## Performance Notes

- Single canvas redraw per frame (no sprite bitmaps — all procedural)
- No external assets (images, sounds, fonts)
- Particle and enemy counts scale automatically with device frame rate
- Touch controls work on devices with 0 maxTouchPoints (fallback to mouse)
- Reduced motion preference respected for critical animations

## Future Enhancement Ideas

- Leaderboard (cloud sync or local IndexedDB)
- New enemy types (turrets, decoys)
- Special weapon effects (spread shot, homing, area denial)
- Boss phases (health threshold state changes)
- Stage hazards (obstacles, terrain)
- Local multiplayer (split-screen or co-op)
