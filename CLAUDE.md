# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML application (1669 lines). No build tools or external assets are required—the entire game, including graphics, audio synthesis, and UI, is self-contained in `index.html`.

### Quick Start

To run the game:
1. Open `index.html` in a web browser
2. Select difficulty (Easy, Normal, Hard)
3. Play using mouse, keyboard (arrows/WASD to move, Space to fire, X/Shift to bomb), or touch controls on mobile

## Architecture

### Single-File Structure

The game is organized in layers within `index.html`:

1. **HTML (lines 361–420)**: DOM structure with canvas, HUD elements, buttons, and controls
2. **CSS (lines 10–358)**: Game styling, animations (keyframes), responsive layout, dark theme
3. **JavaScript (lines 422–1666)**: Core game logic, state management, and rendering

All graphics are procedurally drawn using the Canvas 2D API. All sound is synthesized via the WebAudio API—no external audio files.

### Key JavaScript Sections

#### State Management (lines 863–910)
- Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Metrics: `score`, `lives`, `stage`, `chain` (combo multiplier), `multiplier`
- Difficulty-dependent configs loaded from `DIFFICULTIES` object

#### Input Handling (lines 767–861)
- Keyboard: Arrow keys / WASD for movement, Space / click for firing, X / Shift for bombs, P / Esc for pause
- Mouse: Pointer movement steers the player; click fires
- Touch: Left zone is movement (virtual joystick), right zone has FIRE and BOMB buttons
- Input priority: keyboard/joystick > mouse (prevents conflict)

#### Game Loop (lines 1640–1665)
- `requestAnimationFrame(loop)` drives the game at ~60fps
- Each frame: `update(dt)` for logic, `draw()` for rendering
- Delta-time clamped to 50ms to prevent large jumps
- Adaptive quality system (`trackPerf`) reduces particle/shadow effects on low-end devices

#### Core Systems

**Update Logic** (lines 1148–1302):
- Player movement (keyboard/joystick/mouse priority)
- Bullet lifecycle (player shots and enemy projectiles)
- Enemy spawning and behavior (weaving, shooting patterns)
- Collision detection (enemy–bullet, player–enemy, player–powerup, boss–bullet)
- Power-up system (life, bomb, weapon upgrades)
- Boss AI and telegraph system
- Chain/combo multiplier (resets if > 2.2s without kills)

**Rendering** (lines 1520–1638):
- Sky gradient with animated clouds (parallax effect at two speeds)
- Procedural plane drawing (`drawPlane`): player and enemies
- Procedural boss ship drawing (`drawBossShip`): detailed hull, wings, core, antenna
- Particles: explosion effects with gravity falloff
- Screen shake effect with configurable magnitude/duration
- HUD updates (score, lives, weapon level, bomb count, boss HP bar)

**Boss System** (lines 1372–1439):
- Six unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with distinct firing patterns
- Each boss has 3 unique attack patterns that cycle
- Firing functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- Telegraph phase warns player before each attack
- HP scaled by difficulty and boss multiplier
- Defeated boss triggers stage advance

**Audio (lines 542–676)**:
- WebAudio synth with `beep()` (oscillator-based) and `noiseBurst()` for SFX
- All sounds: shoot, laser, explosions (by enemy type), hit, bomb, power-up, boss telegraph, stage clear, game over
- Mute toggle stored in localStorage
- Master gain and compressor for dynamic audio
- Audio context is lazy-initialized on first user interaction

#### Game State & Persistence (lines 517–527)

Uses `localStorage` for:
- `skyfighter.bestScore`: best score across sessions
- `skyfighter.muted`: mute state (1 or 0)
- `skyfighter.difficulty`: selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: reduced motion preference (respects OS preference on first load)

### Difficulty Balancing

Three presets (defined in `DIFFICULTIES` object, lines 485–489):

| Param | Easy | Normal | Hard |
|-------|------|--------|------|
| Enemy spawn rate | 75 base | 58 base | 40 base |
| Enemy speed | 1.0× | 1.4× | 1.9× |
| Enemy shoot chance | 45% | 70% | 95% |
| Starting lives | 4 | 3 | 3 |
| Starting bombs | 3 | 2 | 2 |
| Boss HP multiplier | 0.8× | 1.0× | 1.3× |

### Visual Themes

Six cyclic sky themes (lines 493–500), changing each stage:
- Daylight Front (cyan sky)
- Sunset Corridor (orange/pink)
- Night Approach (dark blue)
- Storm Front (green-gray)
- Arctic Wastes (light cyan)
- Volcanic Ridge (dark red/orange)

Each theme has gradient colors for sky, cloud color, and sun glow.

### Enemy Types

Three types with stage-gated unlocks (lines 502–506):

| Type | Size | HP | Speed | Score | Unlocks |
|------|------|----|----|-------|---------|
| Grunt | 34×34 | 1 | 1.0× | 10 pts | Stage 1 |
| Interceptor | 26×26 | 1 | 1.7× | 15 pts | Stage 1, weaves |
| Tank | 46×46 | 4 | 0.55× | 35 pts | Stage 2, shows HP bar |

### Weapon System

Two weapons, each with 3 upgrade levels:

**Vulcan** (default):
- Lv 1: Single-line shots down center
- Lv 2: Dual-lane spread (-0.12, +0.12 angle)
- Lv 3: Triple-lane spread (-0.24, 0, +0.24 angle)

**Laser**:
- Lv 1: Single narrow beam with pierce=1
- Lv 2: Dual beams with pierce=1 each
- Lv 3: Triple beams (center + left/right pair) with pierce=2

Fire rate: Vulcan 150ms, Laser 100ms (faster but tighter pattern).

Power-ups cycle: weapon level increments until Lv 3, then switches to the other weapon at Lv 1.

## Common Tasks

### Run the Game
```bash
# Simply open in a browser:
open index.html
# Or serve locally:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Test Gameplay Manually
1. Open DevTools (F12) to monitor console for errors
2. Play through Easy → Normal → Hard to verify balance
3. Check that each boss stage reaches with proper enemy difficulty ramp
4. Verify audio synthesis works (click anything to unlock WebAudio context)

### Adjust Difficulty or Balance
1. Edit `DIFFICULTIES` object (lines 485–489) for spawn rates, speeds, boss HP
2. Edit `ENEMY_TYPES` (lines 502–506) for enemy stats and appearance
3. Edit `BOSSES` (lines 508–515) for boss coloring and HP multipliers
4. Edit `BOSS_PATTERNS` (lines 1372–1409) for attack sequences

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
2. Update `spawnEnemy()` (lines 1022–1041) if the new type needs special logic
3. Add SFX in `EXPLODE_SFX` map (line 678) if using custom explosion sound

### Add a New Boss
1. Add design to `BOSSES` array (lines 508–515) with hull colors, core color, name, `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` (lines 1372–1409) with 3 firing functions
3. Boss index is auto-calculated as `(stage - 1) % BOSSES.length`

### Add/Modify SFX
1. Use `beep(freq, duration, type, volume, opts)` for tonal sounds (lines 566–595)
   - `opts.sweepTo`: frequency ramp
   - `opts.filterFreq`, `opts.filterType`: filter cutoff and type
   - `opts.pan`: stereo panning (left–1 to right +1)
   - `opts.delay`: onset delay in seconds
2. Use `noiseBurst(duration, volume, opts)` for noise-based sounds (lines 597–623)
3. Add new SFX to `sfx` object (lines 624–676)
4. Call in appropriate places: `sfx.shoot()`, `sfx.bossDown()`, etc.

### Reduce Particle Count for Performance
In `spawnExplosion()` (lines 1073–1085), the multiplier on line 1074 auto-reduces particle count based on quality setting:
- `quality = 0` (high-end): full count
- `quality = 1` (medium): 60% count
- `quality = 2` (low): 35% count

The quality adjusts automatically if frame time exceeds thresholds (lines 473–483).

### Modify Weapon Patterns
Edit `fireWeapon()` (lines 1124–1146):
- For Vulcan: adjust `spread` array angles (in radians)
- For Laser: adjust `lanes` offset array

### Adjust Screen Shake
Call `screenShake(magnitude, duration)` anywhere in the update logic:
- Boss hit: `screenShake(18, 0.5)` (line 1443)
- Reduced-motion mode reduces both magnitude and duration (line 728)

### Enable/Disable Animations
Reduced motion mode (toggle with 🎆 button) applies `animation: none !important` to all keyframes when enabled. CSS defined at lines 349–358. Respects OS `prefers-reduced-motion` on first load.

## Development Guidelines

### Code Style
- Vanilla JavaScript (no framework dependencies)
- Use `const` for immutable state, `let` for mutable
- Keep functions short and focused
- Comment only WHY, not WHAT (e.g., "pause resetting chain prevents combo cheese" rather than "reset chain variable")

### Collision Detection
Single unified function `rectsOverlap(a, b)` (line 1069) using half-width/half-height AABB. Used for all checks: bullets, enemies, boss, power-ups.

### Canvas Coordinate System
- Origin (0, 0) is top-left
- Y increases downward
- Canvas scales responsively with `resize()` (lines 754–765) to fit viewport while maintaining 480×720 aspect ratio
- Canvas context is 2D and accessed globally as `ctx`

### Avoiding Common Pitfalls
1. **Invulnerability frames**: Player gets 1.5s of `invuln` after taking damage; always check `player.invuln > 0` before applying hits
2. **Chain/multiplier decay**: Chain resets if > 2.2s without a kill; check `chainTimer` to prevent cheese
3. **Boss telegraph**: Always check `boss.telegraph > 0` before attack fires; telegraphing state is visual feedback
4. **Audio context**: Always call `ensureAudio()` before playing sounds; it handles suspended state on mobile
5. **Screen shake**: Applies ONLY if `reducedMotion` is false; check before rendering shake effect

### Testing Scope
- No automated tests exist; manual gameplay is the primary validation
- Frame-rate tracking (`trackPerf`) helps identify performance regressions
- Check DevTools console for runtime errors (no console.error calls in production path)
- Verify touch controls work on mobile (use a real device or emulator)

### localStorage Keys
Always use the constants defined at lines 463–466:
- `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`

### Accessibility
- Screen reader support via `srLive` element (line 451) and `announce()` function (lines 468–471)
- Reduced motion preference and toggle
- High-contrast dark theme by default
- All interactive elements have semantic HTML or ARIA roles

## Known Limitations

1. **Single file**: All code in one HTML file means no module separation
2. **No local storage for custom settings**: Only difficulty, mute, best score, motion preference are persisted
3. **No save/resume**: Game must be played to completion in one session
4. **WebAudio synthesis**: Audio may not play on devices without WebAudio support; gracefully degrades (muted)
5. **Canvas rendering**: No support for custom shaders; all effects are 2D canvas operations
6. **Mobile UX**: Touch controls are simple; no advanced haptic feedback

## Performance Considerations

- **Adaptive quality**: Game auto-detects low-end devices and reduces particles/shadows (lines 473–483)
- **Frame budget**: Delta-time capped at 50ms (line 1644) prevents spiral of death
- **Array filtering**: Bullets, enemies, particles are filtered out-of-bounds each frame (lines 1189, 1192, 1195, 1228)
- **Canvas context state**: Saved/restored frequently to avoid state leaks
- **WebAudio compressor**: Applied to prevent audio clipping even with many simultaneous SFX
