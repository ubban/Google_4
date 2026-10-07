# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a self-contained vertical shoot-'em-up game (Raiden-style) built as a single HTML5 file with embedded CSS and JavaScript. No external dependencies or build process needed.

- **File Structure**: Everything in `index.html`
- **Canvas Size**: 480×720px (responsive scaling to fit viewport)
- **No Build Required**: Run by opening `index.html` in a browser

## How to Run

1. **Open in Browser**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local Server** (optional for strict CORS): `python -m http.server 8000` then visit `http://localhost:8000/index.html`
3. The game starts with the title screen overlay

## Game Architecture

### Core Game Loop
- **Animation Frame**: RequestAnimationFrame drives the main loop at ~60 FPS
- **Update Phase** (`update(dt)`): Input handling, entity movement, collision detection, scoring
- **Render Phase** (`draw()`): Canvas clear, background layers, entity drawing with effects
- **Delta Time**: Clamped to 50ms max to prevent large timestep jumps

### Key Game Systems

#### Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move cursor to steer player (only when not using keyboard/joystick), click to fire
- **Touch**: Virtual joystick on left (movement), fire/bomb buttons on right
- **Priority**: Keyboard/joystick input takes priority over mouse when active

#### Entity System
- **Player**: Single player plane at bottom, moves within canvas bounds, has invulnerability frames after being hit
- **Enemies**: Spawned continuously, 3 types (grunt, interceptor, tank) with difficulty-scaling HP/speed
- **Bullets**: Player bullets (upward), enemy bullets (downward), pierce/laser mechanics for certain weapons
- **Boss**: Spawns after 18 kills per stage, has complex attack patterns with telegraph phase
- **Particles**: Explosion effects, short-lived with physics
- **Power-ups**: Dropped from destroyed enemies (weapon upgrades, bombs, lives)

#### Collision Detection
- Simple AABB (axis-aligned bounding box) via `rectsOverlap()` function
- Enemy bullets clear on bomb usage; player invulnerable for 1.5s after hit

#### Audio (WebAudio Synthesis)
- No external sound files—all SFX are synthesized via WebAudio API
- `beep()`: Oscillator with optional frequency sweep, filter, panning
- `noiseBurst()`: Noise source with lowpass/highpass filtering
- Master gain + compressor for dynamic range
- Audio context is lazy-initialized on first interaction (user gesture required)

### Difficulty System
Three difficulties (Easy/Normal/Hard) with parameters stored in `DIFFICULTIES` object:
- `spawnBase`, `spawnMin`: Enemy spawn rate (decreases over time)
- `enemySpeed`, `enemyShootChance`: Enemy behavior
- `lives`, `bombs`: Starting count
- `bossHpMul`: Boss health multiplier

### Progression
- **Stages**: Increase every boss kill; each stage has a visual theme and boss design
- **Kills per Stage**: 18 kills triggers boss spawn (after 1.2s warning)
- **Score Multiplier**: Chain kills to build multiplier (resets after 2.2s gap)
- **Themes**: 6 rotating sky themes with parallax clouds

### Boss System
- **6 Boss Designs**: Each with unique art colors, HP multiplier, and 3-attack pattern sequences
- **Attack Patterns**: Radial bursts, aimed spreads, columns, spirals, homing projectiles, teleport-dash attacks
- **Telegraph Phase**: 22-frame warning (animated core glow + SFX) before attack fires
- **Entering Animation**: Boss slides down 120px on spawn

### Adaptive Performance
- **Quality Levels**: 0 (full—with shadows/glow), 1 (medium—reduced particles), 2 (low—minimal effects)
- **Auto-scaling**: Tracks average frame time; upgrades quality if avg < 1/56s, downgrades if > 1/40s
- Window size and quality affect particle counts, shadow blur, and visual polish

### Accessibility Features
- **Reduced Motion**: Toggle via button; disables animations, screen shake, and flash effects
- **Screen Reader Support**: Live region updates for game state changes
- **Color Contrast**: High-contrast HUD elements on translucent dark backgrounds
- **Mobile Safe Area**: Respects `env(safe-area-inset-*)` for notches/rounded corners

## Code Organization (within index.html)

### HTML (Lines 1–421)
- Canvas element for rendering
- HUD overlays (score, health, bombs, weapon info, boss health bar)
- UI elements (buttons, pause/mute/motion toggles)
- Touch control zones (joystick, fire/bomb buttons)
- Overlay for menu/pause/game-over screens

### CSS (Lines 10–359)
- Dark theme with neon accents
- Responsive sizing via `clamp()` for mobile scaling
- Animations: heart beat, combo popup, stage banner, card in
- Touch-friendly button sizing (78px circles)
- Glassmorphism UI (backdrop blur, semi-transparent backgrounds)

### JavaScript (Lines 422–1666)
**Constants & Configuration**
- `DIFFICULTIES`: Spawn rates and difficulty multipliers
- `THEMES`: 6 sky gradients with cloud/sun colors
- `ENEMY_TYPES`: Grunt, interceptor, tank properties
- `BOSSES`: 6 boss designs with color palettes and HP multipliers
- `BOSS_PATTERNS`: 3-pattern attack sequences per boss (18 attack patterns total)

**State Variables** (Line 864–869)
- Game state: `running`, `paused`, `gameOver`, `stage`, `score`, `lives`
- Entity arrays: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Scoring: `chain`, `multiplier`, `kills`, `stageKills`

**Core Functions**
- `update(dt)`: Game logic (movement, spawning, collisions, damage)
- `draw()`: Rendering (background, entities, effects)
- `fireWeapon()`: Bullet creation (vulcan vs laser with pierce/spread)
- `useBomb()`: Clear enemies and bullets, screen shake
- `updateBoss(dt)`: Boss AI (movement, telegraph, pattern firing)
- `applyPowerUp(type)`: Weapon/bomb/life upgrades
- `spawnEnemy()`: Create enemy with random type/pattern
- `spawnBoss()`: Create boss at stage progression
- Collision helpers: `rectsOverlap()`, damage/kill functions

**Audio Functions**
- `beep(freq, dur, type, vol, opts)`: Tone synthesis with sweep/filter/pan
- `noiseBurst(dur, vol, opts)`: Noise with filter
- `sfx.*`: Named sound effects (shoot, laser, explode, bomb, hit, etc.)

**Input Handling** (Lines 768–861)
- Keyboard event listeners (arrow keys, space, X, P, Esc)
- Touch listeners for joystick and buttons
- Mouse movement/click for steering and firing

**UI Functions**
- `resetGame()`: Initialize all state for new game
- `endGame()`: Show game-over overlay and update best score
- `togglePause()`: Show/hide pause overlay
- `setDifficulty(name)`: Update difficulty and local storage
- `updateMuteBtn()`, `updateMotionBtn()`: Toggle button states

**Rendering Functions**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Render player/enemy ships with optional thrust flame
- `drawBossShip(b)`: Boss rendering with color gradients, core glow, wings, and damage indicator
- `drawClouds(speed, fillColor, rx, ry)`: Parallax cloud layers
- `screenShake(mag, dur)`: Apply camera shake (respects reduced-motion preference)
- Flash, combo pop, stage banner: DOM-driven animations with CSS

## Extending the Game

### Adding a New Boss
1. Add entry to `BOSSES` array with `name`, hull colors, core color, and `hpMul`
2. Add 3-attack pattern array to `BOSS_PATTERNS` (use existing pattern functions or new ones)
3. New boss will cycle in automatically based on stage number

### Adding an Enemy Type
1. Define in `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optionally add `weave: true` for sine-wave movement
3. Update `spawnEnemy()` weight logic if needed
4. Add explosion SFX to `EXPLODE_SFX` mapping if using custom sound

### Adding a Weapon
1. Create two-weapon system by modifying `applyPowerUp()` weapon swap logic
2. Add firing pattern in `fireWeapon()` function
3. Update `weaponLabel()` and weapon stat display
4. Add new SFX to `sfx` object if needed

### Tweaking Difficulty
Edit `DIFFICULTIES` object directly:
- `spawnBase` / `spawnMin`: Tighter range = harder spawn rate
- `enemySpeed`: Multiplier on base enemy velocity
- `enemyShootChance`: 0–1 probability per enemy per frame
- `bossHpMul`: Direct HP scaling for boss

### Adding a New Theme
1. Add entry to `THEMES` array: `{ sky: ['color1', 'color2', 'color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }`
2. Sky array is three stops for the vertical gradient; cloud/sun are CSS RGB strings
3. Themes cycle per stage automatically

## Local Storage Keys
- `skyfighter.bestScore`: High score (persists across sessions)
- `skyfighter.muted`: Audio mute state (1 = muted)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility preference (1 = reduced motion on)

## Performance Considerations
- **Canvas Clearing**: Full clear every frame (no dirty-rect optimization needed at 480×720)
- **Particle Count**: Scales with quality level (full 14 particles per explosion → 8 at medium → 5 at low)
- **Enemy Spawning**: Gradually increases as game progresses (frame-based timer decreases)
- **Boss Patterns**: Staggered firing to avoid 20+ bullets per frame on low-end devices
- **Shadow/Glow Effects**: Only in high quality; disabled in reduced quality mode

## Testing the Game
- **Keyboard Only**: Arrow keys + Space/Shift works fine
- **Touch**: Tap FIRE (red), BOMB (yellow), drag left side to move
- **Mouse**: Move cursor, click to fire; drag left side has no effect on mouse
- **Difficulty Toggle**: Change difficulty from main menu (affects next game start)
- **Reduced Motion**: Toggle via button; verify animations disable
- **Pause**: Press P or click pause button mid-game
- **Boss Patterns**: Each stage cycles through different boss designs + patterns
- **High Scores**: Should persist after closing and reopening browser

## Known Quirks
- Audio context requires user gesture (first click/tap) to initialize
- Touch joystick base only shows when moving; position is visual-only (not part of collision bounds)
- Screen shake is reduced by 85% in reduced-motion mode (small shake still visible for feedback)
- Boss telegraph delay is fixed at 22 frames regardless of framerate (intentional for consistency)
