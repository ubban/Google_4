# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. It's a browser-based game with no external dependencies, build process, or asset pipeline. The entire game (HTML, CSS, and JavaScript) is contained in a single `index.html` file (1669 lines).

## Running the Game

### Development / Playing Locally
- Simply open `index.html` in a web browser (no server required)
- Works on desktop (keyboard + mouse) and mobile (touch controls)
- No build step or dependencies needed

### Testing / Debugging
- Open DevTools (F12) and check the Console for any errors
- The game auto-adjusts rendering quality based on frame rate performance
- localStorage is used to persist: high score, difficulty, mute state, reduced motion preference

## Game Architecture

### Core Loop & Rendering
- **Game Loop**: `requestAnimationFrame(loop)` - runs at 60fps
- **Canvas**: 480×720 logical resolution, scales to fill window (max 1.6× scale)
- **Rendering**: Canvas 2D context only (no WebGL)
- **Performance Tracking**: Frame timing tracked over 45 frames; quality level (0–2) reduces particle count if average frame time > 25ms

### Game State Objects

#### Player Object
- `x, y`: Position
- `w, h`: Dimensions (40×40)
- `speed`: Movement speed (260 px/s)
- `cooldown, rapid`: Weapon firing state
- `weapon`: 'vulcan' or 'laser'
- `level`: 1–3, affects shot pattern
- `invuln`: Invulnerability timer (set on hit)

#### Enemies
- Three types: `grunt` (weak), `interceptor` (fast), `tank` (tanky)
- Each has hp, speed, shoot behavior (spread vs. aimed patterns)
- Weaving movement on certain types
- Score multiplier for each type
- Min stage requirement (tanks appear stage 2+)

#### Boss Object
- Unique design for each stage (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Distinct color palette and attack patterns per boss
- Telegraph mechanic: 22-frame warning before attack
- Enters from top (y -80 to 120), then homes side-to-side
- HP scales with difficulty and boss design multiplier

#### Projectiles & Bullets
- **Player bullets**: Straight shots (vulcan) or piercing lasers
- **Enemy bullets**: Speed-based, can be radial, aimed, or homing-seeded
- **Boss bullets**: Complex patterns (spiral, cross, double rings, columns, etc.)
- All tracked in arrays: `bullets`, `enemyBullets`

#### Particles & Effects
- Explosion particles spawned on kills and hits
- Velocity-based movement with alpha fade
- Count reduced on lower quality settings

### Difficulty & Progression

**Difficulty Modes** (stored in localStorage)
- **Easy**: Lives 4, Bombs 3, slower spawn, weaker enemies, 0.8× boss HP
- **Normal**: Lives 3, Bombs 2, standard spawn, standard enemies, 1.0× boss HP
- **Hard**: Lives 3, Bombs 2, fast spawn, aggressive enemies, 1.3× boss HP

**Stage Progression**
- 18 kills per stage triggers boss spawn
- 6 bosses rotate; boss patterns have 3 variations each
- Themes rotate across 6 sky backgrounds per stage
- Difficulty gradually increases: spawn rate improves every ~5 seconds

### Audio System

**Web Audio API Synthesis** (no external files)
- `ensureAudio()`: Lazily initializes AudioContext on first user interaction
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: White noise with lowpass filter
- Master gain + dynamics compressor for loudness management
- Mute state stored in localStorage

**SFX Library**
- `shoot`, `laser`: Gun and laser weapon firing
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Enemy death sounds (type-specific)
- `hit`: Player hit
- `bomb`: Smart bomb detonation
- `powerup`: Weapon pickup
- `bossHit`, `bossTelegraph`, `bossDown`: Boss-specific sounds
- `stageClear`, `gameover`: Game state transitions

### Input Handling

**Keyboard**
- Arrow keys or WASD: Move
- Space: Fire
- X or Shift: Bomb
- P or Escape: Pause

**Mouse**
- Move: Steer player (smooth glide toward pointer)
- Click: Fire continuously

**Touch** (mobile)
- Left zone (55% width): Joystick input
- Right zone (45% width): Fire and Bomb buttons
- Joystick shows on first touch; constrained to 50px radius

**Priority**: Keyboard/joystick input always takes precedence over mouse to prevent fighting

### Weapon System

**Vulcan** (default)
- Spread increases per level: level 1 = center, level 2 = 2 bullets (±0.12), level 3 = 3 bullets (±0.24)
- Non-piercing; one-shot per enemy

**Laser** (acquired via weapon powerup)
- Faster fire rate (0.1s vs 0.15s)
- Pierces enemies: level 1 = 1 pierce, level 2 = 1 pierce, level 3 = 2 pierces
- Wider bullet footprint; laser aura around player

**Power-Up System**
- Life pickup: Restores one life (capped at max)
- Bomb pickup: Adds one bomb (capped at max)
- Weapon pickup: Advances level 1→2→3, then switches weapon and resets level
- Spawn chance: 6% life, 8% bomb, 20% weapon, 66% none

### Boss Attack Patterns

Each boss has 3 attack variations that rotate. Key pattern types:
- **Radial burst**: 14–26 bullets around boss
- **Aimed spread**: 2–7 bullets toward player with spread angle
- **Columns**: 5 bullets dropped in vertical line
- **Spiral burst**: 8 bullets in rotating pattern
- **Cross fire**: 8 bullets in cardinal + diagonal directions
- **Homing seeded**: Multi-bullet aimed shots with slight randomness
- **Double ring**: Two concentric radial bursts at different speeds
- **Teleport + attack**: Boss repositions horizontally, then fires

### Visual Effects

**Screen Shake**
- Magnitude and duration controlled by game events (boss hit, bomb, player death)
- Reduced to 15% magnitude under reduced-motion preference
- Applied via canvas context translate

**Flash Effects**
- Hit flash: Full white fade over 0.25s
- Bomb flash: Longer full white fade over 0.5s
- Both controlled by CSS animation, skipped on reduced-motion

**Combo/Stage Popups**
- Combo pop: Yellow text scaling 0.5→1.15→1 over 0.6s at screen center
- Stage banner: Large white text, 2s display on stage transition

**Vignette & Pulse**
- Red radial vignette appears when health = 1 (critical state)
- Pulses at 1.1s interval; skipped on reduced-motion

### Score & Progression

**Scoring**
- Kill score × combo multiplier (1–10×, one level per 5 kills)
- Chain timer: 2.2s window to maintain combo; breaks on damage or time out
- Boss kills: Fixed 500 points
- Bomb kills: 5 points per enemy

**Best Score**
- Persisted in localStorage
- Shows on game over screen; highlights if new best

### Accessibility Features

- **Screen Reader Support**: Live region (`#srLive`) announces game state changes (pause, reduced motion, lives, stage)
- **Reduced Motion**: Toggle button honors system preference (`prefers-reduced-motion`); disables shake, flash, animations
- **High Contrast**: Dark theme with text shadows for readability
- **Safe Area Insets**: Respects notch/safe areas on mobile devices

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with: `w, h, hp, speedMul, color, dark, score, minStage`
2. Add sound effect to `sfx` object if needed (e.g., `explodeNewType`)
3. Add mapping to `EXPLODE_SFX` for the right sound
4. Update `spawnEnemy()` probability logic to include new type

### Add a New Boss
1. Add entry to `BOSSES` array with: `name, hullTop, hullBottom, wing, core, hpMul`
2. Add 3-item attack pattern array to `BOSS_PATTERNS`
3. Update `drawBossShip()` if custom visuals needed (uses design colors)
4. Boss index auto-cycles; new boss appears at appropriate stage

### Adjust Difficulty Balance
- Modify `DIFFICULTIES` object: `spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul`
- Spawn rate: `max(spawnMin, spawnBase - difficultyRamp)` where ramp increases every ~5 seconds

### Add a Visual Theme
1. Add object to `THEMES` array: `sky: [top, mid, bottom], cloud: 'r,g,b', sun: 'r,g,b', name`
2. Themes cycle: `THEMES[(stage - 1) % THEMES.length]`

### Test Audio
- Ensure first user interaction (click/tap) fires before sounds work (AudioContext suspend rule)
- Check console for audio context errors
- Mute toggle persisted; test localStorage read/write

## Key Constants & Tweakable Values

- **KILLS_PER_STAGE**: 18 (kills to trigger boss)
- **BASE_W / BASE_H**: 480×720 (logical canvas size)
- **MAX_SCALE**: 1.6× (canvas CSS scaling limit)
- **STORAGE_***: localStorage keys for persistence
- Weapon cooldowns: 0.15s (vulcan), 0.1s (laser)
- Frame cap for delta: 50ms (locked loop uses 16.67ms target)

## File Structure

```
Google_4/
├── index.html       (single monolithic file: all HTML, CSS, JS, and game logic)
├── README.md        (project title/timestamp)
└── CLAUDE.md        (this file)
```

## Notes for Future Work

- **No build tools**: Game runs directly; changes to `index.html` take effect on browser refresh
- **localStorage scope**: Keys prefixed `skyfighter.*` to avoid collisions
- **Mobile-first responsive**: Canvas scales to fit viewport; touch controls auto-enable on touch device detection
- **Quality auto-scaling**: Particles reduced on lower frame rates; shadow blur reduced at quality level > 0
- **Performance budget**: Targets 60fps; tracks last 45 frame times and scales quality down if avg > 25ms
- **Accessibility first**: All interactive elements have keyboard shortcuts; screen reader live regions announce state
