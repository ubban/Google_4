# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. All code (HTML, CSS, JavaScript) is contained in `index.html`. The game runs entirely in the browser with no build process, external dependencies, or test framework.

## Running the Game

- **In browser**: Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge)
- **Local server** (optional): `python3 -m http.server 8000` then visit `http://localhost:8000`
- **Touch**: Fully playable on mobile devices with on-screen joystick and buttons

## Code Architecture

The entire game is an IIFE (Immediately Invoked Function Expression) in a `<script>` tag. Key sections:

### 1. **Game State** (~line 864)
- `player`: position, speed, weapon, level, health, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: game entities
- `boss`: optional boss battle state
- `score`, `lives`, `multiplier`, `chain`: scoring system state
- `stage`, `running`, `paused`, `gameOver`: game lifecycle

### 2. **Audio Synthesis** (~line 543)
- Uses Web Audio API (no external audio files)
- `beep()`: tone synthesis with frequency sweeps and filters
- `noiseBurst()`: procedural noise generation (explosions, etc.)
- `sfx` object: ~15 sound effects (shoot, explode, hit, bomb, boss telegraph, etc.)
- Spatial panning based on entity X position

### 3. **Input Handling** (~line 767)
- **Keyboard**: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire (on desktop)
- **Touch**: Left side is movement zone (joystick), right side has FIRE and BOMB buttons
- Priority: keyboard/joystick > mouse movement (no fighting between input methods)

### 4. **Game Loop** (~line 1640)
- `requestAnimationFrame(loop)` drives the main loop
- `update(dt)`: input handling, collision detection, entity movement
- `draw()`: canvas rendering with theme-based visuals
- Performance scaling: tracks frame time and reduces particle count on low-end devices

### 5. **Enemy & Boss Systems**
- **Enemy types** (~line 502): grunt, interceptor, tank (each with HP, speed, score multiplier)
- **Enemy spawning** (~line 1022): randomized based on stage and difficulty
- **Enemy bullets**: pattern-based (spread or aimed at player)
- **Bosses** (~line 1043): 6 boss designs, stage-based progression, complex attack patterns
- **Boss patterns** (~line 1372): 6 different pattern sets (radial bursts, aimed spreads, spirals, homing, etc.)

### 6. **Weapons & Power-ups**
- **Vulcan gun**: spread shot (1→2→3 projectiles at level 1→2→3)
- **Laser**: straight beam with pierce through enemies (1→2→3 lanes at levels 1→2→3)
- **Power-ups**: weapon upgrade, life restore, bomb recharge (6%, 8%, 20% spawn rates)

### 7. **Rendering & Visuals** (~line 1520)
- 6 themed sky gradients (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Procedural cloud layers (two speeds)
- Screen shake on hit/bomb (reducible in motion settings)
- Flash effects for hits/bombs
- Particle explosions with per-frame alpha fade

### 8. **UI & Settings**
- HUD: score, best score, multiplier, weapon info, hearts, bombs
- Settings: difficulty (Easy/Normal/Hard), mute toggle, reduced motion toggle
- Pause overlay with resume/difficulty change
- Game Over screen with score, stage, best score tracking
- Accessibility: screen reader announcements, reduced motion CSS class

## Difficulty Settings

Located in `DIFFICULTIES` (~line 485):
- **Easy**: slower spawns, weaker enemies, 4 lives
- **Normal**: balanced difficulty, 3 lives
- **Hard**: dense enemy waves, strong bosses, 3 lives

Each difficulty scales: enemy spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier.

## Key Formulas & Constants

- **Base canvas**: 480×720 (scaled responsively, max 1.6× on large screens)
- **Stage progression**: 18 kills per stage, then boss battle
- **Scoring**: `base_score * multiplier` where multiplier increases with kill chains (1 → 1.2 → 1.4 ... → 1.9 every 5 kills)
- **Boss HP**: `(70 + stage × 35) × difficulty_mul × boss_design_mul`
- **Chain reset**: 2.2 second timeout between kills (breaks multiplier if too slow)
- **Invulnerability**: 1.5 seconds after taking damage

## Extending the Game

### Add a New Boss
1. Add to `BOSSES` array (~line 508): name, hull colors, core color, HP multiplier
2. Add attack patterns to `BOSS_PATTERNS` array (~line 1372): array of 3 pattern functions
3. Pattern functions receive the boss object and fire enemy bullets via helper functions (see `fireRadial`, `fireAimedSpread`, etc.)

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` object (~line 502): width, height, HP, speed multiplier, color, score
2. Optionally add to `spawnEnemy()` logic (~line 1022) to control spawn probability and stage availability
3. Create/add to `EXPLODE_SFX` lookup if custom explosion sound needed

### Add a New Power-up Type
1. Add to `maybeSpawnPowerUp()` spawn logic (~line 1058)
2. Add handling in `applyPowerUp()` (~line 1304)
3. Add rendering in `draw()` power-up section (~line 1617) with appropriate color/rotation

### Modify Weapon Behavior
1. Vulcan gun: adjust `fireWeapon()` spread angles and speed (~line 1126)
2. Laser gun: adjust `fireWeapon()` lane offsets and pierce count (~line 1136)
3. Adjust `fireDelay` for weapon cooldown (~line 1177)

### Add a Sound Effect
1. Write a function in the `sfx` object (~line 624) using `beep()`, `noiseBurst()`, and envelope functions
2. Call with `sfx.yourSound()` at appropriate game event

### Add a Theme
1. Add to `THEMES` array (~line 493): sky gradient colors, cloud color, sun color, name
2. Rendered automatically based on `(stage - 1) % THEMES.length` in `draw()`

## Performance Considerations

- **Adaptive quality** (~line 473): Tracks frame times and reduces particle count under load
- **Collision detection** (~line 1069): Simple AABB (axis-aligned bounding box) via `rectsOverlap()`
- **Canvas scaling**: CSS-based scaling (canvas is always 480×720 internally, sized via CSS)
- **Mobile optimization**: Touch event debouncing, CSS `touch-action: none`, reduced motion support

## Browser Support

- Modern browsers (Chrome, Firefox, Safari 14+, Edge)
- Requires: Canvas API, Web Audio API, localStorage, requestAnimationFrame
- Mobile: iOS Safari 14+, Android Chrome
- Audio works only after user gesture (click/touch) due to browser autoplay policies

## Debugging Tips

- Open DevTools Console to see any errors
- Check `localStorage` values: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Add `console.log()` in the main `update()` or `loop()` function to monitor state
- Use `canvas.width/height` to understand internal coordinate system (always 480×720)
- Boss behavior is determined by stage: `(stage - 1) % BOSSES.length` selects the boss design
