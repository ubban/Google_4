# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1670 lines).

## Development & Deployment

### Running the Game
- **Local development**: Open `index.html` directly in any modern web browser
- **Live deployment**: Upload `index.html` to any static web host (GitHub Pages, Netlify, etc.)
- **No build step required** — the project has zero dependencies and no build tools

The game uses the `BASE_W = 480` and `BASE_H = 720` canvas dimensions, which scale responsively. It works on:
- Desktop browsers (mouse steering, click to fire)
- Mobile browsers (touch joystick for movement, on-screen fire/bomb buttons)
- Keyboard (arrows/WASD for movement, space to fire, X/Shift for bomb, P/Esc to pause)

## Architecture Overview

The game runs a single `requestAnimationFrame` loop that calls `update(dt)` for game logic and `draw()` for rendering. The code is organized as a single IIFE (Immediately Invoked Function Expression) with these main sections:

### Game State
- **Player**: position, weapon (vulcan/laser), level (1-3), lives, invulnerability frames, cooldown
- **Enemies**: array of enemy objects with position, type, HP, behavior (spread/aimed shooting)
- **Bullets**: player bullets with piercing mechanics for lasers; enemy bullets use simple physics
- **Boss**: optional boss object with telegraph delay before attacking, multiple attack patterns per boss design
- **Score & Progression**: score, combo chain/multiplier, lives, bombs, stage tracking

### Core Systems

#### Input Handling
- **Keyboard**: keys object tracks down state; Space fires, X/Shift bombs, P/Esc pauses
- **Mouse**: `mouseTarget` tracks pointer position; player glides toward it when keyboard input is idle
- **Touch**: multi-touch joystick for movement (left zone), separate fire and bomb buttons (right zone)
- **Priority**: keyboard/joystick input overrides mouse steering when active

#### Rendering
- **Canvas**: 480×720 base resolution, scales to fit viewport up to 1.6× native
- **Themes**: 6 cyclic sky themes (backgrounds) that rotate per stage with parallax cloud layers
- **Drawing order**: sky gradient → sun glow → clouds → player → enemies → bullets → particles → boss
- **Performance**: adaptive quality setting (0=full, 1=medium, 2=low) adjusts particle counts and shadow blur based on frame time

#### Audio (WebAudio Synthesis)
- **No audio files** — all sound effects generated in real-time using the Web Audio API
- **Synth functions**: `beep()` for pitched tones with optional frequency sweep and filters; `noiseBurst()` for noise
- **Panning**: bullets and explosions pan left/right based on screen position
- **Volume control**: master gain at 0.9; compressor to prevent clipping
- **Muting**: toggle via `muteBtn`; stored in localStorage

#### Enemy Spawning & Difficulty
- **Difficulty modes** (stored in localStorage):
  - Easy: `spawnBase=75`, `enemySpeed=1.0`, lower shoot chance, 4 lives, 3 bombs
  - Normal: `spawnBase=58`, `enemySpeed=1.4`, medium shoot chance, 3 lives, 2 bombs
  - Hard: `spawnBase=40`, `enemySpeed=1.9`, high shoot chance, 3 lives, 2 bombs
- **Enemy types**: grunt (1 HP), interceptor (1 HP, fast, weaving), tank (4 HP, slow, heavily armored)
- **Minions per stage**: stage progresses after 18 enemy kills
- **Difficulty ramping**: spawn timer decreases over time (every 300 frames)

#### Weapons
- **Vulcan**: immediate fire rate (0.15s cooldown), bullet spread increases with level (1→2→3 bullets)
- **Laser**: slower rate (0.1s cooldown) but pierces enemies (1→2→3 pierce levels); creates glow aura
- **Power-up drops**: weapon type (6%), bomb (8%), life (6%), random spawn from killed enemies

#### Combo System
- **Chain**: killing enemies within 2.2 seconds extends the combo
- **Multiplier**: x1 base, increases by 1 for every 5 kills (max x10)
- **Break**: missing kills for 2.2s or taking damage resets chain to zero

#### Boss Battles
- **Spawn**: triggers after 18 minion kills per stage
- **Patterns**: 6 unique bosses, each with 3 attack patterns (radial, aimed spreads, teleport dashes, columns, double rings, etc.)
- **HP scaling**: `(70 + stage * 35) * difficulty.bossHpMul * design.hpMul`
- **Telegraph**: 22-frame warning before each attack; core glows white during telegraph
- **Movement**: bosses patrol left/right within canvas bounds; entering stage lasts until y=120

#### Particles & Effects
- **Explosion particles**: spawn on enemy death and boss hit; adaptive count based on quality setting
- **Screen shake**: triggered by bomb use, boss defeat, player hit
- **Flash overlays**: white flash on hit, colored flash on bomb
- **Reduced motion**: respects `prefers-reduced-motion` media query; disables all animations when enabled

### Constants & Configuration

**Key tuning constants** (search for these to modify):
- `BASE_W = 480`, `BASE_H = 720` — canvas dimensions
- `KILLS_PER_STAGE = 18` — enemies to defeat before boss appears
- `DIFFICULTIES` — spawn rates, enemy speed, shoot chance, HP multipliers
- `THEMES` — 6 background/theme configurations (colors, sun position)
- `ENEMY_TYPES` — stats for grunt, interceptor, tank (HP, speed, color, score)
- `BOSSES` — 6 boss designs with hull colors, core colors, HP multipliers
- `BOSS_PATTERNS` — attack pattern functions for each boss

### LocalStorage Keys

The game persists:
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — mute state
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — reduced motion preference

## Common Editing Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
2. Update `spawnEnemy()` to include it in the random choices
3. Add sound effect to `EXPLODE_SFX` object
4. Create explosion SFX function in the `sfx` object

### Adding a New Boss
1. Add entry to `BOSSES` with `name`, hull colors, core color, `hpMul`
2. Create a 3-pattern attack list in `BOSS_PATTERNS` (or copy existing patterns from another boss)
3. No drawing changes needed — `drawBossShip()` automatically uses design colors

### Adjusting Game Balance
- **Enemy difficulty**: modify `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance` in `DIFFICULTIES`
- **Boss difficulty**: scale via `bossHpMul` in `DIFFICULTIES` or `hpMul` in `BOSSES`
- **Player power**: adjust `fireWeapon()` bullet spread, laser pierce count, fire cooldown
- **Progression speed**: change `KILLS_PER_STAGE` (fewer kills = more frequent bosses)

### Adding Visual Polish
- Animations and effects use the `frame` counter and simple trigonometry
- `shakeTime` and `shakeMag` control screen shake magnitude and duration
- Glow effects controlled by `quality` setting (0 = max glow; only in 60fps conditions)
- Cloud parallax speeds controlled in `drawClouds()` calls

## Code Structure Notes

- **No transpilation or minification** — vanilla ES6, but entirely compatible with older browsers (supports IE11 except for optional chaining)
- **Canvas rendering**: all drawing is immediate mode (no retained graphics); frame-by-frame performance depends on particle count
- **Collision detection**: axis-aligned bounding box overlap (`rectsOverlap()`) for all objects
- **Game states**: `running`, `paused`, `gameOver` flags; overlay shown when paused or at game start/end
- **Touch handling**: multi-touch with identifier tracking prevents conflicts between joystick and buttons
- **Accessibility**: screen reader announcements via `#srLive` element for game events (lives lost, boss defeated, stage reached)

## Performance Considerations

- **Adaptive quality**: frame time tracked in 45-frame window; quality level auto-adjusts shadow blur and particle counts
- **Particle culling**: particles removed when life ≤ 0; bullets removed when off-canvas
- **Enemy culling**: enemies removed when y > canvas.height + 30
- **Canvas size**: fixed at 480×720 internally; CSS scaling handles DPI/viewport differences
- **Audio context**: lazily initialized on first input (not on page load)

## Testing the Game

No automated tests. Manual verification should cover:
1. **All three difficulty modes** — spawn rate, enemy behavior, boss HP, lives/bombs
2. **Both weapons** — vulcan spread, laser pierce, weapon power-up progression
3. **Boss patterns** — all 6 bosses should have distinct attack patterns
4. **Mobile input** — joystick movement, fire/bomb buttons on touch devices
5. **Audio** — muting toggle, synthesized SFX play correctly
6. **Scoring** — combo multiplier, stage progression, high score saving
7. **Persistence** — difficulty/mute/motion settings survive reload
