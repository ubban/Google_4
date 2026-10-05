# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game engine, UI, sound synthesis, and graphics are contained in `index.html`. It runs entirely in modern web browsers without any build process or external dependencies.

## Quick Start

### Running the Game
Simply open `index.html` in a web browser. The game loads immediately and is playable on desktop (mouse/keyboard), tablet, and mobile (touch controls).

```bash
# Open in browser (macOS)
open index.html

# Or from terminal
python3 -m http.server 8000  # then visit http://localhost:8000
```

### Testing Changes
1. Edit `index.html` in any text editor
2. Refresh the browser (Cmd+R / Ctrl+R)
3. Changes take effect immediately — no build step needed

## Architecture & Code Structure

The entire game logic is contained within a single **IIFE** (Immediately Invoked Function Expression) to avoid polluting the global namespace. Code flows in this order:

### 1. **DOM & Canvas Setup** (lines 426–461)
   - References all DOM elements (canvas, UI buttons, HUD elements)
   - Sets up the canvas context for 2D rendering

### 2. **Constants & Configuration** (lines 463–515)
   - `DIFFICULTIES`: Easy/Normal/Hard difficulty presets (spawn rates, enemy behavior, lives, boss HP multipliers)
   - `KILLS_PER_STAGE`: Enemies needed to trigger boss (18)
   - `THEMES`: 6 visual themes (sky gradients, cloud colors, sun colors)
   - `ENEMY_TYPES`: Grunt, Interceptor, Tank with HP, speed, color, score values
   - `BOSSES`: 6 unique boss designs with stat multipliers
   - `BOSS_PATTERNS`: Attack pattern functions for each boss type

### 3. **Persistent State** (lines 517–527)
   - `localStorage` keys for best score, mute state, difficulty, reduced motion preference
   - Loads and initializes stored settings on startup

### 4. **Audio Subsystem** (lines 542–676)
   - **No external audio files** — all sound is procedurally generated using Web Audio API
   - `ensureAudio()`: Lazily initializes `AudioContext` (required by browser autoplay policy)
   - `beep()`: Creates pitched oscillator tones with optional frequency sweep and filtering
   - `noiseBurst()`: Creates white noise with envelope and filtering
   - `sfx` object: Named sound effects (shoot, laser, hit, bomb, boss patterns, etc.)
   - Panning is calculated based on pixel position (`panFor()`) for spatial audio

### 5. **Input Handling** (lines 767–862)
   - **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - **Mouse**: Move to steer, click to fire; position tracked in `mouseTarget`
   - **Touch**: Left half of screen for joystick movement, right buttons for Fire/Bomb
   - **Joystick logic**: Virtual stick appears on touch; calculates input vector (`joyVec`) clamped to radius

### 6. **Game State** (lines 863–869)
   - Arrays: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Scalars: `score`, `lives`, `stage`, `multiplier`, `chain`, `bombCount`
   - Flags: `running`, `paused`, `gameOver`

### 7. **Game Loop** (lines 1640–1665)
   - `requestAnimationFrame(loop)` runs at 60 FPS
   - Adaptive delta-time clamped to 0.05s (minimum 20 FPS safety)
   - Calls `update(dt)` for logic, `draw()` for rendering
   - Performance tracking adjusts visual quality if frame times degrade

### 8. **Update Function** (lines 1148–1302)
   - **Player movement**: Reads input, updates position, clamps to screen bounds
   - **Firing**: Weapon cooldown system, calls `fireWeapon()` for Vulcan/Laser spread
   - **Enemy spawning**: Difficulty-based spawn timer, ramps up over time
   - **Boss trigger**: After 18 kills, spawns boss after 1.2s delay
   - **Collision detection**: Enemy→player, bullets→enemies, bullets→boss, powerups→player
   - **Score chain**: Consecutive kills boost multiplier; timer breaks chain if no kill for 2.2s
   - **Particle effects**: Explosions decay and are removed when life expires

### 9. **Rendering Pipeline** (lines 1520–1637)
   - Sky gradient + sun glow + parallax clouds (2 layers, different speeds)
   - Player plane (blue) with invulnerability blink
   - Laser aura when laser weapon active
   - All bullets with glow when quality is high
   - Enemies as plane shapes with HP bars (tanks only)
   - Boss ship with animated engines and telegraph glow
   - Power-ups as rotating squares
   - Particles as fading circles
   - Screen shake (random translation) applied via canvas `ctx.translate()`

### 10. **Boss System** (lines 508–1409)
   - Each boss has unique attack patterns stored in `BOSS_PATTERNS` array
   - Patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, etc.
   - Boss telegraph phase (white glow + sound) gives player 22 frames (~367ms) warning before attack
   - Boss moves side-to-side; bounces off screen edges
   - Destroyed boss triggers stage advance, score bonus (+500), and stage banner

### 11. **Quality Adaptation** (lines 474–483)
   - Tracks last 45 frame times, calculates average
   - If avg > 25ms (~40 FPS), increase quality level (1 = medium, 2 = low)
   - If avg < ~17.86ms (~56 FPS), decrease quality level
   - Affects particle count in explosions and glow effects (shadow blur on canvas)

## Key Game Mechanics

### Weapons & Power-ups
- **Vulcan** (default): Straight shots, wider spread at level 3 (3 bullets)
- **Laser**: Fast piercing shots, split into multiple lanes at level 2+
- Weapon power-ups cycle: Level 1→2→3 per pickup, then switch weapon type
- Fire rate: Vulcan 150ms (6.7 shots/sec), Laser 100ms (10 shots/sec)

### Combo Multiplier
- Multiplier = 1 + floor(chain / 5), capped at 10× (50 consecutive kills)
- Chain resets on death or after 2.2s without a kill
- Displayed in HUD and increases score per kill

### Bombs
- Limited use (2 normal, 3 easy)
- Clears all enemy bullets + all on-screen enemies
- Damages boss (24 HP)
- Cannot be replenished except via power-ups

### Difficulty Modifiers
| Stat | Easy | Normal | Hard |
|------|------|--------|------|
| Spawn Base | 75 | 58 | 40 |
| Spawn Min | 40 | 26 | 18 |
| Enemy Speed | 1.0× | 1.4× | 1.9× |
| Shoot Chance | 45% | 70% | 95% |
| Lives | 4 | 3 | 3 |
| Bombs | 3 | 2 | 2 |
| Boss HP | 0.8× | 1.0× | 1.3× |

### Accessibility Features
- **Reduced Motion Toggle** (button in top-right): Disables screen shake, flash effects, animations
- **Screen Reader Support**: All state changes announce via `aria-live` region
- Respects system `prefers-reduced-motion` media query on first load

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add explosion sound to `EXPLODE_SFX` object (line 678) if desired
3. Update `spawnEnemy()` filter (line 1026) to include new type

### Adding a New Boss Pattern
1. Add boss to `BOSSES` array (line 508–515) with name, hull colors, core color, `hpMul`
2. Create attack pattern functions using helpers like `fireRadial()`, `fireAimedSpread()`, etc.
3. Add array of 3 pattern functions to `BOSS_PATTERNS` in matching index (line 1372)
4. Boss is automatically selected based on stage: `BOSSES[(stage - 1) % BOSSES.length]`

### Changing Difficulty Presets
Edit `DIFFICULTIES` object (line 485–489). Changes affect all new games with that difficulty selected.

### Adding a New Visual Theme
1. Add theme object to `THEMES` array (line 493–500) with `sky` (3-stop gradient), `cloud` (RGB string), `sun` (RGB string), `name`
2. Themes cycle: `THEMES[(stage - 1) % THEMES.length]`
3. Sky gradient applied in `draw()` (line 1530–1535), sun glow (1537–1541), clouds (1543–1544)

### Tweaking Audio
- Adjust master gain: Line 556 (`masterGain.gain.value`)
- Modify sound effect pitch/duration/envelope in the `sfx` object
- Use `beep()` for tonal sounds, `noiseBurst()` for percussive effects
- Filter frequency and type are passed in options object

### Adjusting Visual Effects
- **Screen shake magnitude/duration**: Calls to `screenShake(mag, dur)` (line 727)
- **Particle count**: Multiplier in `spawnExplosion()` (line 1074)
- **Glow effects**: Controlled by `quality` variable and `ctx.shadowBlur` (line 1002, 1583, 1594)

## Performance Considerations

- **No external assets**: Everything is procedurally generated (no image/audio files)
- **Adaptive quality**: Automatically reduces particle effects and glow on slower devices
- **Canvas scaling**: Rendered at 480×720 internal resolution, scaled to fit window (max 1.6× upscaling)
- **Collision checks**: O(n·m) per frame for bullet↔enemy, quadratic for small object counts but acceptable for game scale
- **Particle pruning**: Dead particles removed each frame to prevent memory leak

## Browser Requirements

- Modern browser with:
  - Canvas 2D API
  - Web Audio API (for sound — gracefully degrades if unavailable)
  - `localStorage` for settings persistence
  - Touch Events API (mobile support)
  - `requestAnimationFrame` support

Tested on: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+

## Common Edits

**Increase boss difficulty:**
- Raise `bossHpMul` in `DIFFICULTIES` (line 485–489)
- Or modify base HP calculation in `spawnBoss()` (line 1046)

**Slower enemy fire rate:**
- Decrease `d.enemyShootChance` in difficulties, or adjust shooter timer ranges in `enemies` update (line 1225)

**More/fewer stages before repeating bosses/themes:**
- Bosses repeat via `(stage - 1) % BOSSES.length`; themes similar
- Increase number of unique entries in respective arrays to extend cycle

**Adjust weapon fire rates:**
- `fireDelay` in `update()` (line 1177): lower = faster

**Change splash screen text:**
- Edit overlay text elements (line 403–417) in HTML

**Adjust initial player position:**
- `player.y` in `resetGame()` (line 877)

## Testing Checklist

- [ ] Game starts on "Start Game" click
- [ ] Keyboard input (arrows, WASD, space, X/shift) works
- [ ] Mouse aim circles appear and player follows cursor when stationary
- [ ] Touch controls on mobile: movement zones work, FIRE/BOMB buttons respond
- [ ] Enemies spawn and animate, follow waypoints or weave pattern
- [ ] Bullets collide with enemies, showing explosions
- [ ] Weapons cycle through levels and switch types at level 3
- [ ] Combo multiplier displays correctly and resets on death
- [ ] Boss appears after 18 kills, shows health bar, unique attack patterns
- [ ] Screen shake and flash effects trigger on hit/bomb (disable reduced motion to see)
- [ ] Sound effects play (or muted button toggles correctly)
- [ ] Pause (P/Esc) suspends game and shows overlay
- [ ] Difficulty selection changes spawn rates and boss HP
- [ ] Game Over overlay shows final score, stage reached, best score
- [ ] Best score persists after page reload
- [ ] Reduced motion toggle disables shake/flash (no animation on button click)
