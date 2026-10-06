# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Sky Fighter is a browser-based Raiden-style vertical shoot-'em-up game written as a single self-contained HTML file with embedded JavaScript. The entire game runs client-side with no external dependencies — all assets are generated procedurally (vector graphics, synthesized audio).

## How to Run the Game

Simply open `index.html` in any modern web browser. The game is fully playable immediately. The canvas will automatically scale to fit the viewport while maintaining the 16:9 aspect ratio.

## Key Technical Architecture

### Game Loop (requestAnimationFrame)

The core game loop (`loop()` function, starting ~line 1640) runs at 60 FPS and handles:
1. **update(dt)** — Processes game state: player movement, entity updates, collision detection, scoring
2. **draw()** — Renders the current frame using Canvas 2D context

The game state is managed through several tracking variables initialized around line 864:
- `player` — Position, weapon, health, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` — Entity arrays
- `score`, `lives`, `stage`, `chain`, `multiplier` — Progression and scoring metrics
- `running`, `paused`, `gameOver` — Game state flags

### Difficulty System

Three preset difficulties (Easy, Normal, Hard) defined as `DIFFICULTIES` object (~line 485) control enemy behavior, spawn rates, and boss health. Each difficulty preset affects:
- Enemy spawn frequency and speed
- Enemy shoot probability and bullet patterns
- Player lives and bomb count
- Boss health multiplier

Selected difficulty is persisted to localStorage and can be changed from the title screen.

### Boss System

Boss battles trigger after `KILLS_PER_STAGE` (18) enemy kills. Each stage cycles through 6 distinct boss designs (BOSSES array, ~line 508) with different color schemes and attack patterns. Boss patterns are stored in `BOSS_PATTERNS` (~line 1351) as arrays of firing functions that execute during the telegraph phase. Bosses have:
- Health bars (visual and tracked)
- Entering animation (y position interpolation)
- Telegraph phase before each attack (22 frame warning)
- Attack patterns (radial, columns, homing, aimed spreads, etc.)

### Scoring & Multiplier Chain

The score system uses a chain multiplier:
- Destroying enemies increments the chain counter
- Chain breaks if the player takes damage (via `loseLife()`, ~line 1456)
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Each kill adds `baseScore * multiplier` to the score

### Weapon System

Player can switch between two weapons collected as power-ups:
- **Vulcan** — Spread bullets fired rapidly (cooling mechanic with `cooldown` counter)
- **Laser** — Thin blue bullets with increased damage (identified by a circular aura around the player)

Weapon level increases with each power-up pickup (capped at 5), improving firing rate and spread width.

### Rendering & Visual Themes

The draw function (~line 1520) renders:
- **Background** — Gradient sky with sun glow and scrolling clouds (theme-dependent)
- **Entities** — Player, enemies, bullets, powerups, particles, boss
- **HUD** — Score, multiplier, weapon, lives/hearts, bombs (always visible)
- **Boss bar** — Health bar appears when a boss is present

Six visual themes (`THEMES` array, ~line 493) cycle per stage, each with sky gradients and cloud colors. Quality setting dynamically adjusts particle counts and glow effects based on framerate (adaptive performance, ~line 476).

### Input Handling

- **Keyboard** — Arrow keys or WASD to move; Space to fire; X or Shift for bombs; P or Esc to pause
- **Mouse** — Move pointer to steer; click to fire
- **Touch** — Left zone (55% width, 55% height bottom-left) for joystick movement; right zone buttons for FIRE and BOMB

Joystick state is tracked in `joyVec`, `joyActive`, and `joyId` (for multi-touch). Mouse and keyboard take priority over joystick if active.

### Collision Detection

Simple axis-aligned bounding box (AABB) overlap check via `rectsOverlap()` (~line 1069). Used for:
- Player vs enemy bullets (loses life)
- Player vs powerups (collects)
- Bullets vs enemies (applies damage)
- Bullets vs boss (applies damage)

### Audio

WebAudio API synthesizes all sounds in real-time — no external files. The `ensureAudio()` function initializes the AudioContext on first user interaction (required by browser autoplay policy). Sound effects are defined in the `sfx` object (~line 624) and include:
- Weapon fire (different for Vulcan and Laser)
- Bomb detonation
- Enemy/boss explosions
- Boss telegraph warning
- Game over, pause, UI clicks

All sounds are muted if the player clicks the mute button; state persists via localStorage.

### State Persistence

localStorage keys:
- `skyfighter.bestScore` — Best score ever achieved
- `skyfighter.muted` — Audio muted flag
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference (also respects system preference via `prefers-reduced-motion`)

### Accessibility

- **Reduced Motion** — Disables animations (screen shake, flashes, pulses) if enabled
- **Screen Reader Support** — Lives and bomb counts rendered as emoji; important events announced via `announce()` function to the `#srLive` element
- **Responsive Design** — Viewport clamp ensures text and UI scale appropriately on all screen sizes

## Common Development Tasks

### Testing the Game

1. Open `index.html` in a web browser (Chrome, Firefox, Safari, Edge all supported)
2. Play through a full game or specific difficulty to verify gameplay feels smooth
3. Test on mobile/touch devices to ensure joystick and buttons work correctly
4. Open browser DevTools Console to check for any JavaScript errors

### Adding New Enemy Type

1. Add entry to `ENEMY_TYPES` object (~line 502) with sprite dimensions, HP, speed multiplier, color, score value, and minimum stage
2. Update the `spawnEnemy()` function (~line 1022) to include the new type in the selection logic if needed
3. Update explosion SFX in `EXPLODE_SFX` object (~line 678) if using a unique explosion sound
4. Test spawning at the appropriate stage

### Adding New Boss Design

1. Add entry to `BOSSES` array (~line 508) with hull colors, wing color, core color, and HP multiplier
2. Create a new attack pattern array in `BOSS_PATTERNS` (~line 1351) with the same index
3. Each pattern is an array of functions that fire bullets (e.g., `fireRadial()`, `fireColumns()`, `fireAimedSpread()`)
4. Test the boss fight by reaching the appropriate stage

### Adjusting Game Difficulty

Modify the `DIFFICULTIES` object (~line 485) to change spawn rates, enemy speed, shoot chance, lives, bombs, or boss health. Test across all three difficulty presets to ensure balance.

### Tweaking Visual Effects

- Screen shake: Modify `screenShake()` calls or the magnitude passed (`shakeTime`, `shakeMag` around line 727)
- Particle counts: Adjust the `count` parameter in `spawnExplosion()` or the quality scaling formula (~line 1074)
- Enemy/boss colors: Edit the color values in `ENEMY_TYPES`, `BOSSES`, or `THEMES`
- Cloud/background rendering: Adjust `drawClouds()` parameters or the sky gradient in `draw()`

### Adjusting Audio

- Synth parameters in `beep()` and `noiseBurst()` (~line 566 and 597)
- SFX timing (frequency sweeps, durations, volumes)
- Master gain in `ensureAudio()` (~line 556)

## Key Invariants & Gotchas

- **Single Canvas 2D Context** — All rendering goes through the single `ctx`. State is managed with `ctx.save()` / `ctx.restore()` to avoid interference between draw calls
- **Frame-based Timing** — `frame` counter increments every game loop tick and is used for animations and effect timings (e.g., boss telegraph blink, cloud scrolling)
- **Delta Time Clamping** — `dt` is clamped to 0.05 seconds (line 1644) to prevent large jumps on frame drops, which keeps collision detection and movement stable
- **Chain Multiplier Breaks on Damage** — Any player hit calls `breakChain()`, resetting multiplier to 1x. This is intentional to reward careful play
- **Boss Telegraph Delay** — Bosses telegraph attacks for 22 frames (~0.37s) before firing. This is encoded in the `telegraph` field and must be respected when adding new attack patterns
- **Quality Scaling** — The adaptive performance system (`trackPerf()`) automatically degrades visual quality if framerate drops, reducing particle counts and glow effects. Do not assume full-quality rendering on all devices
- **Joystick Priority** — Keyboard and mouse input is ignored when the joystick is active to prevent conflicting input on touch devices
- **AudioContext Lifecycle** — The AudioContext is created on first user interaction and may be suspended; `ensureAudio()` must be called before any sound playback

## Performance Considerations

- Canvas rendering is the main bottleneck. Particle counts scale with quality setting
- Enemy and bullet counts grow unbounded — no culling is performed, so very long games may slow down
- Browser DevTools Profiler can identify hot spots (usually `update()` or `draw()`)

## Browser Compatibility

Works on all modern browsers (Chrome, Firefox, Safari 12+, Edge). Requires:
- ES6 JavaScript support (arrow functions, const/let, template literals)
- Canvas 2D API
- WebAudio API
- localStorage API
- RequestAnimationFrame
- Flexbox CSS

## File Structure

```
/
├── index.html          # Single game file: HTML, CSS, and JavaScript all embedded
└── CLAUDE.md           # This documentation
```

No build step, no dependencies, no external assets. The game is production-ready as-is.
