# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The entire game logic, rendering, audio synthesis, and UI are contained in `index.html` (1669 lines).

**Key characteristics:**
- No build process, frameworks, or external dependencies
- Canvas-based 2D rendering with adaptive quality levels
- Web Audio API for procedurally generated sound (no asset files)
- localStorage persistence for game state (scores, settings)
- Multi-input support: keyboard, mouse, touch/mobile with virtual joystick
- Responsive design with safe-area awareness for mobile notches

## Running the Game

### Local Development
- Open `index.html` directly in a browser, or
- Serve from any HTTP server: `python -m http.server 8000` (or equivalent)
- Game is immediately playable; no compilation or installation needed

### Testing
No automated tests or test framework. Verify changes manually:
- Test all three difficulty levels (Easy, Normal, Hard)
- Test on both desktop and mobile (Chrome DevTools device emulation recommended)
- Verify keyboard controls: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- Verify mouse controls: Move to steer, click to fire
- Verify touch controls: Drag left zone to move, tap FIRE/BOMB buttons
- Check audio works (both synthesis and muting toggle)
- Check reduced-motion preference is respected

### Accessibility
- Game respects `prefers-reduced-motion` media query and provides toggle button
- Screen reader announcements for key events (via `#srLive` aria-live region)
- Keyboard-only playable (P/Esc for pause, etc.)

## High-Level Architecture

### Game Loop (lines 1640–1665)
```
requestAnimationFrame(loop) → trackPerf(dt) → update(dt) → draw() → repeat
```
The main loop runs at ~60fps with adaptive quality scaling based on frame times.

### Major Game Systems

#### 1. **Input Handling** (lines 767–861)
- **Keyboard:** Map key codes to action flags; checked each frame
- **Mouse:** Steering via pointer position; click to fire
- **Touch:** Virtual joystick on left, action buttons on right
- **Priority:** Keyboard/joystick > mouse (prevents conflicts)

#### 2. **Game State** (lines 863–911)
Global variables track:
- Player object (position, weapon, level, lives, invulnerability timer)
- Bullets, enemies, enemy bullets, particles, power-ups, boss
- Score, lives, stage, kills, spawn timers, chain/multiplier system

**Key constants:**
- `BASE_W = 480, BASE_H = 720` (logical canvas size; scales responsively)
- `KILLS_PER_STAGE = 18` (enemies to defeat before boss)
- `DIFFICULTIES`: Easy/Normal/Hard with spawn rates, enemy speed, lives

#### 3. **Rendering** (lines 1520–1638)
Canvas drawing order:
1. Sky gradient + sun glow + parallax clouds (2 layers at different speeds)
2. Player plane
3. Bullets (yellow/cyan with optional glow)
4. Enemy bullets (red circles)
5. Enemies (planes with health bars for tanks)
6. Boss ship
7. Power-ups (rotating colored squares)
8. Particles (explosion debris)

**Visual effects:**
- Screen shake (amplitude/duration based on event)
- Flash overlays (white for hits, fullscreen for bomb)
- Vignette pulse (red flashing when 1 life remains)
- Combo pop (center screen score multiplier notification)

#### 4. **Physics & Collision** (lines 1069–1302)
- `rectsOverlap(a, b)`: Simple AABB collision using center distance
- Bullets move upward; enemy bullets downward; particles drift with gravity
- Collision checks each frame between: bullets↔enemies, bullets↔boss, enemies↔player, bullets↔player, boss↔player, powerups↔player
- Player constrained to canvas bounds; keyboard/joystick input adds velocity

#### 5. **Weapons System** (lines 1124–1146, 1304–1322)
- **Vulcan:** Default, fires spread of bullets (1, 2, or 3 lanes by level)
  - Level 1: center only
  - Level 2: left + right
  - Level 3: left + center + right
  - Fire delay: 0.15s
- **Laser:** Alternative weapon, narrow beams with pierce (ignores some enemies)
  - Levels 1–3 gain extra lanes
  - Fire delay: 0.1s (faster)
- **Power-up types:** life, bomb, weapon (cycles: Vulcan→Laser, resets level 1)

#### 6. **Enemy System** (lines 1022–1041, 1210–1228)
- Three enemy types (defined in `ENEMY_TYPES`):
  - **Grunt:** Red, 34×34px, 1 HP, basic enemy
  - **Interceptor:** Orange, 26×26px, 1 HP, fast, weaves side-to-side
  - **Tank:** Purple, 46×46px, 4 HP, slow, shows health bar
  - Types unlock by stage (grunt/interceptor stage 1+, tank stage 2+)
- Enemies spawn continuously above screen, move downward
- Two fire patterns: spread (3 bullets in fan) or aimed (track player)
- Spawn rate decreases as difficulty ramps (faster spawns over time)

#### 7. **Boss System** (lines 1043–1056, 1372–1439)
- Six boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) cycle per stage
- Each boss has unique attack patterns (8 pattern types: radial, aimed spread, columns, spiral, cross, homing, double-ring, teleport)
- Boss enters screen from top over ~1.3s, then attacks using telegraph (white glow 22 frames before attack)
- Telegraphing and attack timing create attack-avoid rhythm
- Boss spawn triggered after `KILLS_PER_STAGE` (18 kills)

#### 8. **Score & Chain System** (lines 1087–1101)
- Each enemy kill increments chain counter and resets 2.2s timer
- Chain breaks if player takes damage or timer expires
- Multiplier scales: 1×, 2×, 3×, etc. (capped at 10× at chain=50)
- Score = base points × multiplier
- Combo milestones trigger on-screen "X KILLS!" notifications

#### 9. **Audio Synthesis** (lines 542–676)
- No audio assets; all sounds procedurally generated via Web Audio API
- **Key techniques:**
  - Oscillators (square, sawtooth, sine, triangle) with envelope shaping
  - Frequency sweeps (e.g., explosion "pitch drop")
  - Noise burst generation (explosion texture)
  - Biquad filters (lowpass, highpass for tone shaping)
  - Stereo panning based on bullet/explosion X position
- `beep(freq, dur, type, vol, opts)` generates single tone with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)` generates filtered white noise
- `sfx` object contains ~10 sound effects (shoot, laser, explosion, powerup, boss, etc.)
- Audio paused if game is muted or context is suspended

#### 10. **Performance Adaptation** (lines 473–483)
- Tracks last 45 frame times (rolling average)
- Quality levels: 0 (full), 1 (medium, reduce particles), 2 (low, fewer particles + reduced glow)
- If avg frame time > 1/40s, increase quality (reduce detail)
- If avg frame time < 1/56s, decrease quality (improve visuals)
- Quality affects particle counts, glow effects (shadowBlur)

#### 11. **UI & Overlays** (lines 34–359)
- **HUD (top center):** Score, Best, Multiplier, Weapon name+level, Hearts, Bombs
- **Boss bar (top center when boss active):** Boss name + HP bar
- **Overlay (center modal):** Start screen, pause screen, game-over screen
- **Top-right controls:** Mute 🔊, Motion toggle 🎆, Pause ⏸
- **Touch zone visualization:** Virtual joystick when dragging; FIRE/BOMB buttons always visible on mobile

#### 12. **Difficulty & Progression** (lines 485–489, 738–750)
- Three difficulties affect: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- Difficulty selector on start screen; persists to localStorage
- Game progresses through infinite stages (bosses cycle through 6 designs)
- Each stage shows theme (sky colors/clouds) from `THEMES` array (6 themes cycle)

## Key Code Sections to Know

| Section | Lines | Purpose |
|---------|-------|---------|
| Constants & Difficulty | 485–501 | Game balance parameters, enemy/boss configs |
| Input Setup | 768–861 | Keyboard, mouse, touch event listeners |
| Game State | 863–911 | Global variables and reset function |
| Drawing Functions | 913–1020 | Plane/boss sprite rendering |
| Spawning & Collision | 1022–1071 | Enemy/powerup spawning, AABB overlap |
| Update Logic | 1148–1302 | Main physics, collisions, state updates |
| Boss Patterns | 1372–1409 | Boss attack pattern definitions |
| Rendering | 1520–1638 | Canvas drawing and visual effects |
| Main Loop | 1640–1665 | requestAnimationFrame and game tick |

## Development Tips

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (copy existing, adjust w/h/hp/color/score)
2. Set `minStage` to when it should appear
3. If weaving behavior needed, set `weave: true`
4. Optional: add explosion sound to `EXPLODE_SFX` and `sfx` object

### Adding a New Boss Pattern
1. Define pattern function in one of the `BOSS_PATTERNS` arrays (index = boss design)
2. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. Pattern receives boss object `b`, fires into `enemyBullets` array
4. Telegraph (white glow) appears 22 frames before pattern fires (player has time to react)

### Adjusting Game Balance
- **Difficulty:** Modify entries in `DIFFICULTIES` object
- **Spawn rates:** Increase `spawnBase` / decrease `spawnMin` for faster spawning
- **Enemy speed:** Adjust `enemySpeed`, `speedMul` per enemy type
- **Weapon cooldown:** Modify `fireDelay` in `update()` → `fireWeapon()`
- **Boss HP:** Change `baseHp` formula or `design.hpMul` multiplier

### Visual Tweaks
- **Colors:** Search for hex color codes or theme definitions
- **Particle effects:** Adjust explosion count/color/lifetime in `spawnExplosion()`
- **Screen shake:** Modify `screenShake(mag, dur)` calls (higher mag = more intensity)
- **Quality thresholds:** Adjust frame-time targets in `trackPerf()`

### Sound Design
- **New sound:** Add function to `sfx` object using `beep()`/`noiseBurst()`
- **Frequency ranges:** Lower = bass (50–200Hz), mid = 400–800Hz, high = 1500–4000Hz+
- **Duration:** Keep < 1s for effects; <0.5s for zippy sounds
- **Envelope:** Use `gain.linearRamp...` (attach) / `exponentialRamp...` (release)

## Common Gotchas

1. **Canvas coordinate origin** (0,0) is top-left; Y increases downward
2. **Frame-based timing:** `frame` counter increments each tick; animations often use `Math.sin(frame * 0.N)` for smooth loops
3. **Collision center-based:** Uses center-to-center distance; `(a.w + b.w) / 2` is collision radius
4. **Invulnerability windows:** After taking damage, player is invulnerable (`player.invuln > 0`) for 1.5s; they flicker
5. **Chain timer:** Resets to 2.2s on every kill; breaks if it expires or player takes damage (not from bomb usage)
6. **Mobile z-index:** Touch controls layer is `z-index: 4`; use 5+ for overlays
7. **Audio context:** Must be resumed on first user input (not background auto-play)

## Extending the Game

- **New themes:** Add object to `THEMES` array (sky gradients, cloud color, sun color)
- **New difficulty:** Add entry to `DIFFICULTIES` object
- **Infinite procedural bosses:** Currently cycles 6 designs; could randomize or combine patterns
- **New weapon type:** Add to weapon branching in `applyPowerUp()` and `fireWeapon()`
- **Leaderboard:** Extend localStorage to store top-N scores with name/date

---

**Game design philosophy:** Arcade action with accessibility; responsive controls, forgiving audio/motion toggles, infinite replayability via difficulty selection and procedurally selected stages.
