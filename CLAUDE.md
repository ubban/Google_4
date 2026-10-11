# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single self-contained HTML file. The entire game engine, rendering, audio synthesis, and UI are implemented in a single `index.html` file with embedded CSS and JavaScript (~1,669 lines).

**Key Features:**
- Canvas-based 2D rendering at 480x720 logical resolution
- Synthesized sound effects using WebAudio API (no external audio files)
- Cross-platform controls: keyboard, mouse steering, and touch joystick
- Difficulty modes: Easy, Normal, Hard
- Progressive stages with thematic sky variations
- Boss fights with pattern-based attack systems
- Weapon upgrades (Vulcan/Laser), multipliers, and smart bombs
- Adaptive performance quality scaling based on frame time
- Local storage persistence for best score, settings, and preferences
- Reduced motion accessibility mode

## Architecture Overview

### High-Level Structure

The game is organized as an IIFE (Immediately Invoked Function Expression) containing several interconnected systems:

1. **Input System** - Keyboard, mouse, and touch handling
2. **Game State** - Player, enemies, bullets, powerups, boss
3. **Update Loop** - Physics, collision detection, scoring
4. **Rendering System** - Canvas-based visual output with animations
5. **Audio System** - WebAudio synthesizer for sound effects
6. **UI/HUD** - HTML overlay with score, lives, bombs, weapon status

### Game Loop

The main game loop (`requestAnimationFrame` at line ~1650+):
1. **Update phase** - Moves entities, handles input, updates timers
2. **Collision detection** - Player bullets vs enemies, enemy bullets vs player
3. **Render phase** - Draw background, entities, particles, HUD updates
4. **Frame tracking** - Performance monitoring for adaptive quality

### Core Game Systems

#### Player System
- Position-based movement (keyboard input, mouse tracking, touch joystick)
- Two weapons: Vulcan (rapid multi-shot) and Laser (concentrated beam)
- Weapon levels (1-3) with increased fire rate at higher levels
- Invulnerability frames on spawn and after hit
- Lives system with heart display

#### Enemy System
- **Enemy Types** (defined in `ENEMY_TYPES` constant, ~line 502):
  - `grunt`: 1 HP, slow, red, score 10
  - `interceptor`: 1 HP, fast and weaving, score 15
  - `tank`: 4 HP, slow, purple, score 35
- **Spawning** - Wave-based spawning controlled by difficulty and stage
- **Enemy AI** - Simple downward movement with occasional bullet fire
- **Bullet system** - Enemies fire red projectiles toward player

#### Boss System
- **Boss Types** (defined in `BOSSES` constant, ~line 508):
  - VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
  - Each with unique color scheme and HP multiplier
- **Attack Patterns** (defined in `BOSS_PATTERNS` array, ~line 1340+):
  - Each boss has 3 unique attack patterns (rotating)
  - Patterns include: homing shots, radial spreads, aimed streams, columns
- **Boss Behavior**:
  - Enters from top with animation
  - Bounces left-right while attacking
  - Telegraph phase before each attack
  - One boss per stage after 18 kills

#### Scoring & Multipliers
- Base score from kills (10-500 points)
- **Chain Multiplier** - Increases by 1 for consecutive hits, resets on miss
- **Combo Pop** - Visual indicator showing chain multiplier
- **Best score** - Persisted to localStorage

#### Weapon Power-ups
- Drop randomly when enemies die
- Vulcan: Increases weapon level (up to 3)
- Laser: Switches to laser weapon
- Drop visual: Yellow star that moves toward player

#### Bomb System
- Limited count (2-3 depending on difficulty)
- Smart bomb: Clears all on-screen bullets and damages all enemies
- Visual feedback: Screen flash and vignette pulse
- Detonates with specialized audio effect

### Visual & Audio

#### Rendering Details
- **Background**: Animated gradient sky with moving clouds
- **Themes**: 6 different sky themes that cycle per stage
- **Particles**: Explosion effects with colored squares
- **Screen shake**: Knockback effect on damage/bomb, intensity controlled
- **Flash effects**: White flash on hit, different animation for bomb
- **Quality scaling**: Glow effects disabled on low-performance devices

#### Audio System
- **WebAudio Synth**: Procedural sound generation (no asset files)
- **Core functions**:
  - `beep()` - Oscillator-based sound with frequency sweep, filtering
  - `noiseBurst()` - White noise with filter envelope
- **SFX library** (`sfx` object): Indexed sound effects for:
  - Weapons: `shoot`, `laser`
  - Explosions: Enemy-type specific (grunt, interceptor, tank)
  - Events: `hit`, `bomb`, `powerup`, `bossTelegraph`, `bossHit`, `bossDown`, `stageClear`, `gameover`
  - UI: `uiClick`

### Input Handling

#### Keyboard
- **Arrow keys / WASD** - Movement
- **Space** - Fire
- **X / Shift** - Bomb
- **P / Esc** - Pause

#### Mouse
- **Movement** - Player glides toward pointer when hovering canvas
- **Click/Hold** - Fire continuously

#### Touch
- **Left zone (55% width)** - Joystick for movement
- **Right zone (45% width)** - Fire and Bomb buttons
- Shows circular indicators for joystick base and stick

#### Priority System
Keyboard and joystick input take priority over mouse, preventing input conflicts.

### State Management

#### Difficulty Modes
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

#### LocalStorage Keys
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio mute state
- `skyfighter.difficulty` - Selected difficulty
- `skyfighter.reducedMotion` - Reduced motion preference

#### Performance Adaptation
Tracks frame time over 45 frames and adjusts quality (full → medium → low):
- Low quality disables glow effects (ctx.shadowBlur)
- Respects user's prefers-reduced-motion media query

## Development & Modification Guide

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` constant (~line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add corresponding explosion sound in `sfx` object
3. Add to `EXPLODE_SFX` mapping if custom explosion needed
4. Add spawn logic in enemy spawning section

### Adding a New Boss
1. Add to `BOSSES` array (~line 508) with name and color scheme
2. Create attack patterns in `BOSS_PATTERNS` array (~line 1340+)
3. Patterns are arrays of functions that fire bullets
4. Use existing pattern functions: `fireRadial()`, `fireColumns()`, `fireAimedSpread()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Modifying Game Balance
- **Enemy difficulty**: Adjust `DIFFICULTIES` object
- **Spawn rate**: Change `spawnBase` and `spawnMin` values
- **Enemy HP**: Modify `hp` property in `ENEMY_TYPES`
- **Weapon damage**: Change `damage` property in bullet objects (created in fire functions)
- **Stage progression**: Change `KILLS_PER_STAGE` constant (line 491)
- **Boss HP**: Adjust `bossHpMul` in boss design or difficulty

### Modifying Weapons
Weapon fire patterns are in the main update loop. Vulcan fires from `fireVulcan()` function, Laser from `fireLaser()` function. These use `createPlayerBullet()` to spawn bullets with specific properties.

### Adjusting Visuals
- **Sky themes**: Modify `THEMES` array (~line 493) with new gradient colors
- **Screen shake**: Adjust `screenShake(mag, dur)` call parameters
- **Particle effects**: Modify `spawnExplosion()` parameters
- **Canvas resolution**: Change `BASE_W` and `BASE_H` constants (~line 753)

### Audio Customization
Modify `sfx` object definitions to change sound characteristics:
- Frequency values control pitch
- Duration affects sound length
- `sweepTo` creates frequency glides
- `filterFreq` controls tone filtering
- Use `delay` parameter for multi-layer sounds

## Testing & Validation

Since this is a single HTML file:
- **No build step required** - File works directly in any modern browser
- **Cross-browser testing** - Test in Chrome, Firefox, Safari, Edge
- **Mobile testing** - Test touch controls on mobile devices or Chrome DevTools device emulation
- **Accessibility** - Test with reduced motion enabled, keyboard-only controls, screen reader announcements
- **Performance** - Monitor FPS on low-end devices; quality scaling should adapt automatically

## Browser Compatibility

Requires:
- Canvas 2D context (`canvas.getContext('2d')`)
- WebAudio API (gracefully falls back to muted if unavailable)
- LocalStorage (gracefully falls back if unavailable)
- Touch Events (for mobile, optional)
- Modern JavaScript (ES6 arrow functions, template literals, `const`/`let`)

Minimum supported: Chrome/Firefox/Safari/Edge from 2015+

## Performance Notes

- Canvas is 480x720 logical pixels, scaled to screen size with max 1.6x magnification
- Adaptive quality system reduces shadow effects on low-end devices
- Frame rate cap via `requestAnimationFrame` (typically 60 FPS)
- Enemy bullet objects reuse same properties for memory efficiency
- Particles are pooled implicitly (created on demand, never explicitly pooled)
