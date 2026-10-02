# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a self-contained HTML5 application with Canvas rendering and procedurally generated sound (Web Audio API). The entire game is in a single `index.html` file with no external dependencies, build process, or assets required.

## Running & Testing

- **Play the game**: Open `index.html` directly in a browser or use the `/run` skill
- **No build process**: The game runs immediately as a static file
- **Responsive design**: Automatically scales to window size; supports desktop (keyboard/mouse) and mobile (touch) controls
- **Testing gameplay**: Difficulty can be toggled in-game (Easy/Normal/Hard); best score is persisted in localStorage

## Architecture

### Core Game Loop
The game uses `requestAnimationFrame` for rendering and a delta-time-based update system (`update(dt)` and `draw()`). Game state flows through:
1. Input handling (keyboard, mouse, touch joystick)
2. `update(dt)` — advance all game entities (player, enemies, bullets, particles, boss)
3. Collision detection (bullets vs enemies/boss, enemies/bullets/boss vs player, power-ups vs player)
4. `draw()` — render everything to canvas with current theme sky gradient, clouds, all entities

### Entity System
Game objects are plain JS objects with properties:
- **Player**: position, weapon (vulcan/laser), level, health, invulnerability timer, cooldown
- **Enemies**: spawned in waves, move downward with weaving patterns or aimed shots
- **Bullets**: player bullets move up and pierce/penetrate based on weapon; enemy bullets move arbitrarily
- **Particles**: simple animated debris from explosions, lifecycle-based
- **Power-ups**: weapon upgrades, life, bomb drops from enemies
- **Boss**: complex multi-pattern attack AI with telegraphing, health bar, stage-based design/difficulty

### Difficulty & Progression
- **DIFFICULTIES** object defines spawn rates, enemy speed/accuracy, player lives/bombs, boss HP multipliers for easy/normal/hard
- **KILLS_PER_STAGE** (18 kills) triggers boss spawn
- **Stages** increment after each boss; themes and boss designs cycle through THEMES and BOSSES arrays
- **Score multiplier**: chain counter (resets if player takes damage) increases with consecutive kills

### Boss System
- **6 unique boss designs** (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) each with distinct visual styling and attack pattern sets
- **BOSS_PATTERNS**: Each boss has 3 attack patterns that cycle; patterns use helper functions (fireRadial, fireAimedSpread, fireColumns, etc.) to generate enemy bullet patterns
- **Telegraph mechanic**: Boss flashes and plays audio cue before each attack; gives player brief reaction window
- **Bomb damage**: Smart bombs deal fixed 24 damage to boss and clear all enemy bullets

### Audio System
- **No external assets**: All sound effects generated procedurally via Web Audio API (beep + noiseBurst functions)
- **beep()**: Oscillator with optional frequency sweep, filter, and panning
- **noiseBurst()**: White noise filtered through a high-pass or low-pass filter
- **sfx object**: Named effect functions (shoot, laser, explode variants, hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, uiClick, gameover) each compose beep/noiseBurst calls
- **Muted state**: Toggled via mute button, persisted in localStorage; audio context suspended until user interaction (per browser autoplay policy)

### Rendering & Visual Polish
- **Theme system**: 6 sky gradients + cloud/sun colors rotate by stage; drawClouds() scrolls parallax clouds
- **Adaptive quality**: Performance tracker monitors frame times and drops quality (full → medium → low particle/shadow counts) under load
- **Screen shake**: Magnitude and duration configurable; reduced by 85% if reduced motion is enabled
- **Flash effects**: White flash on hit, larger on bomb; suppressed in reduced-motion mode
- **Animations**: All CSS-based (heartBeat, comboPulse, stagePulse, cardIn, iconBob, etc.) with reduced-motion preference support
- **Glow/shadow**: High-quality mode (quality=0) adds drop shadows to bullets and boss core

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Position determines target; click to fire; mouse moves take priority if active
- **Touch**: Left zone (55% of screen width) is joystick movement, right zone has FIRE/BOMB buttons; joystick base appears on touch, follows with clamped magnitude
- **Priority**: Keyboard/joystick input suppresses mouse movement when active

### State Persistence
- `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Loaded on page load; updated on user actions (score, mute toggle, difficulty change, motion preference toggle)

## Key Variables & Constants

- **BASE_W / BASE_H** (480×720): Canvas resolution; scales to fit window up to 1.6× zoom
- **ENEMY_TYPES**: Grunt (1 HP, fast), Interceptor (1 HP, very fast, weaves), Tank (4 HP, slow, armored)
- **BOSS_PATTERNS**: 2D array of attack pattern functions, one set per boss type; patterns use angle-based bullet generation
- **chain / multiplier**: Kill streak counter and score multiplier (caps at 10×); resets when player takes damage
- **quality**: Adaptive rendering quality (0=full, 1=medium, 2=low) based on average frame time

## Development Notes

- **Single file**: All game logic, rendering, and audio live in one 1669-line HTML file. No module system or build step.
- **Immutable themes**: THEMES, BOSSES, ENEMY_TYPES, DIFFICULTIES, BOSS_PATTERNS are config-like constants—modify these to tweak game balance/visuals without touching core logic.
- **Canvas coordinates**: Origin top-left; positive Y downward. Player starts near bottom; enemies spawn at top and move down.
- **Collision**: Simple AABB (axis-aligned bounding box) via rectsOverlap(); used for all hit detection.
- **Drawing order**: Sky → sun glow → clouds (2 layers) → player → bullets → enemies (with health bars for tanks) → boss → power-ups → particles. Shake offset applied globally before drawing.
- **Accessibility**: Screen reader live region (srLive) announces game events; reduced-motion preference respected; semantic HTML buttons with titles.
- **Performance**: Frame time tracking disables effects on slow devices; particle count scaled by quality; shadows/glow omitted on low-quality mode.

## Modifying Gameplay

**Balance tweaks**:
- Adjust DIFFICULTIES object spawn rates, speeds, shot chances, lives/bombs, boss HP
- Modify KILLS_PER_STAGE to change when boss appears
- Edit ENEMY_TYPES colors, HP, speed multipliers
- Tweak boss pattern functions (fireRadial, fireAimedSpread, etc.) to change attack difficulty

**Visual changes**:
- Modify THEMES sky gradients, cloud/sun RGB values, theme names
- Adjust player/enemy draw functions (drawPlane, drawBossShip) for different ship designs
- Change particle colors in addScore, spawnExplosion, killBoss calls

**Audio changes**:
- Modify sfx functions to adjust frequencies, durations, filter parameters
- masterGain.gain.value controls overall volume

**Controls**:
- Modify key codes in keydown/keyup listeners (ArrowLeft, KeyA, Space, KeyX, etc.)
- Adjust player.speed (movement) or fireDelay (weapon cooldown)
- Tweak joystick/touch zone sizes and button positions
