# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. It's a single-file (~1670 lines) browser-based game with no build tools or external dependencies.

### Game Features
- Three difficulty levels (Easy, Normal, Hard)
- Multiple weapon types with upgrade levels (Vulcan, Laser)
- Smart bomb mechanic for clearing enemies
- Six stage-based bosses with distinct attack patterns
- Combo/multiplier scoring system
- Web Audio API synthesized sound effects
- Mobile touch controls and desktop keyboard/mouse input
- Adaptive quality settings based on frame performance
- Persistent high score storage (localStorage)
- Screen shake and visual feedback effects
- Accessibility features (reduced motion support, screen reader announcements)

## Running the Game

**Desktop browser**: Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge).

**Local development server** (optional, for testing):
```bash
# Using Python 3
python3 -m http.server 8000

# Using Node.js
npx http-server

# Using Ruby
ruby -run -ehttpd . -p8000
```

Then navigate to `http://localhost:8000/index.html`

## Code Architecture

All game code is in a single `<script>` tag (lines 422–1666 in `index.html`). The code is organized into logical sections:

### 1. **Setup & Configuration** (lines 423–467)
- DOM element references (canvas, HUD, buttons)
- Storage key constants for persistence (best score, mute state, etc.)
- Audio context initialization

### 2. **Difficulty & Game Constants** (lines 485–515)
- `DIFFICULTIES`: object with spawn rates, enemy behavior, boss HP multipliers for each difficulty
- `KILLS_PER_STAGE`: kills needed to trigger boss spawn (18)
- `THEMES`: 6 visual themes with sky gradients and cloud colors
- `ENEMY_TYPES`: grunt, interceptor, tank with properties (HP, speed, scoring)
- `BOSSES`: 6 boss designs with color schemes and HP multipliers

### 3. **Audio System** (lines 542–676)
- `ensureAudio()`: lazy-initializes Web Audio API context with compressor
- `beep()`: synthesizes tonal sound effects with optional sweep and filtering
- `noiseBurst()`: creates filtered white noise for explosion sounds
- `sfx` object: named sound effects (shoot, laser, explode, bomb, hit, powerup, etc.)
  - All SFX are generated in real-time; no audio files needed
  - Respects mute state and reduced motion preference

### 4. **Rendering & UI** (lines 680–730)
- `renderHearts()`, `renderBombs()`: Update HUD display
- `flashHit()`, `flashBomb()`: Visual feedback animations
- `showComboPop()`, `showStageBanner()`: Floating text overlays
- `screenShake()`: Camera shake effect (reduced in reduced-motion mode)

### 5. **Canvas Sizing & Input** (lines 752–862)
- `resize()`: Scales canvas to window while maintaining 480×720 aspect ratio
- Keyboard input: Arrow keys, WASD for movement; Space, X, Shift for fire/bomb; P or Esc to pause
- Mouse input: Pointer position steers the plane; click to fire
- Touch input: Left side joystick for movement; right buttons for fire/bomb
- Input buffering: `keys` object tracks held keys; `firing` and joystick state updated in real-time

### 6. **Game State** (lines 863–911)
- `resetGame()`: Initializes all entities and state for a new game run
- **Game objects**:
  - `player`: position, speed, weapon, level, invulnerability timer
  - `enemies`: array of spawned enemy ships
  - `boss`: current boss encounter or null
  - `bullets`, `enemyBullets`, `powerUps`, `particles`: projectiles and effects
- **Game variables**:
  - `score`, `lives`, `stage`, `kills`, `stageKills`
  - `chain`, `chainTimer`, `multiplier`: combo system
  - `running`, `paused`, `gameOver`: state flags
  - `frame`: incremental frame counter (never resets during session)

### 7. **Entity Rendering** (lines 913–1020)
- `drawPlane()`: Renders player/enemy ships using canvas paths (main hull, wings, cockpit, flame)
- `drawBossShip()`: Draws boss with two-tone gradient hull, glowing core, and rotating engines
- Both accept color parameters for customization

### 8. **Spawning & Collisions** (lines 1022–1071)
- `spawnEnemy()`: Creates enemy with random type, position, and behavior pattern
- `spawnBoss()`: Initiates stage boss with design, HP, and attack timers
- `maybeSpawnPowerUp()`: Random drop (6% life, 8% bomb, 20% weapon)
- `rectsOverlap()`: AABB collision detection for all entities
- `spawnExplosion()`: Creates particle burst with adaptive particle count based on quality setting

### 9. **Combat & Scoring** (lines 1087–1146)
- `addScore()`: Increments score and chain counter; updates multiplier (1–10x based on kills)
- `breakChain()`: Resets combo if player takes damage or too much time passes
- `fireWeapon()`: Spawns bullets based on weapon type and upgrade level
  - **Vulcan**: vertical bullets with spread pattern (1/2/3 bullets at levels 1/2/3)
  - **Laser**: piercing beams with wider lane spacing at higher levels
- `useBomb()`: Clears all enemies and enemy bullets; damages boss; resets chain

### 10. **Game Loop & Update** (lines 1148–1302)
- `update(dt)`: Core per-frame logic
  1. **Input processing**: Accumulate keyboard, joystick, and mouse input into movement vector
  2. **Player movement**: Clamp position within screen bounds
  3. **Weapon firing**: Trigger based on held fire button and cooldown
  4. **Enemy spawning**: Increase frequency over time; spawn boss after 18 kills
  5. **Entity updates**: Move bullets, enemies, particles; apply weave patterns to interceptors
  6. **Collisions**: Check bullets vs. enemies/boss; enemies vs. player; bombs vs. enemies; powerups vs. player
  7. **Chain timer decay**: Decrease chain timeout; break combo if expired
- `draw()`: Renders background gradient, clouds, entities, particles with optional shake offset

### 11. **Boss Encounters** (lines 1324–1439)
- `updateBoss(dt)`: Handles boss movement, telegraph indicator, and attack pattern firing
  - Boss bounces horizontally; enters from top on spawn
  - 22-frame telegraph before each attack (white glow pulse)
  - Patterns fire from `BOSS_PATTERNS` array
- **Pattern firing functions**:
  - `fireRadial()`: Radial spread of bullets
  - `fireAimedSpread()`: Aimed shot with spread angle
  - `fireColumns()`: Vertical columns
  - `fireSpiralBurst()`: Expanding spiral
  - `fireHomingSeeded()`: Homing projectiles with random drift
  - `fireDoubleRing()`: Two concentric circles at different speeds
- Each of 6 bosses has 3 unique attack patterns; patterns cycle and increase intensity

### 12. **Game Flow** (lines 1440–1505)
- `killBoss()`: Award points, increment stage, show banner
- `loseLife()`: Apply invulnerability, break chain, check for game over
- `endGame()`: Save best score, show game-over screen with stats
- `togglePause()`: Switch overlay state; pause button toggles resume/paused text

### 13. **Performance Adaptation** (lines 473–483)
- `trackPerf(dt)`: Measures frame time over 45 frames
- `quality` levels (0=full, 1=medium, 2=low) reduce particle counts, glow effects, and shadow blur
- Automatically scales based on average FPS

### 14. **Main Loop** (lines 1640–1666)
- `requestAnimationFrame(loop)`: Continuous game loop
  - Delta-time capped at 50ms to prevent runaway updates
  - Only calls update/draw when `running && !paused`
  - Tracks performance and renders last frame even during pause

## Key Algorithms & Design Patterns

### Combo/Multiplier System
- Every enemy kill increments `chain` counter and resets `chainTimer` to 2.2 seconds
- `multiplier = 1 + floor(chain / 5)` caps at 10x
- Damage, time passing, or getting hit breaks the chain to 1x
- Visual feedback: multiplier text color changes (orange), floating text appears every 10 kills

### Difficulty Ramping
- Base enemy spawn rate decreases over time: `spawnMin = spawnBase - floor(frame / 300) * 4`
- Boss HP scales by difficulty (`bossHpMul`) and boss design (`hpMul`)
- Enemy AI: `patternRoll` (0–1) picks attack pattern (spread or aimed); weave pattern applies sine oscillation

### Input Abstraction
- Movement input comes from three sources (keyboard, joystick, mouse) with fixed priority
- Keyboard/joystick take absolute priority; mouse used only if no active input
- Touch joystick detected via `ontouchstart` and `navigator.maxTouchPoints`; activates mobile UI

### State Machine (Implicit)
- States: menu overlay → playing → paused overlay → playing → game-over overlay
- Overlay visibility gates input (pause button disabled during game-over, etc.)
- `running`, `paused`, `gameOver` flags control behavior branching

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with name, dimensions, HP, speed multiplier, color, score
2. Add sound effect if needed in `sfx` object (lines 624–676)
3. Register the SFX in `EXPLODE_SFX` (line 678) if custom explosion sound
4. Update `spawnEnemy()` (line 1026) to include the type in spawn logic
5. Note: `minStage` property gates when type spawns (e.g., tanks only after stage 2)

### Adding a New Boss
1. Add to `BOSSES` array (line 508) with name, hull colors, core color, HP multiplier
2. Create attack pattern functions (e.g., `fireMyPattern(b)`) or reuse existing ones
3. Add 3 pattern functions to `BOSS_PATTERNS` array (line 1372) in the same index as the boss
4. Boss index is `(stage - 1) % BOSSES.length`, so adding a 7th boss shifts rotation

### Tuning Difficulty
- Modify `DIFFICULTIES` object (line 485): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- Spawn rate formula: `spawnTimer = Math.max(spawnMin, spawnBase - difficultyRamp)`
- Lower spawn values = more frequent; use frame number to ramp up over time

### Tweaking Visuals
- Sky gradients in `THEMES` (line 493): adjust RGB stops for gradient
- Plane colors in draw functions: primary color in first param, dark/shadow in second
- Canvas size `BASE_W = 480, BASE_H = 720` (line 753); increases scale automatically on large screens
- Shadow/glow effects controlled by `quality` and enabled in draw loops (e.g., line 1002)

### Audio Tuning
- All SFX in `sfx` object call `beep()` or `noiseBurst()`
- `beep(freq, duration, type, volume, options)`: freq in Hz, type in ['sine', 'square', 'sawtooth', 'triangle']
- `options.sweepTo` for frequency sweep; `options.filterFreq` for cutoff; `options.pan` for stereo position
- `masterGain.gain.value` (line 556) controls overall volume (default 0.9)

## Testing & Debugging

### Manual Testing Checklist
- **Keyboard**: Arrow/WASD movement, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move to steer; click to fire; verify crosshair follows
- **Touch** (mobile): Joystick on left; FIRE/BOMB buttons on right; verify responsive
- **Audio**: Mute button in top-right; sound effects play on shoot, hit, bomb, boss defeat
- **Scoring**: Chain counter increments; multiplier appears every 10 kills; best score persists
- **Bosses**: Appear after 18 kills; telegraph animation before each pattern; health bar shows
- **Difficulty**: Easy has more lives; Hard has more enemies; confirm spawn rates differ
- **Performance**: Frame rate stable on low-end devices; quality auto-scales; check console for errors

### Browser Console Debugging
- `gameState` or relevant global variables can be inspected in DevTools
- Add temporary `console.log()` inside `update()` or `draw()` to trace behavior
- Frame counter `frame` is global; use for time-based debugging
- `quality` variable shows current performance tier (0=full, 1=medium, 2=low)

### Common Issues
- **Audio not playing**: Web Audio Context suspended; click canvas first to resume
- **Touch controls not showing**: Check `isTouchDevice` detection; may need to trigger touch event
- **Low FPS on old devices**: `quality` should auto-degrade; confirm `trackPerf()` is running
- **Collision feels off**: Adjust `rectsOverlap()` tolerance or check AABB dimensions in entity definitions

## Performance Considerations

- **Canvas rendering**: All 2D shapes drawn per frame; no WebGL; no image assets
- **Particle system**: Adaptive count based on `quality` (full=14, medium=8, low=5 per explosion)
- **Audio**: Web Audio synth runs in separate thread; no memory leaks if muted
- **Storage**: localStorage used only for best score, mute, difficulty, reduced-motion (< 100 bytes)
- **Mobile**: Touch events are passive (non-blocking); joystick uses pointer events for precision

## Accessibility

- **Screen reader support**: Aria-live region `#srLive` announces score, bosses, deaths
- **Reduced motion**: Checkbox in top-right; disables shake, flash, pulsing animations
- **Color contrast**: Text shadows ensure legibility on all background gradients
- **Keyboard full support**: All features accessible via keyboard (no mouse required)
- **Touch accessible**: Large touch targets (78×78px buttons) for mobile

## Browser Support

- Requires: ES6+ JavaScript, HTML5 Canvas 2D context, Web Audio API, localStorage
- Tested on: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Mobile: iOS Safari 12+, Chrome/Firefox Android
- Not supported: IE11 (uses ES6 arrow functions, const/let)
