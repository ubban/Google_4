# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5, CSS3, and vanilla JavaScript. It runs in a single `index.html` file with no external dependencies.

### Key Features
- **Vertical scrolling shooter** gameplay with progressive difficulty
- **Multiple enemy types** with different behavior patterns and health
- **Boss battles** with procedurally varied attack patterns
- **Weapon upgrades** system with two weapon types (Vulcan and Laser)
- **Progressive stages** with thematic visual changes
- **Difficulty levels**: Easy, Normal, Hard
- **Touch support** with virtual joystick and buttons for mobile
- **Accessibility**: Reduced motion preferences, screen reader announcements
- **Web Audio synthesis** for all sound effects (no external audio files)

---

## Getting Started

### Running the Game
1. Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. No build step or dependencies required
3. Game state and preferences are stored in browser localStorage

### Browser Compatibility
- Requires modern Canvas API support
- WebAudio API for sound (graceful degradation if unavailable)
- Touch Events for mobile support
- ES6+ JavaScript support

---

## Codebase Structure

The entire game is contained in `index.html` with three main sections:

### 1. **HTML Markup** (Lines 361-420)
- Game canvas container
- HUD elements (score, best score, multiplier, weapon level, lives, bombs)
- Boss health bar
- Visual effect layers (vignette, flash, combo pop, stage banner)
- Touch control zones (joystick, fire button, bomb button)
- Game overlay (start menu, pause, game over screens)
- Accessibility elements (screen reader announcements)

### 2. **CSS Styling** (Lines 10-359)
- Dark theme color scheme (#05122b base)
- Responsive canvas scaling using `clamp()`
- HUD and UI styling with glassmorphism effects
- Animations for hit feedback, screen shake, combo display, boss appearance
- Reduced motion media query support
- Touch-friendly button sizing

### 3. **JavaScript Game Engine** (Lines 422-1667)

#### Core Game Loop
- `loop()` - RequestAnimationFrame-based game loop
- `update(dt)` - Physics and game logic (60 FPS target)
- `draw()` - Canvas rendering
- `trackPerf(dt)` - Adaptive quality adjustment

#### Game State Management
- `player` - Player aircraft object
- `enemies` - Array of active enemy objects
- `boss` - Current boss object (null when not active)
- `bullets`, `enemyBullets` - Player and enemy projectiles
- `powerUps`, `particles` - Visual effect objects
- Game variables: `score`, `lives`, `stage`, `chain`, `multiplier`

#### Player Controller
- **Keyboard**: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Left zone for joystick movement, right buttons for fire/bomb
- Input is stored in `keys` object for immediate response

#### Enemy System
```javascript
ENEMY_TYPES = {
  grunt:       { hp: 1, score: 10, minStage: 1 },
  interceptor: { hp: 1, score: 15, minStage: 1, weave: true },
  tank:        { hp: 4, score: 35, minStage: 2 }
}
```
- Enemies spawn based on difficulty timer
- Pattern selection: "spread" (radial) or "aimed" (player-targeting)
- `spawnTimer` reduces based on frame count for increasing difficulty

#### Weapon System
- **Vulcan**: Fast-firing bullets with spread pattern
  - Level 1: Single center shot
  - Level 2: Dual shots
  - Level 3: Triple spread shots
- **Laser**: Slower, piercing shots with AOE glow
  - Level 1: Single shot
  - Level 2: Dual shots
  - Level 3: Triple shots + higher pierce count

#### Boss System
```javascript
BOSSES = [
  { name: 'VANGUARD', hpMul: 1.0, ... },
  { name: 'BEHEMOTH', hpMul: 1.35, ... },
  { name: 'SPECTER', hpMul: 0.8, ... },
  // ... more boss types
]
```
- 6 unique boss designs with different visual and behavior characteristics
- `BOSS_PATTERNS` array contains 3 attack patterns per boss type
- Boss telegraph mechanic: visual warning before attack with sound
- Bosses enter from top, patrol left-right, and fire multiple attack patterns

#### Difficulty System
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, enemySpeed: 1.0, enemyShootChance: 0.45, ... },
  normal: { spawnBase: 58, enemySpeed: 1.4, enemyShootChance: 0.7, ... },
  hard:   { spawnBase: 40, enemySpeed: 1.9, enemyShootChance: 0.95, ... }
}
```
- Controls enemy spawn rate, speed, accuracy, and boss health multiplier
- Stored in localStorage and persists between sessions

#### Scoring & Progression
- **Chain System**: Kill multiplier increases with consecutive kills (resets after 2.2s)
  - Multiplier = 1 + Math.floor(chain / 5), max 10x
- **Stage Progression**: 18 kills per stage triggers boss battle
- **Best Score**: Saved to localStorage

#### Visual Effects
- **Screen Shake**: Magnitude and duration for hits/bombs
- **Color Themes**: 6 different sky/cloud themes that cycle per stage
- **Particles**: Explosion effect particles with velocity and fade
- **Vignette**: Red pulsing edge effect when at 1 health
- **Flash**: White/bright flash on hit or bomb detonation

#### Audio System
- WebAudio synthesis (no external files)
- `beep()` - Generates square/sine/sawtooth tones with envelope
- `noiseBurst()` - Generates white noise bursts
- Sound effects for: shoot, laser, explosions, hit, bomb, powerup, boss telegraph, boss down, stage clear, ui, gameover
- Stereo panning based on X position for spatial audio
- Master gain control with compression

#### Input Handling
- **Mouse**: Movement fires ray-cast calculations for steering
- **Touch**: Multi-touch support with gesture tracking
- **Joystick**: Virtual analog stick on left side for movement
- **Keys**: Raw key state tracking with event listeners

---

## Key Balance Parameters

### Enemy Spawn Tuning
- **spawnBase** (35-75): Base spawn delay in frames
- **spawnMin** (18-40): Minimum spawn delay as difficulty ramps
- Frame-based ramp: `spawnBase - Math.floor(frame / 300) * 4`

### Weapon Balance
- **Vulcan**: Fast (150ms cooldown), wide spray, lower damage per shot
- **Laser**: Slow (100ms cooldown), piercing, high damage per shot
- Weapon switching at max level changes weapon back to level 1

### Boss Health Calculation
```javascript
baseHp = 70 + stage * 35  // Increases per stage
hp = baseHp * difficultyMul * bossHpMul
```

### Combo & Scoring
- Kill any enemy: +10-35 points × multiplier
- Kill from bomb: +5 points × multiplier
- Kill boss: +500 points × multiplier

---

## Adding New Features

### Adding an Enemy Type
1. Add entry to `ENEMY_TYPES` with hp, color, score, minStage
2. Update `spawnEnemy()` to include in choices
3. Add explosion SFX to `EXPLODE_SFX` mapping if needed
4. Adjust spawn probability in `spawnEnemy()` selection logic

### Adding a Boss
1. Add object to `BOSSES` array with name, colors, hpMul
2. Create attack pattern function array in `BOSS_PATTERNS`
3. Pattern functions use: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireDoubleRing()`, etc.
4. Boss automatically uses pattern based on position in BOSSES array

### Adding a Stage Theme
1. Add object to `THEMES` array with sky gradient, cloud color, sun color, name
2. Themes cycle automatically: `THEMES[(stage - 1) % THEMES.length]`

### Adding a Power-up Type
1. Update spawn probability in `maybeSpawnPowerUp()`
2. Add case to `applyPowerUp()` for effect
3. Add color to rendering logic in `draw()` powerup section
4. Add SFX if needed

### Modifying Difficulty
- Edit `DIFFICULTIES` object parameters
- Test impact on enemy/boss spawn timing and challenge
- Consider separate balance for each difficulty

---

## Performance Optimization

### Adaptive Quality
- Tracks frame times over 45 frames
- Automatically adjusts quality level (0=full, 1=medium, 2=low)
- Affects:
  - Particle count reduction
  - Shadow/glow effects
  - Effect filters and animations

### Rendering Optimizations
- Uses canvas 2D context with context saves/restores
- Batch drawing by type (enemies, bullets, powerups)
- Reduced motion preference respects system accessibility settings
- Canvas size: 480×720 (base), scaled to fit window

### Input Optimization
- Keyboard state tracked in object for O(1) lookup
- Mouse steering uses simple interpolation (no per-frame calculations)
- Touch events debounced with joystick tracking

---

## Browser LocalStorage Keys

- `skyfighter.bestScore` - Best score achieved
- `skyfighter.muted` - Audio mute state (0/1)
- `skyfighter.difficulty` - Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` - Reduced motion preference (0/1)

---

## Accessibility Features

- **Keyboard navigation**: All controls accessible via keyboard
- **Reduced motion**: System preference detection + manual toggle
- **Screen reader support**: `#srLive` element for announcements
- **High contrast**: Dark UI with bright accent colors
- **Touch-friendly**: Large button targets (78px diameter)
- **Responsive**: Scales to any device size using fluid typography

---

## Debugging Tips

### Enable Console Logging
- Add `console.log()` statements in update/draw loops
- Watch `player`, `enemies`, `boss` objects
- Monitor `frame` counter for timing issues

### Visual Debugging
- Use `ctx.strokeStyle/lineWidth` to draw collision boxes
- Render `spawnTimer` and `stageKills` on canvas
- Check quality level with `console.log(quality)`

### Performance Profiling
- Chrome DevTools Performance tab
- Look for long frame times > 16.7ms
- Check `trackPerf()` output in console

### State Inspection
- `console.log(player)` - Check position, weapon, health
- `console.log(enemies.length)` - Count active enemies
- `localStorage` - View saved game settings

---

## Future Enhancement Ideas

- [ ] High score leaderboard system
- [ ] Particle effects improvements (trails, impact effects)
- [ ] Additional enemy behaviors (charging, spiral patterns)
- [ ] Power-up shop/selection screen
- [ ] Persistent upgrades across runs
- [ ] Multiplayer scoreboard
- [ ] Custom difficulty settings
- [ ] Replay/spectate recorded games
- [ ] Boss-specific unlocks/achievements
- [ ] Procedural enemy patterns
- [ ] Screen effect post-processing (chromatic aberration, etc.)

---

## Code Style & Conventions

- Vanilla JavaScript (no frameworks)
- Immediate Function Expression (IIFE) for scope
- camelCase for variables and functions
- UPPERCASE for constants
- Comments only for non-obvious logic
- Canvas context state managed with save/restore
- Time values in seconds (dt parameter)

---

## Last Updated
Created by Claude Code for documentation and future maintenance.
