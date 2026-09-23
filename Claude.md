# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single self-contained HTML file. The game features:
- Player-controlled aircraft with weapon switching (Vulcan/Laser)
- Multiple enemy types with distinct behaviors
- Six unique boss ships with pattern-based attack systems
- Smart bomb mechanic for crowd control
- Progressive difficulty across stages
- Scoring chain/combo system with multipliers
- Touch controls for mobile devices and keyboard/mouse controls for desktop

## Getting Started

### Running the Game
Simply open `index.html` in a modern web browser. The game works offline and requires no build process or external dependencies.

### Development Environment
No special setup required. Make changes directly to `index.html` and refresh the browser to test.

## Architecture Overview

The game follows a simple single-file architecture with clear logical sections, all contained within an IIFE (Immediately Invoked Function Expression) to avoid global namespace pollution.

### Core Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw()
```

The main loop runs at 60fps. The `update()` function handles all game logic (player movement, collision detection, entity updates), and `draw()` renders the canvas each frame.

### Game State Structure

**Player State:**
- Position, velocity, dimensions
- Current weapon and level
- Lives, invulnerability timer, cooldown
- Bomb count

**Entity Collections:**
- `enemies[]` – enemy ships with AI
- `boss` – current boss ship (null when none active)
- `bullets[]` – player projectiles
- `enemyBullets[]` – incoming fire
- `powerUps[]` – collectable items (weapons, lives, bombs)
- `particles[]` – visual effects

**Game State:**
- `stage`, `stageKills`, `kills` – progression tracking
- `score`, `multiplier`, `chain` – scoring system
- `running`, `paused`, `gameOver` – game mode flags

### Key Systems

**1. Input Handling**
- Keyboard: Arrow keys/WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- Mouse: Move to steer, click to fire
- Touch: Left zone for joystick movement, right zone for fire/bomb buttons
- Priority: Keyboard/joystick input overrides mouse steering

**2. Collision Detection**
Simple rectangular overlap test: `rectsOverlap(a, b)` checks if axis-aligned bounding boxes intersect. Used for:
- Player-enemy collisions → lose life
- Player-enemyBullet collisions → lose life
- Player-powerUp collisions → apply effect
- Bullet-enemy collisions → damage/destroy
- Bullet-boss collisions → damage boss

**3. Difficulty System**
Three difficulty levels stored in localStorage, each affecting:
- Enemy spawn rate and minimum spawn interval
- Enemy movement speed
- Enemy shooting frequency
- Starting lives and bombs
- Boss health multiplier

**4. Weapon System**
Two weapon types:
- **Vulcan**: Spread shots (1, 2, or 3 bullets depending on level)
- **Laser**: Piercing shots (1, 2, or 3 lanes depending on level)

Switching weapons resets level to 1. Power-ups cycle between increasing weapon level and switching weapon types.

**5. Boss System**
Bosses spawn after 18 enemy kills per stage. Each boss has:
- A design (visual styling and HP multiplier)
- Attack patterns (array of firing functions)
- A telegraph phase before attacking (visual telegraph + sound cue)
- Movement (bouncing horizontally within bounds)

Six unique boss designs with distinct attack pattern sets, cycling through stages.

**6. Scoring & Chain System**
- Kills extend a chain timer (2.2 seconds)
- Chain resets on player damage
- Multiplier increases with chain: `1 + floor(chain/5)` (max 1.8x)
- Boss kill grants 500 points base
- Each kill multiplies base points by current multiplier

**7. Audio System**
Procedural WebAudio synthesis (no external asset files):
- `beep()` – sine/square/sawtooth tone with envelope and optional filter/sweep
- `noiseBurst()` – white noise with filter decay
- `sfx.*` – named sound effects for all game events
- Pan effects based on screen position for spatial audio
- Master compressor and gain control
- Mute toggle with localStorage persistence

**8. Visual Effects**
- Screen shake on impacts
- Flash effects (white flash on hits, larger on bombs)
- Combo pop-ups and stage banners with animations
- Reduced motion mode (toggleable) respects prefers-reduced-motion media query
- Vignette pulse effect when critical (1 life remaining)
- Particle explosions on all kills

**9. Adaptive Performance**
Tracks frame times over 45 frames and adjusts rendering quality:
- Quality 0: Full effects (shadows, glows, particle counts)
- Quality 1: Medium effects (~60% particles)
- Quality 2: Minimal effects (~35% particles)

Thresholds: degrade if avg frame time > 1/40s, upgrade if < 1/56s.

**10. Accessibility**
- Screen reader announcements via `srLive` region
- Keyboard and touch alternatives for all input
- Reduced motion toggle with system preference detection
- Safe area insets for notched devices (mobile)

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add sound effect to `sfx` object under `EXPLODE_SFX` if needed
3. Add spawn logic in `spawnEnemy()` where enemy types are selected by probability

### Adding a New Boss
1. Add boss design to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` at matching index
3. Boss will cycle through designs based on `(stage - 1) % BOSSES.length`

### Adding Boss Attack Patterns
Attack patterns are functions that call firing functions:
- `fireRadial(b, count, speed)` – bullets in circle
- `fireAimedSpread(b, count, speed, spreadStep)` – fan centered on player
- `fireColumns(b, speed)` – vertical columns
- `fireSpiralBurst(b, speed)` – 8-way expanding spiral
- `fireCross(b, speed)` – 8-direction cross
- `fireHomingSeeded(b, count, speed)` – aimed with jitter
- `fireDoubleRing(b, speedA, speedB)` – concentric rings

Each can be called with unique parameters in attack pattern functions.

### Adding a Theme
1. Add entry to `THEMES` array with: `sky` (3-stop gradient colors), `cloud`, `sun`, `name`
2. Themes cycle via `(stage - 1) % THEMES.length`

### Tweaking Game Balance
Key tunable constants:
- `KILLS_PER_STAGE` – kills before boss (currently 18)
- `DIFFICULTIES` – per-difficulty spawn/enemy/boss stats
- `ENEMY_TYPES` – individual enemy balance (hp, speed, score)
- Fire rates in `fireWeapon()` – weapon cooldown (0.1s laser, 0.15s vulcan)
- `updateBoss()` – boss movement speed and attack timing

### Sound Design Notes
Audio is synthesized with frequency, duration, waveform, volume, and optional filters:
- High frequencies (800+Hz) feel sharp/electronic
- Low frequencies (100-300Hz) feel heavy/impactful
- Sawtooth = harsh/edgy, sine = smooth, square = retro
- Sweep down (sweepTo < freq) feels like energy dissipating
- Noise bursts add "crunch" to explosions

## Testing

Since there's no test suite, manually verify:
- **All input methods**: keyboard, mouse, touch on a mobile device
- **Difficulty modes**: verify enemy spawn rates and boss HP differ
- **Collision detection**: test edge cases (bullets at boundaries, boss edge hits)
- **Audio**: test mute toggle works, spatial panning on left/right kills
- **Mobile responsiveness**: test on various screen sizes and orientations
- **Reduced motion**: verify animations are disabled, performance improves

## Performance Considerations

- Particle limits scale with quality level to maintain 60fps on lower-end devices
- No shadow effects or glows rendered below quality 1
- Canvas is 480x720 (logical), scaled via CSS for display
- Enemy/bullet/particle arrays are pruned each frame to remove off-screen objects
- Joystick visualization is hidden by default, shown only on touch start
- High frame count tracking (frame variable) used for animations and patterns

## Browser Compatibility

Requires:
- Canvas 2D context
- Web Audio API (graceful fallback if unavailable)
- Touch Events (for mobile)
- localStorage (for persistence)
- CSS Grid/Flexbox, CSS variables (modern browser features)

Tested on recent versions of Chrome, Firefox, Safari, Edge. Mobile browsers: iOS Safari, Chrome Android.
