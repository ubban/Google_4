# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single, self-contained HTML file. The entire game—rendering, audio, input handling, and game logic—is implemented in vanilla JavaScript using HTML5 Canvas and the Web Audio API, with no external dependencies or build process.

## Getting Started: Running the Game

Simply open `index.html` in any modern web browser. The game runs entirely client-side with no server required. Use localStorage to persist the best score and settings.

**Quick test**: Open the file in Chrome, Firefox, Safari, or Edge and start playing.

## Game Architecture

The game uses a classic real-time game loop pattern with three main phases:

### Game Loop Structure
- **`loop(now)`** (line ~1640): Main animation frame handler that orchestrates the game tick
  - Calculates delta time (capped at 50ms to prevent spiral of death)
  - Calls `update(dt)` for game logic if running and not paused
  - Calls `draw()` for rendering
  - Continues via requestAnimationFrame

### Core Game Logic (`update(dt)`)
Processes all frame updates:
- Player movement and weapon firing
- Enemy spawning, movement, and shooting patterns
- Projectile lifecycle (player bullets and enemy fire)
- Collision detection (bullets vs enemies, enemies vs player)
- Boss behavior and phase transitions
- Power-up spawning and collection
- Score multiplier chain system

### Rendering (`draw()`)
Renders all visual elements to canvas in order:
- Background with layered cloud parallax scrolling
- HUD elements (score, weapons, health, bombs)
- Player and enemies
- Projectiles and particles
- Boss and boss health bar

## Key Game Systems

### Player Object
```javascript
player = {
  x, y, w, h,                 // Position and dimensions
  hp,                         // Current health (starts at 3)
  weapon,                     // Current weapon type ('vulcan', 'laser', 'spread')
  weaponLevel,               // Weapon upgrade level (1-3)
  spreadAngle,               // Spread weapon angle offset
}
```

### Enemies
Spawned continuously (`spawnEnemy`). Enemy types defined in `ENEMY_TYPES`:
- `fighter`: Fast, weak, basic pattern
- `ranger`: Shoots spread bullets
- `tank`: Heavily armored, fires powerful shots
- `zig`: Erratic movement pattern

Each enemy has `x, y, w, h, hp, color, type, shootTimer`, and a movement/firing pattern.

### Boss System
Each stage spawns one boss after 18 enemy kills. Bosses have:
- Health that scales with difficulty
- Multiple firing patterns (e.g., `fireRadial`, `fireAimedSpread`, `fireColumns`, etc.)
- Phase transitions when health drops below 50%
- Built-in "AI" that rotates through attack patterns

### Weapons System
Three weapons with upgrade levels:
1. **Vulcan** (default): Fast, narrow spread
2. **Laser**: High damage per shot, laser beam effect
3. **Spread**: Wide spread angle, lower individual damage

Weapons are obtained via power-ups and upgrade to level 3 through collection.

### Score Multiplier Chain
- Killing consecutive enemies increases combo (`multiplier`)
- Breaking the chain (letting an enemy escape or dying) resets to 1x
- Multiplier displayed in HUD, affects power-up rewards

### Power-up System
Three types spawned randomly when enemies die:
- `life`: Red square, restores 1 HP
- `bomb`: Yellow square, refills smart bombs
- `upgrade`: Green square, levels up current weapon or switches weapon if already level 3

### Particle Effects
- Explosions (colored particles on enemy death)
- Weapon fire trails
- Screen shake and flash effects
- Vignette pulse on player hit

## Game States and Flow

```
[Title Screen] ──(click Start)──> [Game Running] ──(Stage 1-3 bosses)──> [Game Over]
                                        ↕ (click Pause)
                                   [Paused]
```

- **`running`**: Game actively updating
- **`paused`**: Game updates frozen, UI overlay visible
- **`gameOver`**: Game ended, shows final stats

## Difficulty System

Three difficulty levels affect:
- Enemy bullet speed and frequency
- Enemy HP
- Boss health pools
- Damage multipliers

Configured in `DIFFICULTIES` object. Persisted in localStorage and changeable from the title screen.

## Audio System

All sounds generated programmatically using Web Audio API (no audio files):
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones (sine, square, triangle)
- `noiseBurst(dur, vol, opts)`: White/brown noise for explosions
- Effects have panning based on horizontal position (`panFor(x)`)
- All audio disabled if `muted` flag set (localStorage persisted)

## Mobile / Touch Input

- `#touchControls` elements provide on-screen joystick and fire/bomb buttons
- Joystick (`#moveZone`) drives player movement
- Action buttons trigger fire and bomb directly
- Automatically enabled on touch-capable devices
- Keyboard still works on desktop (WASD/arrows + space/ctrl)

## Performance Considerations

- Performance tracked in `frameTimes` array
- Delta time capped at 50ms to prevent runaway updates
- Canvas quality scaled for reduced-motion preference (fewer shadow/glow effects)
- Particle count limited by frame rate constraints
- localStorage write only on game end (minimized I/O)

## localStorage Keys

- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Mute preference (true/false)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## CSS Structure

The file is structured in blocks:
- `:root` and `*`: Global styles
- `html, body, #container`: Layout foundation
- `#gameCanvas`: Main game rendering
- `#hud`: Heads-up display (score, weapon, hearts)
- `#overlay`: Title and game-over screens
- `#touchControls`: Mobile input overlay
- Animations: `@keyframes` for heartBeat, vignettePulse, flashHit, etc.
- Reduced-motion: CSS media query and class `.reduced-motion`

## Viewport & Responsive Design

- Uses CSS `clamp()` for responsive font sizing
- Viewport meta tag set to prevent zoom and maximize for mobile
- Safe area insets respected for notched devices
- Touch action and user select disabled for game feel

## HTML Structure

Two main DOM subtrees:
1. **Game UI** (`#container`): Canvas, HUD, effects overlays, touch controls
2. **Title/End Screen** (`#overlay`): Card-based modal UI for menus

All interactive elements are in the IIFE; minimal reliance on external scripts.

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `hp`, `w`, `h`, `fireRate`, `color`, `dark` properties
2. Add shooting logic in `spawnEnemy()` or the enemy update loop
3. Assign the type when spawning: `enemies.push({ type: 'newType', ... })`

### Add a New Weapon
1. Create a firing function (e.g., `fireNewWeapon(b)`)
2. Update `weaponLabel()` to return the weapon name
3. Add case in `fireWeapon()` to call your function
4. Update power-up logic to include the new weapon type

### Add a New Boss or Boss Pattern
1. Add to `BOSSES` array with `name`, `hp` (base multiplier), `patterns` (array of function names)
2. Create the pattern function (e.g., `fireHelixSeeded(b, speed)`)
3. Boss selection in `spawnBoss()` pulls from this array by stage

### Tune Difficulty Scaling
Edit `DIFFICULTIES` object to adjust enemy/boss health, bullet speed, and scoring multipliers per difficulty level.

### Add Visual Effects
- Use `screenShake(magnitude, duration)` for collision feedback
- Use `flashHit()` or `flashBomb()` for impact flashes
- Use `spawnExplosion(x, y, color, count)` for particle bursts
- Adjust animation `@keyframes` in CSS for vignette, heart beat, etc.

## Key Constants and Magic Numbers

- **Canvas size**: Responsive, scaled to fit window (see `resize()`)
- **Player speed**: `player.speed = 180` (pixels per second)
- **Enemies per stage**: `KILLS_PER_STAGE = 18` before boss spawns
- **Max combo multiplier**: Uncapped (scales with consecutive kills)
- **Frame rate cap**: Delta time capped at 0.05s (50ms)
- **Sound notes**: Middle C ≈ 261.63 Hz; A4 = 440 Hz

## Testing & Validation

Since this is a client-side game with no build step:
- Open in browser and play through each difficulty
- Test touch controls on a mobile device or browser DevTools
- Check that localStorage persists best score across sessions
- Verify audio plays (check browser dev console for Web Audio errors)
- Test pause, resume, and game-over transitions

No automated tests present; manual play-testing is the validation method.

## Notes for Future Work

- Entire game logic is in one IIFE (~1700 lines)
- Could benefit from modularizing into classes (Player, Enemy, Boss, Game states)
- Audio synthesis is simple; could add more sophisticated sound design
- Boss patterns currently rotate through a fixed array; dynamic or procedural patterns could add variety
- Leaderboard could be added (use IndexedDB or backend API)
- Replay system or recording could be implemented
