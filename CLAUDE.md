# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. A single `index.html` file contains the entire game: HTML structure, CSS styling, and JavaScript game engine with Web Audio synthesis. The game features difficulty levels, boss encounters, weapon upgrades, power-ups, and a score multiplier system.

## Running the Game

- **Local browser**: Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge).
- **Web server**: Serve the file via HTTP/HTTPS (avoids CORS issues if deploying).
- No build step, no external dependencies, no setup required.

## Game Architecture

The game is a single 1,667-line IIFE (Immediately Invoked Function Expression) in vanilla JavaScript. Key architectural sections:

### 1. **State Management** (lines 863–870)
Core game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain` (combo counter), `multiplier`.

### 2. **Difficulty & Content** (lines 485–515)
- `DIFFICULTIES`: three presets (easy/normal/hard) controlling spawn rates, enemy speed, enemy accuracy, lives, bombs, and boss HP scaling.
- `THEMES`: six visual themes with gradient sky, cloud, and sun colors. Cycles per stage.
- `ENEMY_TYPES`: grunt, interceptor, tank with varying HP, speed, color, and score values.
- `BOSSES`: six named bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) each with unique hull/core colors and HP multiplier.
- `BOSS_PATTERNS` (lines 1372–1409): per-boss attack patterns using helper functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).

### 3. **Input Handling** (lines 768–861)
- Keyboard: arrow keys / WASD for movement, Space to fire, X/Shift to bomb, P/Esc to pause.
- Mouse: aim by moving pointer over canvas, click to fire.
- Touch (mobile): left zone is movement joystick, right zone has FIRE and BOMB buttons.
- Audio context is resumed on first input via `ensureAudio()`.

### 4. **Game Loop & Update** (lines 1640–1653)
- `requestAnimationFrame(loop)` calls `update(dt)` and `draw()` each frame.
- `dt` is delta time clamped to 50ms per frame to prevent spiral on slow frames.
- `trackPerf()` maintains a 45-frame rolling average of frame times and adjusts `quality` (0=full, 1=medium, 2=low) to reduce particles and glow effects when FPS drops.

### 5. **Update Logic** (lines 1148–1302)
- **Movement**: keyboard/joystick input takes priority; mouse is fallback for steering.
- **Firing**: cooldown-based weapon fire (`fireWeapon()` at line 1124) with spread patterns for Vulcan and pierce mechanics for Laser.
- **Enemies**: spawn timer decreases; enemies move down, weave, and shoot based on pattern (spread vs. aimed). Boss spawns after 18 kills per stage.
- **Collisions**: AABB (axis-aligned bounding box) overlap checks against bullets, enemies, enemy bullets, boss, and power-ups.
- **Chain/Multiplier**: each enemy kill increments chain counter; resets after 2.2 seconds of no kills. Multiplier = 1 + floor(chain/5), max 10×.

### 6. **Rendering** (lines 1520–1638)
- Sky gradient with parallax clouds and sun glow per theme.
- Player ship drawn via `drawPlane()` with optional flame when firing.
- Bullets: laser (blue) or Vulcan (yellow), rendered with optional glow at high quality.
- Enemies: same plane shape as player, tank enemies show HP bar.
- Boss: `drawBossShip()` draws hull, wings, and pulsing core; color scheme unique per boss.
- Particles: explosion debris with life fading.
- HUD: score, best score, multiplier, weapon name/level, hearts (lives), bombs, and boss HP bar.

### 7. **Audio** (lines 542–676)
- All sound is synthesized in real-time via Web Audio API (no audio files).
- `beep()` creates oscillator-based tones with optional frequency sweep, filter, and panning.
- `noiseBurst()` creates noise-based sounds (explosions, bombs).
- `sfx` object contains named sounds: shoot, laser, explode/explodeGrunt/explodeInterceptor/explodeTank, hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, uiClick, gameover.
- Panning is spatial based on object X position to create stereo localization.
- `masterGain` controls global volume; compressor prevents clipping.

### 8. **Boss Behavior** (lines 1411–1454)
- Boss enters from top, moves side-to-side within bounds, pauses between attacks.
- Each boss has a unique pattern set; patterns fire in sequence.
- Telegraph phase (22 frames) warns player before attack; telegraph sound + visual core pulse.
- Boss stays at Y=120 after entering.

### 9. **Storage** (lines 463–527)
- LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
- Difficulty and reduced-motion settings persist between sessions.

## Modifying the Game

**Adding a new enemy type**: Add to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave: true`.

**Adding a new boss**: Append to `BOSSES` array and add a corresponding pattern array to `BOSS_PATTERNS`.

**Tuning difficulty**: Edit `DIFFICULTIES` object or adjust spawning logic in `update()` around line 1197–1203.

**Adding a visual theme**: Append to `THEMES` array with `sky` gradient stops, `cloud` and `sun` RGB values, and a `name`.

**Adjusting sounds**: Edit functions in `sfx` object; `beep()` parameters are `(freq, duration, type, volume, options)` and `noiseBurst()` takes `(duration, volume, options)`.

**Canvas size**: Base resolution is 480×720 pixels (line 753). Scaling happens in `resize()` based on window and device pixel ratio.

## Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari 14+, Edge 79+).
- Requires: Canvas 2D, Web Audio API, localStorage, requestAnimationFrame.
- Touch support detected via `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.
- Respects `prefers-reduced-motion` media query for accessibility.

## Testing & Iteration

- No test suite; validate changes by playing through stages in the browser (especially boss encounters).
- Performance: open DevTools (F12) → Performance tab → record a game session to profile frame times and spot bottlenecks.
- Difficulty balance: test all three difficulty levels to ensure boss fights and enemy spawns feel fair.
