# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-em-up game inspired by Raiden. The entire game is implemented in `index.html` (~1,670 lines) with embedded HTML, CSS, and JavaScript—no build process, no external dependencies, no assets.

The game features:
- 6 difficulty-scalable stages with procedural boss encounters
- 2 weapon types (Vulcan spread, Laser piercing) with 3 power levels each
- 3 enemy types with distinct behaviors (grunt, interceptor with weaving, tank with HP)
- 6 unique boss designs, each with 3+ attack patterns (radial bursts, aimed spreads, homing, spirals, etc.)
- Synthesized audio via WebAudio API (no MP3/WAV files)
- Full touch and keyboard input with mouse steering
- Accessibility: screen reader announcements, reduced motion preference
- Responsive canvas that scales to device, localStorage for settings/scores

## Architecture

### Code Organization Within index.html

**Lines 1–360:** HTML structure + CSS
- Canvas and HUD elements (score, lives, bombs, weapon level, boss bar)
- Game overlay (menu, pause, game over states)
- Touch control buttons (left half = movement zone, right half = fire/bomb buttons)
- Responsive design using viewport-relative units (clamp, vw/vh)

**Lines 422–676:** Game initialization + audio system
- LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- WebAudio synthesis: `beep()` for tone generation, `noiseBurst()` for white noise
- SFX registry: `shoot`, `laser`, `explode*` (enemy-type-specific), `bomb`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, etc.
- Audio is context-dependent: muted setting respected, panning based on screen position

**Lines 752–862:** Input handling
- Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- Mouse: moves canvas, hover steers player toward pointer
- Touch: left zone for joystick movement, right zone for fire/bomb buttons
- Input priority: keyboard/joystick > mouse steering (never fight)

**Lines 863–911:** Game state + reset
- Core state: `player` (position, weapon, level, invulnerability), `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Stage-dependent: `stage`, `stageKills`, `kills` (for milestones), `chain` (combo multiplier)
- `resetGame()` initializes from difficulty settings

**Lines 1022–1067:** Spawn functions
- `spawnEnemy()`: draws from `ENEMY_TYPES`, applies difficulty-scaled speed, assigns shoot patterns (aimed vs spread)
- `spawnBoss()`: creates boss from `BOSSES` array, scales HP by stage and difficulty multiplier, displays name + health bar
- Weaving interceptors use sine wave modulation; tanks spawn with full HP; enemies can be configured per-stage

**Lines 1148–1302:** Main update loop
- Player movement: keyboard/joystick > mouse steering (smooth glide)
- Weapon firing: rapid-fire with cooldown (0.15s Vulcan, 0.1s Laser)
- Bullet-enemy collision: per-enemy-type SFX, pierce logic for lasers, power-up drops
- Enemy spawning: controlled by difficulty spawn rates, accelerates over time
- Boss spawn trigger: `KILLS_PER_STAGE` (18) triggered with 1.2s delay
- Chain/multiplier: 2.2s window to maintain, +1 per 5 consecutive kills (cap 10x)

**Lines 1372–1439:** Boss AI
- `BOSS_PATTERNS`: indexed by boss (0–5), each has 3+ attack functions
- Attacks telegraph with 22-frame warning (visual flash + sound)
- Patterns include:
  - Radial bursts (all directions)
  - Aimed spreads (fan toward player)
  - Homing with noise (toward player ± randomness)
  - Teleport dashes (SPECTER, WRAITH)
  - Columns, crosses, double rings
- Boss movement: oscillates left-right, always visible

**Lines 1520–1637:** Rendering
- Sky gradient + sun glow (per-stage theme)
- Scrolling clouds at 2 depths
- Player plane with weapon aura (laser) and thrust flame
- Bullets: glowing rectangles (laser) vs bright squares (vulcan)
- Enemy bullets: red circles
- Power-ups: rotating squares (red life, yellow bomb, green weapon)
- Particles: expanding circles fading out
- Screen shake applied via ctx.translate before drawing

**Lines 1640–1665:** Main loop
- `requestAnimationFrame` for 60fps targeting
- Delta time capped at 50ms (safeguard against tab backgrounding)
- `trackPerf()` monitors frame time, auto-scales quality (0=full glow, 1=medium, 2=minimal) to maintain 60fps

### Key Data Structures

**DIFFICULTIES** (line 485–489):
```
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```
Controls spawn rate ramp, enemy aggression, player resources, and boss health scaling per difficulty.

**ENEMY_TYPES** (lines 502–506):
```
{ w, h, hp, speedMul, color, dark, score, minStage, [weave] }
```
Tanks are 4x HP, interceptors weave, grunts are baseline.

**BOSSES** (lines 508–515):
```
{ name, hullTop, hullBottom, wing, core, hpMul }
```
Visual design + HP multiplier. Used to render boss hull gradient and select attack patterns.

**THEMES** (lines 493–500):
```
{ sky: [stop0, stop45, stop100], cloud, sun, name }
```
Controls background gradient, cloud/sun color, visible when entering each stage.

## Common Development Tasks

### Adjust Difficulty Tuning
Edit `DIFFICULTIES` (line 485). Increase `spawnBase` for faster spawning, `enemyShootChance` for harder enemies, or `bossHpMul` for tank bosses.

### Add or Modify a Boss
1. Add entry to `BOSSES` array (line 508), increment size limit in `BOSS_PATTERNS` (line 1372)
2. Add attack pattern array to `BOSS_PATTERNS` with 3+ functions (examples: `fireRadial`, `fireAimedSpread`, `fireCross`)
3. Boss selects patterns by `(stage - 1) % BOSSES.length`

### Add Enemy Type
1. Add to `ENEMY_TYPES` with minStage (line 502)
2. Add SFX handler to `sfx[EXPLODE_SFX[newType]]` (line 678)
3. Update spawn logic in `spawnEnemy()` to include the new type's spawn chance (line 1026)

### Add Weapon or Power-up
1. Modify weapon logic in `fireWeapon()` (line 1124) or `applyPowerUp()` (line 1304)
2. Update HUD display in `weaponLabel()` and UI text

### Tune Performance
- Edit `quality` scaling in `trackPerf()` (line 476): adjust thresholds or particle counts in `spawnExplosion()` (line 1074)
- Reduce shadow blur in `drawBossShip()` (line 1002) when quality > 0
- Disable glow entirely by setting `glow = false` override

### Change Visual Theme
- Add new entry to `THEMES` array (line 493) with RGB values
- Adjust in-game color overrides: player plane (line 1558), bullets (line 1589), power-ups (line 1621)

### Add New Audio
- Add function to `sfx` object (line 624)
- Use `beep(freq, duration, type, volume, opts)` for tones or `noiseBurst(duration, volume, opts)` for noise
- Call `ensureAudio()` before using (initializes WebAudio context on first user interaction)

## Testing & Verification

- **No test suite:** manually verify in-browser by opening `index.html` in a modern browser
- **Keyboard:** Arrow keys, Space, X, P
- **Mouse:** move to steer, click to fire
- **Touch:** swipe left to move, tap FIRE/BOMB
- **Console:** check for errors (Ctrl+Shift+I / Cmd+Option+I)

## Performance Notes

- **Adaptive Quality:** game auto-scales glow effects, particle counts, and shadow blur if FPS drops below 40
- **Canvas Sizing:** BASE_W=480, BASE_H=720, scales up to device size (max 1.6x) while maintaining aspect ratio
- **Audio:** synthesized in-thread; may stutter on low-end devices if too many voices active simultaneously
- **Memory:** no texture atlases, no sprite sheets; all drawing is procedural (ctx.fillPath, ctx.arc)

## Browser & Platform Support

- **Desktop:** Chrome, Firefox, Safari, Edge (WebAudio required)
- **Mobile:** iOS Safari 14.5+ (WebAudio + touch events), Android Chrome/Firefox
- **Accessibility:** ARIA live region for screen readers, prefers-reduced-motion media query respected

## Deployment

No build, transpile, or bundling needed. Serve `index.html` as-is via any HTTP server. Scores/settings persist in localStorage (no server required).
