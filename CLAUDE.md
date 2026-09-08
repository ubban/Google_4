# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. It features procedurally generated sound effects (no external audio assets), adaptive quality rendering, multiple difficulty levels, and six boss enemies with distinct attack patterns.

## Running the Game

- Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
- No build step, dependencies, or server required—it's a pure vanilla JavaScript game
- Game runs at ~60 FPS with adaptive performance (quality degrades gracefully on slower devices)

## Game Architecture

The entire game runs inside a single IIFE (Immediately Invoked Function Expression) that manages three interconnected systems:

### 1. **Input System** (Lines 767–862)
- **Keyboard**: Arrow keys + WASD for movement, Space/click to fire, X/Shift/P for bomb/pause
- **Mouse**: Hover over canvas to steer; click to fire; uses smooth interpolation when keyboard isn't active
- **Touch**: Full joystick support on left side (movement), fire and bomb buttons on right
- The three input methods are mutually exclusive by design: keyboard/joystick > mouse > idle

### 2. **Rendering System** (Lines 520–1638)
- **Canvas 2D**: 480×720 base resolution, automatically scales to fit viewport (up to 1.6× on large screens)
- **Performance-adaptive rendering**: `quality` variable (0=full, 1=medium, 2=low) disables glow effects and reduces particle counts to maintain 60 FPS
- **Parallax clouds**: Two layers scroll at different speeds; skipped entirely on `quality >= 1`
- **Theme rotation**: Six visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) advance per stage

### 3. **Game Loop** (Lines 1640–1665)
- `requestAnimationFrame` continuously calls `loop()` which drives `update()` then `draw()`
- `update(dt)` advances game state (player position, enemy AI, collision detection, scoring)
- Capped at 50ms delta time to prevent catastrophic jumps on tab switches
- Audio context is lazy-initialized on first input (browser autoplay restriction)

## Key Game Systems

### Difficulty Settings (Lines 485–489)
Four parameters scale per difficulty (easy/normal/hard):
- `spawnBase/spawnMin`: Enemy spawn rate (seconds between spawns)
- `enemySpeed`: Enemy movement multiplier
- `enemyShootChance`: Probability an enemy fires bullets
- `lives`, `bombs`, `bossHpMul`: Resource scaling

These are stored in localStorage so player preference persists.

### Enemy Types (Lines 502–506)
- **Grunt** (red, 1 HP): Slow, high-frequency spawn; appears at stage 1+
- **Interceptor** (orange, 1 HP): Fast, weaving pattern; appears at stage 1+
- **Tank** (purple, 4 HP): Tanky, draws health bar; appears at stage 2+

Enemies spawn randomly within the top-left to top-right of the canvas and descend. Interceptors weave left/right using sine waves. Enemies fire either in a spread pattern or aimed at the player.

### Weapon System (Lines 1124–1146)
- **Vulcan** (spread bullets): Level 1 (single center shot) → Level 2 (dual offset) → Level 3 (triple spread)
- **Laser** (piercing beams): Level 1 (single center beam) → Level 2 (dual offset) → Level 3 (triple + center)
- Weapon pickups toggle between vulcan/laser and reset to level 1; level pickups inside the current weapon advance it

### Scoring & Combo (Lines 1087–1101)
- Each enemy kill adds `base_score × multiplier` to the score
- `chain` counter increments on every kill; `multiplier = 1 + floor(chain / 5)`, capping at ×10
- Chain timer resets on player hit; player has 2.2 seconds to continue the combo
- Boss defeats award 500 bonus points

### Boss System (Lines 508–515, 1043–1056, 1372–1434)
- Six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle through stages
- Each boss has a unique color scheme and HP multiplier (0.8× to 1.7×)
- Boss enters with 2-second delay and flies in from top until reaching y=120
- Each boss has 3 unique attack patterns that repeat; telegraph effect (core glows white for 22 frames) signals each attack
- Bullets from bosses use radial motion (dx/dy velocity) rather than linear

## Important Constants & Configuration

| Constant | Location | Purpose |
|----------|----------|---------|
| `BASE_W`, `BASE_H` | 753–754 | Canvas resolution (480×720) |
| `KILLS_PER_STAGE` | 491 | Kills required to trigger boss fight (18) |
| `THEMES` | 493–500 | Visual themes: sky gradient, cloud color, sun color |
| `DIFFICULTIES` | 485–489 | Spawn rates, enemy behavior, lives/bombs per difficulty |
| `BOSSES` | 508–515 | Boss designs and HP multipliers |
| `BOSS_PATTERNS` | 1372–1409 | Boss attack sequences (6 patterns, 3 attacks each) |
| `ENEMY_TYPES` | 502–506 | Enemy stats: size, HP, speed, score |

## Storage Keys (Lines 463–466)

Game state is persisted in localStorage:
- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Mute toggle (1/0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Screen shake/flash toggle (1/0)

## Audio System (Lines 542–676)

All sound is procedurally generated via Web Audio API:
- **Oscillators** (sine, square, sawtooth, triangle): Pitched sounds
- **Noise bursts**: Unfiltered white noise with envelope
- **Filters**: Lowpass and highpass dynamic filters
- **Panning**: Spatial audio that pans left/right based on bullet position on screen
- **Envelope**: Attack, sustain, release applied via gain nodes

Sounds are only played if `muted === false` and `audioCtx` is available. Master gain is 0.9 with dynamic range compression.

## Collision Detection (Lines 1069–1071, 1232–1294)

Axis-aligned bounding box (AABB) collision: `Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2`

Collision pairs checked per frame:
1. Bullets vs. enemies (bullets deal 1 damage; lasers pierce based on `pierce` counter)
2. Bullets vs. boss (boss takes 3 damage per hit)
3. Enemies vs. player (player loses 1 life, gains 1.5s invulnerability)
4. Enemy bullets vs. player (same as above)
5. Boss vs. player (same as above)
6. Power-ups vs. player (apply effect)

## Common Modifications

### Adjusting Difficulty Parameters
Edit `DIFFICULTIES` object (line 485). Key tuning variables:
- `spawnBase`: Lower = more enemies spawn
- `enemySpeed`: Speed multiplier (1.0 = base speed)
- `bossHpMul`: Boss health multiplier

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add corresponding explosion sound to `EXPLODE_SFX` (line 678)
3. Update `spawnEnemy()` (line 1022) to include your type in the random pool

### Adding a New Boss Attack Pattern
1. Create a function like `fireRadial(b, count, speed)` (line 1324)
2. Add the function call sequence to `BOSS_PATTERNS` (line 1372) in the appropriate boss index
3. Each pattern should populate `enemyBullets` array directly

### Tweaking Visual Theme
Edit `THEMES` array (line 493) to add new sky gradients, cloud colors, or sun effects. Each theme is applied per `(stage - 1) % THEMES.length`.

## Performance Considerations

- **Frame time tracking** (lines 476–483): Averages 45 frames; if average > 25ms, quality decreases; if < 17.9ms, quality increases
- **Particle limits**: Full quality spawns up to 14 particles per explosion; medium quality spawns ~60% of that; low spawns ~35%
- **Shadow rendering**: Only applied at `quality === 0` (full quality)
- **Touch device detection** (line 778): Enables on-screen joystick and buttons if `ontouchstart` or `navigator.maxTouchPoints > 0`

## Accessibility Features

- **Reduced motion**: Toggle via settings button (top-right 🎆); disables all CSS animations and camera shake
- **Screen reader support** (lines 390, 468–471): `#srLive` element with `role="status"` and `aria-live="polite"` announces game events
- **Mute button**: 🔊 (top-right); silences all audio
- **High-contrast UI**: Game layer is opaque dark (#03050d) with bright text and glow effects
