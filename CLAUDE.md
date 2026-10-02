# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. The entire game runs in `index.html` — a self-contained Canvas application with embedded CSS, JavaScript, WebAudio synthesis, and no external dependencies.

## Running & Testing

Since there's no build step, testing is straightforward:

- **Open in browser**: Simply open `index.html` directly in a web browser (Firefox, Chrome, Safari, Edge all work)
- **Local server** (if needed): `python -m http.server 8000`, then visit `http://localhost:8000/index.html`
- **Mobile testing**: The game detects touch devices and renders mobile controls (joystick + fire/bomb buttons)

**Play the game**:
- Desktop: Mouse to move, Click/Space to fire, X/Shift/Space for bombs, P/Esc to pause
- Mobile: Drag left zone to move, tap FIRE, tap BOMB

## High-Level Architecture

The game runs a `requestAnimationFrame` loop that drives three phases each frame:

1. **Update** (`update(dt)`): Physics, collisions, spawning, state changes
2. **Draw** (`draw()`): Canvas rendering with themes, particles, effects
3. **Audio**: Procedural SFX via WebAudio (no asset files)

### Core Systems

**Game State** (lines 863-901)
- `player`: Position, weapon, health, cooldown
- `enemies`, `enemyBullets`, `bullets`: Arrays of projectiles and entities
- `boss`: Active boss object (if spawned); spawns after 18 kills per stage
- `score`, `multiplier`, `chain`: Score mechanics with kill-chain bonuses
- `lives`, `bombCount`, `stage`: Player resources and progression

**Rendering** (lines 1520-1637)
- Sky gradients cycle through 6 themes based on stage
- Canvas coordinate system: 480×720 (scaled to fit window)
- Plane sprite drawing: `drawPlane()` for player/enemies, `drawBossShip()` for bosses
- Particle effects for explosions (spawned on kill/bomb/hit)
- Screen shake effect for impact feedback (reduced if `reducedMotion` is on)

**Audio Synthesis** (lines 542-676)
- WebAudio synth with oscillators, filters, and noise generation
- No external audio files; every sound is generated via `beep()` and `noiseBurst()`
- Stereo panning: enemy sounds pan left/right based on screen position
- Mutable via localStorage preference; auto-initializes on first input

**Input Handling** (lines 767-861)
- Keyboard: Arrow keys/WASD for movement, Space/X/Shift for fire/bomb
- Mouse: Pointer position drives player movement (smoothly glides to target)
- Touch: Left zone is joystick, right zone has FIRE/BOMB buttons
- Priority: Keyboard/joystick > mouse (mouse doesn't fight other input when both active)

**Collision & Scoring** (lines 1232-1294, 1087-1101)
- Axis-aligned bounding box overlap: `rectsOverlap(a, b)`
- Enemy kills increment `chain`; chain resets after 2.2s inactivity
- Multiplier grows: 1x (0 kills) → 2x (5 kills) → 10x (45 kills)
- Boss/enemy/bullet collisions trigger explosions and audio

**Boss Patterns** (lines 1372-1409)
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), cycling by stage
- Each has 3 attack patterns: radial, aimed spread, special (teleport, columns, homing, etc.)
- Bosses telegraph attacks (visual glow + SFX) then fire
- Laser weapons pierce bosses (break through armor), regular bullets don't
- Bomb always deals 24 fixed damage to boss

**Difficulty Scaling** (lines 485-489)
- Easy/Normal/Hard presets affect: enemy spawn rate, speed, bullet frequency, player lives/bombs, boss HP
- Enemy difficulty also ramps over time: spawn timer decreases based on frame count
- Player can switch difficulty from menu (not mid-game)

**Weapon System** (lines 1124-1146, 1304-1322)
- Two weapons: Vulcan (spread cone as level 1→3) and Laser (pierce count 1→3)
- Power-up pickups cycle: Level 1→2→3→switch weapon→reset to Level 1
- Weapon cooldowns: Vulcan 0.15s, Laser 0.1s

**Adaptive Performance** (lines 473-483)
- Tracks frame times and scales `quality` (0=full, 1=medium, 2=low)
- Lower quality reduces particle counts and disables shadow/glow effects
- Checks average of last 45 frames; ups quality if frames > 25ms, downs if < 17.8ms

**Mobile/Accessibility** (lines 520-540, 778-779, 349-358)
- Detects touch devices and renders overlay with FIRE/BOMB buttons + virtual joystick
- Safe-area insets for notched phones (`env(safe-area-inset-top)`)
- `prefers-reduced-motion` media query + toggle: disables animations, screen shake, flashes if enabled
- Screen reader support: `aria-live` region announces kills/score/stage changes

**Persistence** (lines 463-466, 517-523)
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Saves settings immediately on toggle

## Common Tasks

**Add a new enemy type**: Define in `ENEMY_TYPES` object (line 502-506), set `minStage`, adjust spawn logic in `spawnEnemy()`.

**Add a boss pattern**: Create a function like `fireRadial()`, add it to a boss's patterns array in `BOSS_PATTERNS` (line 1372-1409).

**Change difficulty values**: Edit `DIFFICULTIES` object (line 485-489). Affects spawn rate, enemy speed, lives, bombs, boss HP scaling.

**Tweak visual theme**: Edit `THEMES` array (line 493-500) — sky gradient colors, cloud color, sun color, and name.

**Adjust weapon balance**: Modify `fireWeapon()` (line 1124-1146) for spread angles, speeds, pierce counts; modify cooldowns (line 1177).

**Add visual feedback**: Use `screenShake(mag, dur)` for impact, `flashHit()`/`flashBomb()` for vignette, `showComboPop(text)` for floating text.

**Add audio**: Call existing SFX like `sfx.shoot()`, `sfx.bomb()`, etc., or add a new key to the `sfx` object with `beep()` or `noiseBurst()` calls. Pan is handled via `panFor(screenX)` for stereo positioning.

## Key Implementation Details

**Canvas sizing** (line 752-765): Base resolution is 480×720; scales up to fit window (max 1.6×). Resize handler fires on window resize or orientation change.

**Delta time clamping** (line 1644): `dt` is capped at 0.05s to prevent spiral deaths if frame times spike.

**Particle physics** (line 1296): Particles have `vx`, `vy` velocity and fade via `life`; removed when `life <= 0`.

**Enemy weaving** (line 1210-1215): Interceptor type uses sine wave (`Math.sin`) to weave horizontally as they descend.

**Boss telegraph mechanic** (line 1421-1434): Boss enters telegraph state (`telegraph = 22`), player sees glow + hears warning tone, then fires pattern after telegraph expires.

**Invulnerability frames** (line 1174, 1457-1459): Player blinks when `invuln > 0`; set to 1.5s after taking hit so they can't chain-die.

**Chain/multiplier timeout** (line 1183-1186): Chain decays if no kills for 2.2s (`chainTimer`).

**Power-up spawn rates** (line 1058-1067): 6% life drop, 8% bomb drop, 20% weapon drop from killed enemies.

## File Structure

- **index.html** (1669 lines): Entire game
  - Lines 1–359: CSS (layout, animations, HUD, overlay, touch controls, accessibility)
  - Lines 360–421: HTML structure (canvas, HUD, buttons, touch zones)
  - Lines 422–1667: JavaScript game engine (state, rendering, audio, input, physics, AI)

No other files are needed.
