# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a self-contained HTML5 vertical shmup (shoot-'em-up) game. The entire game—HTML, CSS, and JavaScript—is in a single `index.html` file (~1670 lines). It features:

- Multiple enemy types (grunt, interceptor, tank) with progressive difficulty
- 6 unique boss designs with distinct attack patterns
- Weapon upgrade system (Vulcan guns → Laser, 3 levels each)
- Smart bomb mechanic
- Multiplier/combo system based on kill streaks
- 6 dynamic sky themes with parallax clouds
- Touch controls for mobile + keyboard/mouse support for desktop
- Adaptive performance quality scaling based on frame timing
- Web Audio API synthesized sound effects (no external audio files)
- Difficulty settings (Easy/Normal/Hard) with persistent localStorage

## Running the Game

No build step required. Simply open `index.html` in a web browser:

```bash
# Desktop browser
open index.html

# Or serve locally if using web features that need HTTPS/origin
npx http-server -p 8000
# Then visit http://localhost:8000/index.html
```

The game runs completely client-side with no dependencies or external assets.

## Key Architecture

### Game State & Loop

The game is a single-threaded state machine driven by `requestAnimationFrame`. The main loop (`loop()` at line 1640):

1. **Update phase** (`update(dt)`) — handles input, movement, collisions, spawning
2. **Draw phase** (`draw()`) — renders all entities to canvas

State is maintained in global variables (e.g., `player`, `enemies`, `bullets`, `boss`). These are reset in `resetGame()` when the player starts.

### Entity Architecture

Entities are plain objects with position (`x`, `y`), dimensions (`w`, `h`), and type-specific properties:

- **Player** — `speed`, `cooldown` (fire rate), `weapon`, `level`, `invuln` (hit cooldown)
- **Enemies** — `speed`, `hp`, `shootTimer`, `pattern` (spread or aimed), `type` (grunt/interceptor/tank)
- **Bullets** — player: `speed`, `vx` (spread angle), `pierce` (laser penetration); enemy: `speed`, `vx`/`dy` (radial movement)
- **PowerUps** — `type` (life/bomb/weapon), `speed` (downward)
- **Particles** — visual feedback: `life`, `vx`/`vy` (velocity)

### Collision & Cleanup

- `rectsOverlap()` (line 1069) — AABB collision detection
- Bullets removed after leaving screen bounds
- Enemies removed after leaving bottom of screen
- Player invulnerability window after hit prevents rapid damage stacking

### Difficulty & Progression

Three difficulty levels defined in `DIFFICULTIES` object (line 485) control:
- Enemy spawn rate, speed, and shoot chance
- Player starting lives and bomb count
- Boss HP multiplier

**Stages** progress after killing 18 enemies (`KILLS_PER_STAGE` = 18 at line 491). Boss spawns after reaching stage threshold. Boss designs cycle through 6 patterns in `BOSSES` array (line 508). Each has unique HP multiplier.

### Boss Patterns

Boss attack patterns are in `BOSS_PATTERNS` array (line 1372). Each boss has 3 attack patterns that cycle:
- **Radial fire** — bullets spread in all directions
- **Aimed spread** — fan of bullets tracking toward player
- **Teleport + special** — boss repositions then fires variant pattern
- **Columns/walls** — static or moving bullet walls
- **Homing** — seeded toward player position with scatter

Pattern functions populate `enemyBullets` array. Boss telegraph animation (line 1421) plays before each attack.

### Weapon System

Two weapons scale across 3 levels:

**Vulcan** (default):
- Level 1: Single bullet straight up
- Level 2: Dual bullets (-0.12, +0.12 angle)
- Level 3: Triple spread (-0.24, 0, +0.24)

**Laser** (switched via weapon power-up):
- Level 1: Single laser lane
- Level 2: Two lanes (±6 offset)
- Level 3: Three lanes with pierce=2 (penetrates 2 enemy hits)

Weapon stats displayed in HUD; laser has longer range and faster fire rate but travels up immediately.

### Audio (Web Audio Synth)

All sound effects are synthesized using Web Audio API in the `sfx` object (line 624):
- `beep()` — oscillator with sweep and filter
- `noiseBurst()` — noise (for explosions)
- Each SFX function handles unique timings and frequencies

Audio context lazy-initialized on first user input (`ensureAudio()`). Mute state stored in localStorage.

### Performance Adaptation

`trackPerf()` (line 476) monitors frame times and adjusts quality level (0=full, 1=medium, 2=low):
- Full quality: all particle counts, shadow effects, glow effects
- Medium: 60% particle count
- Low: 35% particle count

Quality affects particle counts and shadow rendering at draw time.

### Input Handling

- **Keyboard**: Arrow keys or WASD to move; Space to fire; X/Shift to bomb; P/Esc to pause
- **Mouse**: Move to steer (smooth glide toward cursor); click to fire
- **Touch**: Left side joystick for movement; right-side FIRE/BOMB buttons
- **Precedence**: Keyboard/joystick input takes priority over mouse when active

Joystick uses touch events and velocity constraints (max 50px radius).

### UI State & Persistence

localStorage keys (defined at line 463):
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — sound mute state
- `skyfighter.difficulty` — last selected difficulty
- `skyfighter.reducedMotion` — accessibility: disable screen shake, flash, animations

The overlay (game-over/pause screen) is toggled via `.hidden` property. HUD updates are synchronous via `.textContent` assignments.

### Rendering Pipeline

Canvas rendering order (in `draw()` at line 1520):

1. Sky gradient with parallax sun glow
2. Parallax clouds (two layers at different speeds)
3. Player ship with engine flame (when firing)
4. Bullets (player: yellow/cyan with optional glow; enemy: red circles)
5. Enemies (plane silhouettes with tank health bars)
6. Boss ship (if present)
7. Power-ups (rotating squares with glow)
8. Particles (explosion debris, fading)
9. Screen shake applied via context translate

Theme data at line 493 defines 6 sky gradients, cloud colors, and sun colors. Theme cycles per stage.

## Development Notes

### Modifying Game Balance

- **Difficulty curves**: Adjust spawn rate ramping in `update()` line 1201 (`difficultyRamp`)
- **Enemy stats**: ENEMY_TYPES object (line 502) — adjust `hp`, `speedMul`, `score`
- **Boss HP**: Modify `baseHp` formula at line 1046 or `hpMul` in BOSSES array
- **Weapon damage**: Enemy bullets deal 1 HP; bullets use `pierce` property; boss bullets deal 3 HP
- **Combo/multiplier**: `addScore()` line 1087 chains up to 10x; adjust `chainTimer = 2.2` for window length

### Adding Enemies or Bosses

1. New enemy type: Add to `ENEMY_TYPES`, set `minStage` to gate appearance
2. New boss: Add to `BOSSES` array with color scheme, add pattern functions to `BOSS_PATTERNS` at matching index

### Audio Tweaking

See `beep()` and `noiseBurst()` parameters (frequency, duration, amplitude, filters). SFX timing is critical—test by calling functions directly in console.

### Mobile Responsiveness

Canvas sizing handled in `resize()` (line 754). Max scale is 1.6x. Safe-area insets applied for notched devices. Touch zones (left 55% for movement, right 45% for actions) defined at line 299–308.

### Accessibility

- Reduced motion preference respected (`prefers-reduced-motion` media query at line 520)
- Screen reader live region (`#srLive`) for announcements
- Pause button muted during boss telegraph to allow audio cue
- Color contrast checked in UI elements; critical states (1 life) have visual + audio cues

## File Structure

The entire codebase is in `index.html`:
- Lines 1–360: HTML structure + inline CSS
- Lines 422–1670: JavaScript game engine

No external dependencies. Game requires only HTML5 Canvas 2D API, Web Audio API, and localStorage.
