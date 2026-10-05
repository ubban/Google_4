# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application (1,600+ lines). It features Canvas-based rendering, synthesized audio via WebAudio, adaptive quality scaling, and responsive touch/keyboard controls for desktop and mobile.

## Architecture & Core Systems

### Game State Machine
- **Idle/Overlay State**: Initial menu screen with difficulty selection and game-over replay screen
- **Running**: Active game loop with update + draw each frame
- **Paused**: Overlay visible, no update/draw
- **GameOver**: Shows final score and statistics

State is managed by `running`, `paused`, and `gameOver` booleans; game state stored in `score`, `lives`, `stage`, `boss`, etc.

### Entity System
Entities (player, enemies, bullets, particles, power-ups, boss) are stored in arrays and updated/drawn each frame:
- **Player**: Single instance, position-based (x, y), firing cooldown, weapon/level, invulnerability timer
- **Enemies**: Spawned procedurally, each has type (grunt/interceptor/tank), HP, position, velocity, shooting behavior
- **Bullets**: Stored as player bullets and enemy bullets; player bullets pierce/laser variants, enemy bullets support radial motion
- **Particles**: Temporary visual explosions (life-based decay)
- **PowerUps**: Dropped by defeated enemies, collected by player (life/bomb/weapon upgrades)
- **Boss**: One-per-stage, multi-phase enemy with pattern-based bullet attacks

### Rendering Pipeline
1. Clear canvas and apply screen shake translation if active
2. Draw sky (gradient), sun halo, and scrolling cloud layers (theme-based)
3. Draw player plane (with weapon glow for laser mode)
4. Draw player and enemy bullets
5. Draw enemies and enemy planes
6. Draw boss ship (if present)
7. Draw power-ups (rotating)
8. Draw particles (fading explosions)

### Audio System
- **Initialization**: WebAudio context created lazily on first user interaction (`ensureAudio()`)
- **Synth Design**: Pure oscillator + noise-based sound effects (no external assets)
- **Sound Functions**:
  - `beep(freq, dur, type, vol, opts)` — frequency sweep oscillators with optional filters/panning
  - `noiseBurst(dur, vol, opts)` — white noise with envelope
  - `sfx` object — named sound effect functions (shoot, laser, explode, powerup, bomb, etc.)
- **Muting**: Stored in localStorage; audio skips rendering if muted

### Input System
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move to steer (plane glides toward cursor while hovering canvas), click to fire
- **Touch**: Left zone drag for movement (visual joystick), FIRE/BOMB buttons on right
- **Priority**: Keyboard/joystick input takes priority over mouse steering (no fighting)

### Difficulty & Balance
Three preset difficulty profiles (`DIFFICULTIES` object):
- **Easy**: Slower spawns, slower enemies, lower shoot frequency, 4 lives
- **Normal**: Medium values, 3 lives (default)
- **Hard**: Fast spawns, fast/aggressive enemies, 3 lives, 30% boss HP multiplier

Difficulty persisted to localStorage; can be changed anytime from overlay (unless game running).

### Visual Themes
Six sky themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) that cycle by stage, each with:
- Sky gradient (3-stop linear)
- Cloud color and sun halo color
- Loaded from `THEMES` array, applied in `draw()` via canvas gradient

### Adaptive Performance
Simple frame-time tracking (`trackPerf()`) adjusts quality setting (0=full/glow, 1=medium, 2=low):
- Full quality: Shadow/glow effects on bullets and bosses
- Medium: 60% particle spawn, no glow
- Low: 35% particle spawn, no glow
Affects visual polish; core gameplay remains constant.

### Boss System
- **Spawning**: After 18 enemy kills per stage, boss enters from top
- **Patterns**: Each of 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has 3 unique bullet-pattern functions:
  - `fireRadial()` — bullets in ring, all angles
  - `fireAimedSpread()` — cone toward player, multiple angles
  - `fireColumns()` — vertical lanes from boss center
  - `fireSpiralBurst()` — rotating 8-way pattern
  - `fireHomingSeeded()` — aimed bullets with spread noise
  - `fireDoubleRing()`, `fireCross()` — hybrid patterns
- **Attack Cycle**: Telegraph phase (22 frames visual telegraph), then fire, then cooldown (85 frames)
- **Movement**: Bounces horizontally within canvas bounds

### Scoring & Multiplier System
- **Base Scoring**: Enemy killed → `addScore(score_base)` which increments chain counter and applies multiplier (1 + floor(chain/5), capped at 10x)
- **Chain Reset**: 2.2 second timeout on last kill; if no new kill within window, multiplier resets
- **Bomb Kills**: Also grant points (5 pts per enemy, applied with current multiplier)
- **Boss Kill**: Flat 500 points
- **Best Score**: Persisted to localStorage; shown at game over if beaten

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Update `spawnEnemy()` filtering logic to include new type
3. Add explosion sound to `EXPLODE_SFX` if needed (else defaults to `explode()`)
4. Enemy will automatically spawn and behave (weaving, shooting)

### Adding a New Boss Pattern
1. Create a new function in `BOSS_PATTERNS` section (e.g., `fireHelixBarrage()`)
2. Add it to one of the 6 boss pattern arrays
3. Use existing helper functions or create new ones; push to `enemyBullets` with `{ x, y, w, h, radial, dx, dy }` for radial bullets or `{ x, y, w, h, speed, vx }` for aimed bullets
4. Test in-game by reaching that stage/difficulty

### Adjusting Difficulty Parameters
Edit the `DIFFICULTIES` object properties:
- `spawnBase` / `spawnMin` — enemy spawn rate (lower = more frequent)
- `enemySpeed` — baseline enemy velocity multiplier
- `enemyShootChance` — probability enemy fires [0–1]
- `lives`, `bombs` — starting resources
- `bossHpMul` — boss health multiplier

### Modifying Weapon Balance
Edit `fireWeapon()` function:
- **Vulcan** spread angles controlled by `player.level` (1–3 levels, more bullets at higher levels)
- **Laser** lane spacing and pierce count set by level
- Cooldown times set in `update()` (`fireDelay` variable)
- Base damage to enemies: 1 HP per bullet (laser), 3 HP to boss (laser), 1 HP to boss (vulcan)

### Adding Visual Effects
- **Screen Shake**: Call `screenShake(magnitude, duration)` — applies random translation each frame for shake duration
- **Flash Overlay**: Call `flashHit()` or `flashBomb()` — CSS animation triggers on `#flash` element
- **Vignette (damage indicator)**: Automatically shown when lives === 1 (via CSS, respects reduced-motion)
- **Combo/Banner Text**: Call `showComboPop(text)` or `showStageBanner(text)` — CSS animations, respects reduced-motion

### Creating New Sound Effects
Edit or extend `sfx` object:
```javascript
sfx.newSound = () => {
  beep(440, 0.2, 'sine', 0.1, { sweepTo: 220, filterFreq: 3000 });
  noiseBurst(0.1, 0.08, { filterFreq: 1200 });
};
```
- `beep()` — single oscillator with optional sweep and filter
- `noiseBurst()` — noise with filter, full envelope
- Both respect mute state; called from game events

### Tuning Score/Multiplier Feel
Edit `addScore()`:
- `chainTimer = 2.2` — seconds before chain resets
- `multiplier = 1 + Math.min(9, Math.floor(chain / 5))` — formula; currently +1x every 5 kills, capped at 10x
- Adjust numerator in floor() to change curve (smaller = faster scaling)

## Technical Details

### Canvas Sizing
- Base resolution: 480×720 (portrait aspect)
- Scaled up to fit window (max 1.6× scale factor for desktop)
- Responsive: resizes on window resize and orientation change via `resize()` function
- Touch: disables zoom, sets viewport-fit=cover for notch devices

### LocalStorage Keys
- `skyfighter.bestScore` — high score integer
- `skyfighter.muted` — '0' or '1' boolean
- `skyfighter.difficulty` — 'easy', 'normal', or 'hard'
- `skyfighter.reducedMotion` — '0' or '1' (if not set, checks system preference)

### Collision Detection
Simple AABB (axis-aligned bounding box) via `rectsOverlap(a, b)`:
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```
Checked every frame for player-enemy, player-bullet, player-powerup, and boss collisions.

### Frame Counting
- `frame` counter increments every update (used for animation timing, pattern sequences)
- Boss telegraph and pattern timing keyed to frame count (not dt) for determinism
- Visual tweening also frame-based for smooth, predictable animation

### Accessibility
- Screen reader live region (`#srLive`) announces game state changes (game over, boss defeat, lives remaining)
- Reduced motion preference respected; CSS `@media (prefers-reduced-motion: reduce)` disables animations
- Motion button toggles reduced motion on/off
- HUD uses high-contrast colors with text-shadow for readability

## Code Organization

The single HTML file is split into logical sections (marked with comments):
1. **Styles**: Full CSS in `<style>` tag (canvas + UI + animations)
2. **HTML markup**: Minimal DOM (canvas, overlay, HUD, buttons)
3. **JavaScript** (anonymous IIFE):
   - Element references and constants
   - Audio system
   - Game state initialization
   - Entity spawning and updates
   - Collision/scoring logic
   - Rendering pipeline
   - Main game loop

**Minimal external dependencies**: None. Pure HTML5 Canvas, WebAudio, and vanilla JS.

## Testing & Iteration

- Open `index.html` in a browser to play
- Desktop: Mouse to aim, click/Space to fire, arrows/WASD to move, X/Shift bomb, P/Esc pause
- Mobile: Tap FIRE/BOMB, drag left side to move
- Use difficulty buttons to test balance at different settings
- Check browser console for any audio context errors (non-blocking)
- Frame time tracking in `trackPerf()` automatically adapts rendering quality if FPS drops

