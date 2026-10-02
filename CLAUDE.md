# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single-file HTML5 Canvas application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` with no external dependencies or build step required.

## Architecture Overview

### Canvas-Based Game Engine
The game uses a simple game loop pattern:
- `requestAnimationFrame(loop)` drives all game updates at 60 FPS
- `update(dt)` handles game logic and physics each frame
- `draw()` renders everything to canvas after updates
- Delta time (`dt`) is clamped to max 0.05s to prevent jump on slow frames

### Game State Model
Core game state lives in module-scope variables:
- **Player**: position, dimensions, weapon, health, invulnerability timer
- **Bullets/Enemies/Enemy Bullets**: arrays of entities with position, velocity, hit detection
- **Game Flow**: `running`, `paused`, `gameOver` flags; `score`, `lives`, `stage`, `kills` counters
- **Particles**: temporary explosion effects for visual feedback
- **Boss**: optional entity during boss stages with telegraphed attack patterns

### Performance Adaptation
The `quality` variable (0=full, 1=medium, 2=low) auto-adjusts based on average frame time. Lower quality reduces particle counts and disables shadow effects on canvas, reducing render overhead.

## Game Systems

### Difficulty Scaling
Three difficulty presets (easy/normal/hard) in `DIFFICULTIES` control:
- Enemy spawn rates and minimum spawn intervals
- Enemy speed and shooting frequency
- Player health/bomb counts
- Boss HP multiplier
- Persisted in localStorage under `skyfighter.difficulty`

Six dynamic themes cycle per stage in `THEMES` (sky gradients, cloud colors, lighting) picked by `(stage - 1) % THEMES.length`.

### Enemy Types
Defined in `ENEMY_TYPES`:
- **Grunt**: basic red fighter, stage 1+
- **Interceptor**: fast orange weaver, stage 1+, sinusoidal flight pattern
- **Tank**: heavy purple unit, stage 2+, 4 HP, dangerous
- Each type has its own explosion SFX via `EXPLODE_SFX` mapping

Enemies pick random behaviors: `spread` pattern (3-shot spread) or `aimed` (heat-seeking toward player). Interceptors weave using a sine wave offset.

### Weapon System
Two weapons toggle via power-ups after reaching max level (3):
- **Vulcan**: bullet spread pattern, fire rate 0.15s per shot
  - Level 1: single center bullet
  - Level 2: spread to ±0.12 radians
  - Level 3: wide spread ±0.24 radians
- **Laser**: piercing beam, fire rate 0.1s per shot
  - Level 1: single lane
  - Level 2: dual lanes (±6 offset)
  - Level 3: triple lanes (dual + center)

### Boss System
Six boss designs in `BOSSES` (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage. Each has:
- Unique color palette (hull top/bottom, wings, core glow)
- HP multiplier (0.8–1.7 relative to stage)
- 3-attack pattern pulled from `BOSS_PATTERNS` array (position-matched to boss index)

Boss telegraphs attacks: 22-frame telegraph window with visual/audio cue before firing pattern. Attack patterns use helper functions:
- `fireRadial(b, count, speed)`: full 360° circle
- `fireAimedSpread(b, count, speed, spreadStep)`: cone at player with spread
- `fireColumns(b, speed)`: vertical downward columns
- `fireSpiralBurst(b, speed)`: rotating spiral
- `fireHomingSeeded(b, count, speed)`: homing missiles with randomness
- `fireDoubleRing(b, speedA, speedB)`: two concentric rings

### Chain/Multiplier System
- Killing an enemy increments `chain`; resets on miss/hit after `chainTimer` expires (2.2s)
- Multiplier = `1 + Math.min(9, Math.floor(chain / 5))` (max 10x at 50+ kills)
- Score gained per kill = base score × multiplier

### Input Handling
Four input modes (simultaneous, not exclusive):
1. **Keyboard**: Arrow keys + A/D for movement; Space for fire; X/Shift for bomb
2. **Touch Joystick**: Left-side drag to move (joystick visual appears); right buttons
3. **Mouse**: Plane glides toward pointer; click to fire
4. **Touch Buttons**: Mobile FIRE and BOMB buttons

Keyboard/joystick movement takes priority over mouse steering. Audio context resume on first user input.

### Audio (WebAudio Synthesis)
No external audio files. All sounds generated via Web Audio API:
- `beep(freq, dur, type, vol, opts)`: oscillator-based tones with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: noise filtered via biquad filter for explosions
- `sfx` object holds named sound effects (shoot, laser, explode*, hit, bomb, powerup, bossTelegraph, etc.)
- Master gain at 0.9; dynamicsCompressor for loudness limiting
- Muted state persisted in localStorage; respects system `prefers-reduced-motion`

### Screen Shake
`screenShake(mag, dur)` applies random translation each frame for `dur` seconds. Respects reduced motion preference (dampened to 15% intensity).

### UI & Rendering

**HUD Elements** (position: fixed, z-index layer):
- Score/Best/Multiplier/Weapon stats in capsules with backdrop blur
- Hearts (❤️) and bombs (💣) display with opacity toggle for state
- Boss health bar (when active)
- Combo pop animation at stage center
- Stage banner on entry (smooth fade-in/out)

**Canvas Rendering** (stored as style.width/height, scaled from 480×720):
- Adaptive DPI scaling: `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)` (capped at 1.6x)
- Three-layer sky gradient per theme with sun glow and scrolling cloud layers
- Plane sprite: triangular fuselage + wings + cockpit highlight (drawn via `drawPlane()`)
- Boss sprite: elliptical hull + wings + animated core glow (drawn via `drawBossShip()`)
- Particles: circles scaled by remaining life fraction, fading alpha
- Glow effects (shadow/shadowBlur) only at `quality === 0` to reduce overhead

**Overlay Modes**:
- Start screen: difficulty buttons, instructions, best score
- Game Over: final score, stage, new best indicator
- Pause: title card with resume button
- Rendered via show/hide `overlay` div with glass-morphism card style

## Input Flow & Common Tasks

### To Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add spawn logic in `spawnEnemy()` to pick it by probability or stage gate
3. Add explosion SFX in `EXPLODE_SFX` mapping if custom behavior needed
4. Collision/bullet damage handled generically in update loop

### To Add a New Boss
1. Add to `BOSSES` array with name, hull colors, core color, HP multiplier
2. Add 3-attack pattern function set to `BOSS_PATTERNS[index]` matching boss position
3. Boss HP scales: `baseHp * d.bossHpMul * design.hpMul` where baseHp = `70 + stage * 35`
4. Boss sprite drawn via `drawBossShip()` using colors from design object

### To Modify Game Balance
- **Difficulty curves**: tweak spawn rates, enemy speed, HP in `DIFFICULTIES`
- **Stage progression**: `KILLS_PER_STAGE = 18` kills required before boss
- **Power-up drops**: adjust probabilities in `maybeSpawnPowerUp()` (6% life, 8% bomb, 20% weapon)
- **Weapon fire rates**: `fireDelay` in update loop (0.1s laser, 0.15s vulcan)

### To Test Locally
1. Open `index.html` directly in a web browser (no server needed)
2. Game state persists to localStorage; clear browser storage to reset high scores
3. Toggle audio/motion/debug via buttons in top-right
4. Resize browser window to test responsive scaling

## Storage & Persistence

localStorage keys under `skyfighter.*` namespace:
- `bestScore`: high score (number)
- `muted`: audio mute state (1 or 0)
- `difficulty`: selected difficulty (string: easy/normal/hard)
- `reducedMotion`: animation preference override (1/0, or unset to follow system)

## Performance Considerations

1. **Frame rate handling**: delta time clamped to 0.05s max to prevent spiral on lag
2. **Particle limits**: Reduce per-explosion based on `quality` setting (full/60%, medium, low/35%)
3. **Canvas shadow effects**: Only rendered at `quality === 0` (full quality); shadow blur is heavy
4. **Enemy/bullet limits**: Arrays filter out off-screen entities; no pooling (acceptable for this scale)
5. **Screen shake**: Appliedrandom offset to ctx each frame; minimal cost

## Accessibility

- Screen reader live region (`#srLive`) announces state changes (lives, stage, high score)
- Reduced motion preference detected via `prefers-reduced-motion` media query; CSS animations disabled when active
- Button labels with title attributes for tooltips
- Font sizing responsive via clamp() for mobile-to-desktop ranges
- High contrast UI on dark background (WCAG AA compliant colors)

## Common Edits

**To add a new theme**: Insert object into `THEMES` with sky gradient colors, cloud color, sun color, and descriptive name. Used cyclically.

**To adjust canvas size**: Change `BASE_W` and `BASE_H` constants; all positioning scales proportionally.

**To add screen effects**: Use `screenShake()`, `flashHit()`, `flashBomb()`, `showComboPop()`, `showStageBanner()` for visual feedback.

**To tweak weapon behavior**: Modify spread angles in `fireWeapon()` or pierce/damage via `bullets` collision handler.

**To add sound effects**: Create new function in `sfx` object using `beep()` and `noiseBurst()` helpers.
