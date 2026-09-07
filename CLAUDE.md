# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The entire codebase is contained in a single `index.html` file with no build process, dependencies, or external assets—all sounds are synthesized with the WebAudio API.

## Running the Game

1. **Local Browser**: Open `index.html` directly in any modern web browser
2. **Local Server** (optional, for development): 
   ```bash
   python3 -m http.server 8000
   # or: npx http-server
   ```
   Then navigate to `http://localhost:8000`

No build process, compilation, or tests—the game runs immediately.

## Architecture

The game follows a classic game loop pattern with no external frameworks:

### Core Game Loop (line 1640–1653)
- **`loop(now)`**: Main frame callback using `requestAnimationFrame`
  - Calculates delta time (dt)
  - Calls `update(dt)` for game logic
  - Calls `draw()` to render
  - Tracks performance and adapts rendering quality
- **`update(dt)`**: Game state updates (collisions, enemies, bullets, scoring)
- **`draw()`**: Canvas rendering (backgrounds, sprites, particles)

### Game State (line 863–869)
Central mutable state includes:
- `player`: Position, health, weapon level, cooldown timers
- `enemies`, `bullets`, `enemyBullets`, `powerUps`, `boss`: Entity arrays
- `score`, `lives`, `stage`, `chain`, `multiplier`: Game metrics
- `running`, `paused`, `gameOver`: State flags

### Entity System
All entities (player, enemies, bullets, particles) are plain JavaScript objects with properties like `x`, `y`, `w` (width), `h` (height). No classes or inheritance—functions operate directly on these objects.

**Key entity operations:**
- **Spawning** (`spawnEnemy`, `spawnBoss`): Create and push to arrays
- **Collision** (`rectsOverlap`): AABB check for rectangles
- **Removal**: Filter arrays to remove out-of-bounds or destroyed entities

### Audio System (line 542–676)
Pure WebAudio synthesis with no external files:
- **`beep(freq, dur, type, vol, opts)`**: Oscillator-based tones with envelope, sweep, filtering, panning
- **`noiseBurst(dur, vol, opts)`**: White/brown noise for explosions and impacts
- **`sfx` object**: Pre-composed sound effects for shooting, explosions, power-ups, boss actions

Audio is muted by default on first load (needs user interaction to enable via `ensureAudio()`).

### Input Handling (line 767–861)
Supports keyboard, mouse, and touch:
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P to pause
- **Mouse**: Move to steer, click to fire; player glides toward cursor
- **Touch**: Left side for joystick movement, right side for FIRE/BOMB buttons
- Input is stored in `keys` object and `firing`/`joyVec` flags; update loop reads these

### Difficulty System (line 485–489)
Three presets (easy/normal/hard) with tunable parameters:
- Spawn rates and minimum spawn interval
- Enemy speed and shoot chance
- Player lives and bombs
- Boss health multiplier
Saved to localStorage and selected via UI buttons.

### Scoring & Combo (line 1087–1101)
- **`addScore(base)`**: Awards points multiplied by current multiplier (1–10x based on kill chain)
- **Chain timer** resets when you go 2.2 seconds without killing anything
- Multiplier increases every 5 consecutive kills

### Stages & Progression (line 491, 493–500)
- **Stages**: Increase difficulty, spawn tankier enemies, change visual theme
- **Themes**: 6 procedural sky gradients + cloud/sun colors (no sprite assets)
- **Boss progression**: 6 unique boss designs that cycle; each has unique attack patterns
- **KILLS_PER_STAGE**: 18 kills triggers boss warning and spawn after 1.2s delay

### Boss AI & Attack Patterns (line 1372–1409)
Bosses telegraph attacks (red glow + sound) before firing. Each boss has 3 distinct attack patterns that cycle:
- **Radial bursts**: Bullets in full rings
- **Aimed spreads**: Fan-shaped patterns tracking player position
- **Homing/teleport**: Bosses move or projectiles track player
- **Dual rings**: Multiple concentric circles

Pattern selection is deterministic based on stage, then cycles through the pattern array.

### Adaptive Performance (line 473–483)
Automatically adjusts rendering quality based on frame time:
- **Quality 0** (full): Glow effects, shadows, more particles
- **Quality 1** (medium): Reduced glow, fewer particles
- **Quality 2** (low): No shadows, minimal effects
Tracks last 45 frames; adjusts when average frame time drifts above/below target.

## Key Sections for Modification

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object (line 502–506) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` (line 1022–1041) to select new type
3. Add sound effect to `sfx` object if needed

### Adding New Boss Patterns
1. Add design to `BOSSES` array (line 508–515): name, colors, hp multiplier
2. Add 3 attack patterns to `BOSS_PATTERNS` array (line 1372+): functions that call `fireRadial`, `fireAimedSpread`, etc.
3. Pattern functions receive boss object; call helper functions or push to `enemyBullets` directly

### Tuning Difficulty
Edit `DIFFICULTIES` object (line 485–489) or adjust multipliers in spawn/boss logic (line 1202, 1046).

### Sound Design
Oscillators and filters are tuned in each `sfx` function (line 624–676). Change frequencies (Hz), duration (seconds), waveform type, sweep targets, filter cutoff, and panning to alter sounds.

### Visual Effects
- **Screen shake**: Call `screenShake(magnitude, duration)` (line 727–730)
- **Flash effect**: Triggered by `flashHit()` or `flashBomb()` (line 698–709)
- **Vignette pulse**: Activates when player health is critical (line 687)
- **Particle explosions**: `spawnExplosion(x, y, color, count)` (line 1073–1085)
- **Canvas quality adjustments**: Controlled by `quality` variable; check `if (glow)` blocks in `draw()`

### UI & Accessibility
- HUD stats updated via DOM elements (lines 367–389 HTML, various update calls)
- Screen reader announcements via `announce()` (line 468–471)
- Reduced motion preference detected and respected (line 520–523, 529–540)

## Storage & Persistence

LocalStorage keys (line 463–466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Animation preference

Loaded on startup and restored when game restarts.

## Canvas Scaling

Base resolution is 480×720 (line 753). Canvas is scaled up to fit the viewport with `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)` scaling cap, maintaining aspect ratio on all devices. Resize handler ensures responsive behavior (line 764–765).

## Browser Compatibility

- **Required**: ES6+ (arrow functions, const/let, template literals)
- **Required**: HTML5 Canvas 2D Context, WebAudio API
- **Optional but recommended**: Touch Events API for mobile, Media Queries for reduced motion

Game degrades gracefully if WebAudio is unavailable (no sound, but gameplay continues).

## Development Notes

- **No external dependencies**: Pure vanilla JavaScript
- **No build step**: Edit the file and reload the browser
- **Single-threaded**: Everything in one script block to keep HTML lean
- **Performance-conscious**: Particle counts and glow effects scale based on frame time; entities are pooled (arrays reused, old items removed via `filter`)
- **Accessibility**: Reduced motion respect, screen reader support, keyboard/mouse/touch parity
