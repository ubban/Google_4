# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript, HTML, and CSS. The entire game is self-contained in a single `index.html` file with embedded styles and logic. No external dependencies or build process required.

## Running the Game

The game runs directly in a web browser. Simply open `index.html` in any modern browser that supports:
- Canvas 2D API
- Web Audio API
- ES6+ JavaScript
- localStorage API

**Development server** (to handle CORS and live-reload):
```bash
# Using Python 3
python3 -m http.server 8000

# Using Node.js/npx
npx http-server

# Using other tools
php -S localhost:8000
```

Then navigate to `http://localhost:8000` in your browser.

## Architecture Overview

The codebase is organized into logical sections within a single IIFE (Immediately Invoked Function Expression) to avoid global scope pollution:

### 1. **Game Loop & Canvas Setup**
- Canvas initialization and responsive sizing (`BASE_W: 480px`, `BASE_H: 720px`)
- Frame-based update/draw cycle using `requestAnimationFrame`
- Delta-time based physics (`dt`) for consistent movement across frame rates
- Adaptive performance tracking to reduce visual quality on low-end devices

### 2. **Input Handling**
Three input methods work together without conflict (keyboard has priority):
- **Keyboard**: Arrow keys + WASD for movement, Space/Click to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Cursor position steers the player; click fires
- **Touch**: Left half of screen is movement joystick, right half has Fire/Bomb buttons

The `joyVec` object tracks normalized joystick input; `firing` boolean controls continuous fire.

### 3. **Game State**
Core mutable state defined in the `resetGame()` function:
- **Player**: position, dimensions, weapon (vulcan/laser), level, cooldown, invulnerability timer
- **Entities**: arrays for bullets, enemies, enemy bullets, particles, power-ups, and one active boss
- **Scoring**: current score, multiplier (chain-based), kill count, stage progression
- **Lives & Bombs**: current count, max count

All state flows through the `update(dt)` function each frame.

### 4. **Audio System**
Procedural audio using Web Audio API (no external files):
- `beep()`: oscillator-based tones with frequency sweep and filters
- `noiseBurst()`: filtered white noise for impact sounds
- `sfx` object: pre-defined sound effects for all game events (shoot, explode, bomb, boss hit, etc.)
- Master gain and compressor for volume control
- Stereo panning based on entity position (`panFor()`)
- Respects user mute preference stored in localStorage

### 5. **Rendering**
Canvas drawing happens in `draw()`:
- **Background**: gradient sky, sun glow, procedurally-animated clouds
- **Entities**: planes (player/enemies), bullets, boss, particles, power-ups
- **UI**: HUD displays score, best score, multiplier, weapon, hearts, bombs
- **Effects**: screen shake, white flash on hit/bomb, vignette red tint when critical health
- **Quality scaling**: particles and glow effects reduce at lower quality levels (detected via frame time monitoring)

### 6. **Game Systems**

#### Enemy System
- **Types**: Grunt (basic), Interceptor (fast, weaving), Tank (durable)
- **Spawning**: difficulty-based spawn timer with ramp-up over time
- **AI**: two shoot patterns (spread fan or aimed at player)
- **Progression**: new enemy types unlock at higher stages

#### Weapon System
- **Vulcan** (default): spread increases at levels 2-3
- **Laser**: faster fire rate, pierce through enemies, narrower spread
- **Power-ups**: weapon pickups increment level or toggle weapon type and reset level to 1

#### Boss System
- **Phases**: Entering (travel to center) → Active (attacks with varying patterns)
- **Attacks**: Each of 6 bosses has 3 distinct attack patterns (radial, aimed, teleport, columns, spirals, etc.)
- **Progression**: Boss HP scales with stage number and difficulty multiplier
- **Design**: Unique visual color schemes and core effects for each boss

#### Combo/Multiplier System
- `chain`: incremented on each kill
- `chainTimer`: resets to 2.2s after each kill (breaks if timer expires)
- `multiplier`: 1-10x, increases by 1 for every 5 kills in the chain
- Breaking chain resets multiplier to 1x

#### Difficulty
Three preset difficulties with parameters:
- Enemy spawn rates, speed, shoot chance
- Starting lives and bombs
- Boss HP multiplier
- Stored in localStorage as user preference

#### Particle Effects
Explosion particles spawn with:
- Random velocity in all directions
- Color-coded by source (enemy, boss, player hit, bomb)
- Life timer for fade-out; count scales with performance quality level

### 7. **Storage (localStorage Keys)**
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state (0/1)
- `skyfighter.difficulty`: current difficulty selection
- `skyfighter.reducedMotion`: animation reduction for accessibility (0/1)

## Key Constants & Configuration

### Canvas & Display
```javascript
BASE_W = 480, BASE_H = 720  // internal resolution
```

### Gameplay
```javascript
KILLS_PER_STAGE = 18  // kills needed before boss spawns
DIFFICULTIES = { easy, normal, hard }  // difficulty parameters
```

### Content
```javascript
THEMES[6]  // sky gradient, cloud color, sun color, display name for each stage
ENEMY_TYPES = { grunt, interceptor, tank }  // enemy definitions with stats
BOSSES[6]  // boss designs with color schemes and HP multiplier
BOSS_PATTERNS[6]  // 3 attack patterns per boss (pattern functions)
```

### Physics & Timing
```javascript
player.speed = 260  // pixels/second
fireDelay = 0.15s (vulcan), 0.1s (laser)
chainTimer = 2.2s  // time before chain breaks
```

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w, h, hp, speedMul, color, dark, score, minStage, weave`
2. Add spawning logic in `spawnEnemy()` if needed (probability roll based on stage)
3. Add explosion effect in `EXPLODE_SFX` mapping if unique sound is needed
4. Update enemy rendering in `draw()` if special visuals required

### Adding a New Boss Attack Pattern
1. Define the pattern function in `BOSS_PATTERNS` (e.g., `fireRadial()`, `fireColumns()`)
2. Call appropriate bullet-spawning helpers from the pattern function
3. Boss will cycle through its 3 patterns automatically on attack

### Tweaking Difficulty
Edit the `DIFFICULTIES` object entries. Common adjustments:
- `spawnBase/spawnMin`: lower = more frequent spawning
- `enemySpeed`: multiplier on enemy speed
- `enemyShootChance`: 0-1 probability enemy can shoot
- `lives/bombs`: starting counts
- `bossHpMul`: multiplier on base boss HP

### Adding a New Visual Theme
1. Add entry to `THEMES[]` array with sky gradient colors, cloud color, sun color, and name
2. Game cycles themes automatically per stage: `THEMES[(stage - 1) % THEMES.length]`

### Screen Effects
- **Screen shake**: call `screenShake(magnitude, duration)`
- **Flash**: call `flashHit()` or `flashBomb()`
- **Red vignette**: automatically triggered when player has 1 life
- **Combo popup**: call `showComboPop(text)`

### Accessibility
The game respects `prefers-reduced-motion` media query. When toggled:
- Animations are removed via CSS (`animation: none`)
- Screen shake is reduced to 15% intensity
- Shake timer is capped at 0.1s

## Performance Considerations

### Quality Levels
The `quality` variable (0–2) is tracked based on average frame time:
- **Quality 0** (full): all glow effects, max particles
- **Quality 1** (medium): reduced particles (60%), reduced glow
- **Quality 2** (low): minimal particles (35%), no glow

Particles, shadows, and glow are conditionally rendered based on quality level.

### Canvas Sizing
The canvas scales responsively via CSS transforms (not resampling). The actual canvas is always 480×720; the display size adapts to window size with `max-width` and `max-height` constraints.

## Testing Gameplay Features

Since there's no test suite, manual testing is the primary approach:

### Difficulty Testing
Change the difficulty in the start menu and verify:
- Enemy spawn rate and speed
- Enemy bullet patterns
- Boss HP
- Starting lives and bombs

### Weapon Testing
- Collect weapon power-ups to test level progression
- Test Vulcan spread at levels 1, 2, 3
- Test Laser pierce behavior and fire rate
- Verify weapon swap at level 3 power-up

### Boss Testing
- Reach stage thresholds to spawn each boss (Stage 1–6 spawn different bosses)
- Verify each boss cycles through its 3 attack patterns
- Test bomb damage on boss (24 HP per bomb)
- Verify boss defeat triggers stage clear and next stage spawn

### Combo Testing
- Kill enemies quickly and watch multiplier increase
- Let timer expire and verify chain breaks (multiplier resets)
- Check score updates correctly with multiplier

### Audio Testing
- Mute button toggle and localStorage persistence
- Verify sound effects pan left/right based on screen position
- Test boss telegraph and hit sounds

### Accessibility Testing
- Toggle reduced motion and verify animations stop
- Test with `prefers-reduced-motion: reduce` OS setting
- Verify touch controls on mobile viewport

## Code Style Notes

- **Self-contained**: All game logic is in one IIFE to avoid polluting global scope
- **No dependencies**: Uses only native Web APIs (Canvas, WebAudio, localStorage)
- **Procedural rendering**: All graphics are drawn via canvas; no spritesheets or asset files
- **Data-driven**: Game content (bosses, enemies, themes) is defined as configuration objects, not hard-coded in logic
- **Immutable canvas**: Game does not modify HTML elements during play; only updates text content (HUD) and CSS classes
- **Delta-time physics**: All movement and timing uses `dt` (delta seconds) for frame-rate independence

## Debugging Tips

- **Console logging**: Insert `console.log()` in `update()` to debug state changes
- **Visual debug**: Temporarily draw bounding boxes or collision areas to verify hitbox logic
- **Pause and step**: Use the pause button (P or Esc key) to pause, then add breakpoints in DevTools
- **localStorage inspection**: Check browser DevTools → Application → Local Storage to inspect saved settings
- **Performance profiling**: Use Chrome DevTools → Performance tab to capture frame-by-frame timing
- **Audio context state**: Check `audioCtx.state` if sound isn't working (may be 'suspended' until user interaction)
