# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The entire game is implemented in a single `index.html` file with embedded CSS and JavaScript. No build process, bundlers, or external dependencies are required.

### Key Characteristics
- **Canvas Resolution:** 480×720 (base size, scales responsively)
- **Rendering:** Canvas 2D API with particle effects and glow effects
- **Audio:** WebAudio API with synthesized SFX (no external audio files)
- **Storage:** localStorage for high scores, settings, and user preferences
- **Accessibility:** Screen reader support, reduced motion preferences, keyboard/mouse/touch controls

## Running & Testing the Game

### Quick Start
```bash
# No build needed—open in a browser:
open index.html
# Or serve locally for safety:
python3 -m http.server 8000
# Then visit: http://localhost:8000
```

### Controls
- **Desktop:** Arrow keys or WASD to move, Space/click to fire, X/Shift for bombs
- **Mobile:** Drag left zone to move, tap FIRE button, tap BOMB button
- **UI:** P or Esc to pause, mute/motion buttons in top-right corner

### Testing Focus Areas
1. **Gameplay Loops:** Stage progression, boss fights, score multipliers
2. **Difficulty:** Easy/Normal/Hard mode balance and enemy spawn rates
3. **Input:** Keyboard, mouse (steering), joystick (touch), and cross-input priority
4. **Responsiveness:** Canvas scaling on window resize / orientation change
5. **Mobile:** Touch controls, safe area insets, reduced motion preference
6. **Audio:** Synth effects, muting, audio context suspension/resume
7. **Performance:** Adaptive quality scaling based on frame times
8. **Accessibility:** Screen reader announcements, reduced-motion compliance

## Architecture & Code Organization

The JavaScript is organized functionally rather than with classes. Key sections:

### 1. **Global Game State** (lines 863–869)
```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score = 0, lives = 3, running = false, paused = false, gameOver = false;
let chainTimer = 0, multiplier = 1, stage = 1;
```
All game state is mutable and updated each frame in the `update(dt)` loop.

### 2. **Input Handling** (lines 768–861)
- **Keyboard:** Captured via `keydown`/`keyup`; keys object stores current state
- **Mouse:** `mousemove` updates `mouseTarget`; `mousedown` sets `firing`
- **Touch/Joystick:** Virtual joystick on left side for movement, buttons for actions
- **Priority:** Keyboard/joystick input takes priority over mouse steering

### 3. **Game Loop** (lines 1640–1665)
```javascript
requestAnimationFrame(loop);  // Calls update(dt), then draw()
```
Frame counter, dt clamping to 0.05s max, and performance tracking happen here.

### 4. **Update Loop** (lines 1148–1302)
Handles:
- Player movement and weapon firing (with cooldown)
- Enemy spawning and movement
- Boss behavior and attack patterns
- Collision detection (bullets ↔ enemies, enemies ↔ player, etc.)
- Power-up collection
- Particle updates
- Score/multiplier logic

### 5. **Rendering** (lines 1520–1638)
- Canvas backgrounds (sky gradient, sun glow, cloud parallax)
- Player plane drawing (`drawPlane`)
- Boss ships with unique colors and glow
- Bullets, particles, power-ups
- Screen shake applied via canvas translation

### 6. **Audio System** (lines 542–676)
**WebAudio synth approach** — no external files:
- `beep(freq, dur, type, vol, opts)`: Oscillator with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: White noise with envelope
- SFX effects (shoot, laser, explode, bomb, etc.) layer multiple oscillators
- `masterGain` controls master volume; `muted` flag silences all

### 7. **Difficulty Settings** (lines 485–489)
Three tiers: Easy/Normal/Hard. Affect:
- Enemy spawn rate (`spawnBase`, `spawnMin`)
- Enemy speed and shoot chance
- Starting lives and bombs
- Boss HP multiplier

### 8. **Boss System** (lines 508–1409)
- **6 Boss Types:** VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Attack Patterns:** Each boss has 3 attack patterns (radial, aimed spreads, homing, etc.)
- **Pattern Functions:** `fireRadial()`, `fireAimedSpread()`, `fireCross()`, etc.
- **Telegraph Mechanic:** Boss pauses before attack; player can see core glow + hears sound cue

### 9. **Theming** (lines 493–500)
Six sky themes with gradient colors and cloud styles. Cycles by stage.

### 10. **Quality/Performance Scaling** (lines 474–483)
Adaptive rendering: tracks frame times and reduces particle/glow count on slower devices.

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502–506) with `w, h, hp, speedMul, color, dark, score, minStage`
2. Optional: Add weave pattern and custom explosion SFX (see `EXPLODE_SFX` at line 678)
3. Enemy spawning logic reads from this table; no other changes needed

### Adding a New Boss or Attack Pattern
1. **New Boss Design:** Add to `BOSSES` array (lines 508–515) with hull colors and HP multiplier
2. **New Attack Pattern:** Add pattern function to line 1324+ (e.g., `fireNewPattern(b) {...}`)
3. **Assign Patterns:** Add 3-pattern array to `BOSS_PATTERNS` (lines 1372–1409)
4. Boss selection uses `(stage - 1) % BOSSES.length` to cycle

### Tweaking Balance
- **Difficulty:** Edit `DIFFICULTIES` object (lines 485–489)
- **Spawn Rate:** Adjust `spawnBase`/`spawnMin` in difficulty config
- **Weapon Fire Rate:** `fireDelay` at line 1177
- **Enemy Bullet Speed:** Search `enemyBullets.push()` calls (e.g., line 1219, 1223)
- **Boss Attack Speed:** Modify `attackTimer` reset value (line 1429)

### Modifying Input
- **Keyboard Key:** Add to `keys` check (e.g., line 1153–1156) or special handlers (line 772–773)
- **Touch Button:** Edit touch event listeners (lines 829–834) or button styling (lines 310–326)
- **Mouse Steering:** Adjust lerp speed at line 1167 (`maxStep`)

### Changing Canvas Size or Aspect Ratio
- Edit `BASE_W`, `BASE_H` (line 753)
- Adjust max scale at line 757
- May need to tweak enemy spawn positions and boss attack trajectories

### Adding a New Power-Up Type
1. Add logic to `maybeSpawnPowerUp()` (lines 1058–1067)
2. Add case to `applyPowerUp()` (lines 1304–1322)
3. Add rendering code in `draw()` (lines 1617–1626)

## Key Constants & Tuning

| Constant | Location | Purpose |
|----------|----------|---------|
| `BASE_W`, `BASE_H` | 753 | Canvas size |
| `KILLS_PER_STAGE` | 491 | Enemies to defeat before boss appears |
| `DIFFICULTIES` | 485–489 | Difficulty presets |
| `ENEMY_TYPES` | 502–506 | Enemy definitions |
| `BOSSES` | 508–515 | Boss designs |
| `THEMES` | 493–500 | Sky themes |
| `fireDelay` | 1177 | Weapon cooldown (0.15 for vulcan, 0.1 for laser) |

## Storage Keys

Game persists to localStorage under these keys:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference override

## Performance Notes

1. **Quality Levels:** 0 (full, with glow), 1 (medium), 2 (low, no glow). Adapts based on frame time averages.
2. **Particle Limits:** Explosion particle counts scale with quality (lines 1074)
3. **Screen Shake:** Disabled if reduced motion is on (line 728–729)
4. **Touch Optimization:** `touch-action: none` prevents browser-default pan/zoom (line 17)
5. **Audio Context:** Suspended on page load; resumed on first user input (line 560)

## Accessibility Features

- **Screen Reader:** `#srLive` element announces game state changes (announce() function, line 468)
- **Reduced Motion:** Respects system preference; can override via settings button
- **Keyboard Controls:** Full game is playable without mouse
- **Touch Controls:** Mobile-friendly with large hit targets (78px buttons)
- **Color Contrast:** UI text uses text shadows for readability over gradients

## Browser Compatibility Notes

- **Canvas 2D:** Widely supported; uses standard context methods
- **WebAudio API:** Prefix fallback for older Safari (`webkitAudioContext`, line 548)
- **localStorage:** Required for settings persistence
- **prefers-reduced-motion:** Media query respected (line 520)
- **IndexedDB:** Not used; localStorage is sufficient for this game

## Future Enhancement Ideas

- Leaderboard backend integration (currently only local best score)
- Sound volume control slider (currently binary mute)
- Additional boss types or stage themes
- Game replays or demo mode
- Procedural level generation
- Multiplayer (network play would require significant refactor)
