# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features:
- Multiple enemy types with distinct behaviors and patterns
- Boss encounters with varied attack patterns
- Weapon progression system (Vulcan and Laser)
- Difficulty levels (Easy, Normal, Hard)
- Full touch and keyboard input support
- Web Audio API synthesized sound effects
- Responsive canvas rendering with adaptive quality
- Progressive difficulty scaling

## Architecture

### Single-File Design
The entire game is in `index.html` (~1670 lines) with no external dependencies. The file is structured as:
- **HTML structure** (lines 1–420): Canvas, HUD elements, overlay UI, touch controls
- **CSS styling** (lines 10–358): Dark theme, responsive layout, animations, HUD design
- **JavaScript game engine** (lines 422–1666): Wrapped in an IIFE for scope isolation

### Core Game Systems

**Game Loop** (line 1640–1652): `requestAnimationFrame` drives ~60 FPS updates with:
- Delta-time based physics (`dt` in seconds)
- `update(dt)` for game state changes
- `draw()` for rendering
- Frametime tracking for adaptive quality

**Entity System**: Objects use simple property bags:
- `player`: Position, dimensions, cooldown, weapon state, invulnerability
- `enemies`: Array of objects with position, HP, shooting behavior, enemy type
- `bullets`: Player and boss projectiles (pierce mechanics for laser)
- `enemyBullets`: Radiating or homing patterns from bosses
- `particles`: Explosion effects with velocity and fading
- `powerUps`: Life, bomb, and weapon upgrades

**Physics**: Collision detection uses `rectsOverlap()` (AABB bounding boxes). Movement updates coordinates directly in `update()`.

**Difficulty Tiers** (line 485–489): Each difficulty (`DIFFICULTIES` object) modulates enemy spawn rates, speeds, shoot chances, HP multipliers, and bomb/life counts. Hard increases complexity through tighter enemy patterns and boss HP.

### Rendering Pipeline

**Background & Themes** (line 493–500): Six themed skies (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with gradient backgrounds, colored clouds, and sun glows. Theme cycles per stage.

**Drawing Order** (line 1520–1637):
1. Clear and optionally apply screen shake offset
2. Gradient sky background and sun radial glow
3. Parallax clouds at two speeds
4. Player ship (with laser aura if weapon equipped)
5. Bullets (with glow if quality = 0)
6. Enemy bullets (red circles)
7. Enemies (plane sprites + tank health bars)
8. Boss ship
9. Power-up pickups (rotating squares)
10. Particle explosions (fading circles)

**Screen Shake**: Offset by random pixels during impacts (lines 726–730). Reduced or disabled in reduced-motion mode.

### Input Handling

**Keyboard/Arrow Keys** (line 768–776): WASD or arrow keys for movement, Space/X/Shift for fire/bomb, P/Esc to pause.

**Mouse**: Glides player toward pointer when hovering canvas (line 836–861). Loses priority if keyboard/joystick is active.

**Touch**: 
- Left zone for movement joystick (draggable, visible base + stick)
- Right zone with FIRE and BOMB buttons
- Events have `passive: false` to prevent scroll

### Boss AI

**Boss Patterns** (line 1372–1409): Six design templates (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with three attack patterns:
- Radial bursts
- Aimed/homing spreads
- Special patterns (teleport, column fire, spirals, crosses)

**Attack Flow** (line 1421–1434):
1. Telegraph phase (22 frames) with visual feedback (white core, audio cue)
2. Fire pattern based on current stage and pattern index
3. 85-frame cooldown between attacks
4. Pacing prevents input feel from being overwhelming

**Boss Stats** (line 1043–1056): Health = `(70 + stage * 35) * difficultyMul * bossDesignMul`. Each design has color scheme and core glow.

### Audio System

**WebAudio Synth** (line 542–676): No external sound assets. All SFX generated with oscillators, filters, noise bursts, and envelopes.

- `beep()`: Frequency-swept square/sawtooth/triangle tones (shoot, hit, power-up, boss patterns)
- `noiseBurst()`: White/filtered noise (explosions, bomb)
- Gain envelope: fast attack, exponential decay
- Stereo panning with `panFor(x)` based on on-screen position
- Dynamic compressor prevents clipping
- Mute state persists in localStorage

### State Management

**Persistent Settings** (localStorage, lines 463–527):
- `bestScore`: High score
- `muted`: Audio mute toggle
- `difficulty`: Selected difficulty
- `reducedMotion`: Animation intensity (respects system `prefers-reduced-motion`)

**Game State** (line 863–869): Top-level variables for score, lives, stage, combo counter, multiplier. Reset on each game start in `resetGame()`.

**UI Sync**: HUD elements updated directly in `update()` or on events (weapon level, hearts, bombs). No separated model layer.

## Development Workflow

### Running the Game
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Start screen shows difficulty buttons and controls hint
3. Click "Start Game" or press Return to begin

### Testing Locally
- No build step required
- No external dependencies (vanilla JS + Canvas + WebAudio)
- Test on desktop (mouse/keyboard) and mobile (touch) via device or DevTools emulation

### Code Changes Patterns

**Adding a New Enemy Type**:
1. Add entry to `ENEMY_TYPES` object (line 502–506)
2. Add drawing logic in `draw()` (line 1603–1613)
3. Update enemy spawning logic in `spawnEnemy()` if needed

**Modifying Boss Patterns**:
1. Add pattern function to `BOSS_PATTERNS` array (line 1372–1409)
2. Patterns call helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. All projectiles pushed to `enemyBullets` array
4. Boss HP scaled by `bossHpMul` in difficulty settings

**Tuning Difficulty**:
- Adjust `DIFFICULTIES` object (spawn rates, enemy speed, shoot chance, HP multipliers)
- Ramping is controlled by `Math.floor(frame / 300) * 4` in spawn throttle (line 1201)

**Visual Polish**:
- Themes defined in `THEMES` array
- Animations use `@keyframes` CSS
- Screen shake magnitude and duration in `screenShake()` function
- Particle system controlled by spawn count and velocity in `spawnExplosion()`

### Performance Optimization

**Adaptive Quality** (line 473–483): System tracks average frametime over 45 frames.
- Quality 0 (full): All glow effects, full particle count, shadow rendering
- Quality 1 (medium): 60% particles, reduced shadows
- Quality 2 (low): 35% particles, no shadows
- Escalates automatically if frametime exceeds target

**Optimization Flags**:
- `quality` checked before expensive shadow/glow operations (line 1579–1602)
- Particle count scaled by quality in `spawnExplosion()` (line 1074)
- Reduced-motion mode disables CSS animations and reduces screen shake

### Key Constants

- **Canvas size**: 480×720 (BASE_W, BASE_H). Scaled to fit window up to 1.6× (line 757)
- **Kills per stage**: 18 (KILLS_PER_STAGE, line 491). Boss spawns after this many enemy kills
- **Combo timeout**: 2.2 seconds (chainTimer, line 1089). Chain multiplier resets if no kills within window

## Commits & Changes

When making changes:
- Test both desktop (mouse/keyboard) and mobile (touch) input modes
- Verify audio initializes on first interaction (`ensureAudio()`)
- Check framerate impact in DevTools (Quality may auto-adjust)
- Ensure animations respect reduced-motion preference for accessibility
- Use descriptive commit messages referencing mechanic/feature changed
