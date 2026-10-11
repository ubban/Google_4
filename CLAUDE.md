# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. No build process, bundler, or external assets are required—the entire game runs standalone in any modern browser.

## Running the Game

- **Direct**: Open `index.html` in a web browser
- **Local Server**: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`
- **No build step required**

## Architecture

### Single-File Design
The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1670 lines):
- **HTML (lines 1-421)**: DOM structure and layout
- **CSS (lines 10-359)**: Styling, animations, responsive design, reduced-motion support
- **JavaScript (lines 422-1666)**: Game engine, logic, and audio synthesis

### Core Game Loop
The main game runs in `requestAnimationFrame(loop)` at ~60 FPS (target 60fps, adaptive quality):
1. **Update phase** (`update(dt)`) — physics, collision, spawning
2. **Render phase** (`draw()`) — canvas drawing with effects
3. **Performance tracking** (`trackPerf(dt)`) — adaptive quality scaling

### Key Game Systems

#### Input Handling (lines 767-861)
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer (passive), click to fire
- **Touch**: Left side for joystick movement, right side for FIRE/BOMB buttons
- Joystick implementation: `joystickStart/Move/End` functions handle multi-touch
- `mouseActive` priority: keyboard/joystick always take priority when active

#### Physics & Collision (lines 1069-1295)
- **Collision**: `rectsOverlap(a, b)` — axis-aligned bounding box collision using center and half-width/height
- **Player movement**: Clamped to canvas bounds with smooth gliding toward mouse target
- **Bullet lifecycle**: Removed when off-canvas
- **Screen shake**: Stored in `shakeTime` and `shakeMag`, applied in `draw()` via context translation

#### Weapon System (lines 1124-1146)
Two weapons with 3 upgrade levels each:
- **Vulcan** (default): Spread increases with level ([-0.24, 0, 0.24] at L3)
- **Laser**: Multiple lanes unlock at higher levels, piercing capability (L3 = 2 pierce)
- Fire rates: 0.15s cooldown (Vulcan) vs 0.1s (Laser)
- Toggle weapons on 3rd weapon pickup; resets level to 1

#### Bomb System (lines 1103-1122)
- **Effect**: Clears all enemy bullets and enemies; damages boss for 24 HP
- **Resources**: 2-3 per game depending on difficulty
- **Acquisition**: Random drop from killed enemies or pickups
- **Cooldown**: None; limited only by max bombs

#### Enemy Spawning (lines 1022-1041)
- **Base spawn rate**: Varies by difficulty (Normal: 58 frames between spawns, ramping down)
- **Enemy types**:
  - Grunt (L1+): Basic, 1 HP, score 10
  - Interceptor (L1+): Fast, erratic movement, score 15
  - Tank (L2+): Slow, 4 HP, heavy score (35), health bar visible
- **Patterns**: Enemies choose between "spread" (fixed angles) or "aimed" (at player) fire

#### Boss Encounters (lines 1043-1056, 1372-1439)
- **Trigger**: After 18 enemy kills per stage
- **Stages 1-6**: Cycle through 6 boss designs with unique attack patterns
- **HP scaling**: Base 70 + (stage × 35), multiplied by difficulty and boss-specific factors (0.8-1.7×)
- **Attack patterns**: Each boss has 3 signature patterns rotated with telegraph warning (22 frames)
- **Patterns include**: radial bursts, aimed spreads, spirals, columns, homing, crosses, double rings

#### Difficulty Settings (lines 485-489)
Three presets affecting spawn rate, enemy speed, shoot chance, lives, bombs, and boss HP:
- **Easy**: 4 lives, 3 bombs, slower enemies (1.0× speed), 0.8× boss HP
- **Normal**: 3 lives, 2 bombs, 1.4× enemy speed, 1.0× boss HP
- **Hard**: 3 lives, 2 bombs, 1.9× enemy speed, 1.3× boss HP
- Settings persist in `localStorage`

#### Scoring & Multipliers (lines 1087-1101)
- **Base score**: Per enemy (10-35) or boss kill (500)
- **Multiplier**: 1 + floor(chain / 5), max 10× at 50+ chain
- **Chain timer**: 2.2s per kill; breaks on taking damage
- **Best score**: Persisted in `localStorage`

#### Audio Synthesis (lines 542-676)
**No external audio files** — all sound effects generated with Web Audio API:
- `beep(freq, dur, type, vol, opts)` — oscillator-based tones with sweeps, filters, panning
- `noiseBurst(dur, vol, opts)` — white noise generator with decay
- Effects include: shoot, laser, explosions (enemy type-specific), hits, bombs, power-ups, boss sounds
- Audio context auto-suspends on mobile; must be resumed on first user interaction

#### Visual Effects
- **Particles** (lines 1073-1085): Explosion clouds with velocity, gravity falloff, max count adaptive to quality
- **Screen shake** (lines 726-730): Magnitude and duration stored; reduced motion mode scales down
- **Flash overlays** (lines 698-709): White flash on hit or bomb detonation
- **Combo pop-ups** (lines 711-717): "X KILLS!" notifications at 10-kill intervals
- **Stage banners** (lines 719-724): Scene transition messages with staggered animations
- **Boss warning** (lines 1055): Red vignette pulse when health is critical

#### Themes (lines 493-500)
Six parallax sky themes, one per stage (wraps on stage 7+):
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each theme defines: sky gradient, cloud color, sun color, and name

#### Adaptive Quality (lines 473-483)
Frametime tracked over 45 frames; quality levels (0 = full, 1 = medium, 2 = low):
- Full: All particle effects, glow shadows, quality rendering
- Medium: 60% particle count, reduced shadow blur
- Low: 35% particle count, no shadows
- Scales automatically based on frame time

#### Accessibility Features
- **Reduced motion**: Respects `prefers-reduced-motion` media query; removes animations
- **Screen reader announcements** (lines 468-471): `srLive` element for game state updates
- **Keyboard controls**: Full support; no mouse required

## Important Constants & Balancing

### Gameplay Values
- `BASE_W = 480, BASE_H = 720` — Canvas resolution (responsive scaling)
- `KILLS_PER_STAGE = 18` — Enemies to defeat before boss encounter
- Enemy cooldown ranges: 30-120 frames between shots

### Camera & Canvas
- Canvas scales to fit window with 1.6× max (line 757)
- Safe area insets respect notches on mobile

### State Management
All game state is in the outer scope:
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` — entity arrays
- `score`, `lives`, `running`, `paused`, `gameOver`, `stage` — game flags
- `chain`, `multiplier`, `frame` — scoring mechanics
- Persisted in `localStorage`: best score, mute state, difficulty, reduced motion preference

## Code Organization

### Functions by Category
- **Game logic**: `update()`, `draw()`, `loop()`
- **Initialization**: `resetGame()`, `spawnEnemy()`, `spawnBoss()`
- **Collision/damage**: `rectsOverlap()`, `addScore()`, `loseLife()`, `endGame()`
- **Rendering**: `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()`
- **Boss AI**: `updateBoss()`, `fireRadial()`, `fireAimedSpread()`, etc.
- **Audio**: `beep()`, `noiseBurst()`, `sfx` object
- **UI updates**: `renderHearts()`, `renderBombs()`, `showComboPop()`
- **Utilities**: `panFor()`, `weaponLabel()`, `togglePause()`

### Event Listeners
- `startBtn`: Start/resume game
- `diffButtons`: Change difficulty (only when not running or paused)
- `muteBtn`, `motionBtn`, `pauseBtn`: Game controls
- `moveZone`, `fireBtn`, `bombBtn`: Touch controls
- `canvas`: Mouse movement and firing
- `window`: Keyboard input, touch/mouse events, resize

## Common Changes

### Balancing Difficulty
Edit `DIFFICULTIES` (line 485) to adjust spawn rates, enemy speed, boss HP. Each difficulty level has 6 parameters.

### Adding a Boss Pattern
1. Add pattern function to `BOSS_PATTERNS` (line 1372) at the appropriate index (0-5 for each boss)
2. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Adding Themes
Push to `THEMES` array (line 493) with `{ sky: [color1, color2, color3], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }`

### Adjusting Weapon Stats
Edit `fireWeapon()` (line 1124): adjust spread angles, pierce values, fire rates, or projectile speeds.

### Tweaking Enemy Spawning
Modify `ENEMY_TYPES` (line 502) for new enemy stats or edit `spawnEnemy()` (line 1022) for spawn logic.

## Testing Notes

- **No automated tests** — the game is designed for manual testing in browsers
- Test across different devices: desktop (mouse + keyboard), tablets (touch), mobile (touch + orientation)
- Use browser DevTools to check console for errors
- Use the `quality` variable in the console to test performance modes: `quality = 0/1/2`
- LocalStorage can be cleared via `localStorage.clear()` to reset saved data

## Performance Considerations

- Canvas rendering is optimized with `ctx.save()`/`restore()` for grouping state changes
- Particle count adapts to frame rate automatically
- Screen shake is computed per-frame but reduced on lower-quality settings
- Audio uses lazy initialization: `ensureAudio()` creates the context on first user interaction
- Touch joystick only renders when active (hidden when not in use)
