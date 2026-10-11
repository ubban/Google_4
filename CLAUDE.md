# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 canvas application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (1,669 lines).

### Key Features
- Canvas-based rendering with adaptive quality levels
- WebAudio API-synthesized sound effects (no external audio files)
- Three difficulty levels (Easy, Normal, Hard)
- 6 distinct boss encounters with unique attack patterns
- Power-up system (weapon upgrades, bombs, health)
- Touch controls for mobile, keyboard/mouse for desktop
- Persistent high score storage
- Accessibility features (screen reader announcements, reduced motion support)

## Architecture Overview

The entire application uses a single-file architecture with the following organization:

### Initialization & Globals (lines 426-527)
- DOM element references for all UI components
- LocalStorage keys for persistence (bestScore, muted, difficulty, reducedMotion)
- Game state variables (running, paused, gameOver, score, lives, stage, etc.)
- Quality adaptation system for performance tracking

### Core Systems

**1. Audio System (lines 542-676)**
- `ensureAudio()`: Lazy initializes WebAudio context with compressor chain
- `beep(freq, dur, type, vol, opts)`: Synthesizes tones with frequency sweeps, filters, and panning
- `noiseBurst(dur, vol, opts)`: Generates noise bursts with filtering
- `sfx` object: Pre-configured sound effects for all game events
- Spatial audio: enemy bullets and explosions pan based on screen position

**2. Rendering & Canvas (lines 698-1620+)**
- `draw()`: Main render function that clears canvas, applies effects (shake, flash), draws layers:
  - Background sky gradient with sun and clouds
  - Player aircraft with weapon visualization
  - All bullets (player and enemy)
  - Enemies and bosses
  - Power-ups and particle effects
- Dynamic theme system: 6 visual themes per stage, cycling based on stage number
- Quality-aware rendering: Shadow effects and glows disabled when quality > 0 (performance mode)
- Screen shake and flash effects for feedback

**3. Game Update Loop (lines 1148-1310)**
- `update(dt)`: Called every frame with delta time
- Player movement (keyboard: arrows/WASD, mouse: follow cursor, touch: joystick)
- Bullet updates and collision detection with enemies/bosses
- Enemy spawning, movement, and AI (weaving patterns, shoot chance)
- Power-up collision and application
- Score multiplier decay and combo system
- Boss phase transitions

**4. Entity Systems**
- **Player**: Single plane with weapon type (Vulcan/Laser), health, invulnerability, bomb count
- **Enemies**: Three types with progression
  - Grunt: Fast, weak, basic spawning
  - Interceptor: Very fast, weaving patterns
  - Tank: Slow, high HP, high score
- **Boss**: Single entity per stage with complex attack patterns, health bar tracking
- **Bullets**: Player bullets (simple), Enemy bullets (simple), Boss bullets (homing, radial, columnar)
- **Power-ups**: Life, bomb, weapon (extends Vulcan level or switches to Laser)
- **Particles**: Explosion effects with fade and physics

**5. Combat & Scoring**
- `fireWeapon()`: Player shooting, weapon-specific behavior
- `useBomb()`: Screen-wide damage, invulnerability window
- Multiplier system: Score multiplier increases with consecutive kills, resets on death
- Boss-specific score bonuses (500 points per boss)
- Enemy kills award base score × multiplier

**6. Difficulty System (lines 485-489)**
- Three presets: Easy, Normal, Hard
- Parameters: spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Stored in localStorage; user can switch on title screen

**7. Boss Battles (lines 508-1409)**
- 6 unique boss designs with distinct colors and HP multipliers
- 6 separate attack pattern arrays in `BOSS_PATTERNS`
- Boss patterns use helper functions:
  - `fireRadial(b, count, speed, offset)`: Ring attacks
  - `fireDoubleRing(b, speed)`: Double concentric rings
  - `fireColumns(b, speed)`: Vertical columns
  - `fireAimedSpread(b, count, speed, spread)`: Player-aimed patterns
  - `fireHomingSeeded(b, count, speed)`: Homing bullets
- Telegraph system: 22-frame warning before each attack
- Health bar shows boss HP percentage

**8. Input Handling**
- Keyboard: Arrow keys/WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
- Mouse: Movement tracks cursor, click to fire
- Touch: Left zone for movement joystick, right zone for FIRE/BOMB buttons
- `canvasPointFromEvent(e)`: Converts mouse/touch coordinates to canvas space

**9. State Management**
- Game states: Title screen → Game → Paused → Game Over
- Pause state preserves all entity state; can resume without reset
- Score, stage, lives, and multiplier persist within a single run

### Configuration & Constants

**Difficulty Presets** (lines 485-489)
```
DIFFICULTIES = { easy, normal, hard }
Each with: spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
```

**Visual Themes** (lines 493-500)
```
THEMES = 6 arrays of sky gradients, cloud color, sun color, and theme name
Accessed by (stage - 1) % THEMES.length
```

**Enemy Types** (lines 502-506)
```
ENEMY_TYPES = { grunt, interceptor, tank }
Each with: width, height, hp, speedMul, color, dark (for shading), score, minStage
```

**Boss Designs** (lines 508-515)
```
BOSSES = 6 unique bosses with hull colors, wing colors, core color, and hpMul
```

**Game Constants**
- `KILLS_PER_STAGE = 18`: Enemies to kill before boss spawn
- Quality levels: 0 (full effects), 1 (medium), 2 (low)
- Screen shake magnitude and duration hardcoded per event

### Performance Optimization

- **Adaptive Quality**: Frame-time tracking adjusts visual effects
  - Full quality: All shadows and glows
  - Medium quality: Reduced glow effects
  - Low quality: No shadow/glow effects
- **Particle System**: Explosions use simple circular particles with fade
- **Canvas Rendering**: Single-pass draw minimizes context switches
- **requestAnimationFrame**: Native timing loop, delta-time clamped to 50ms max

## Common Development Tasks

### Adding a New Boss
1. Add entry to `BOSSES` array with unique colors (hullTop, hullBottom, wing, core) and hpMul
2. Add attack pattern array to `BOSS_PATTERNS` with 2–3 pattern functions
3. Use existing bullet-fire helpers: `fireRadial`, `fireColumns`, `fireAimedSpread`, `fireHomingSeeded`
4. Boss is automatically selected sequentially per stage

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties (w, h, hp, speedMul, color, dark, score, minStage)
2. Update `ENEMY_TYPES.*.minStage` to control when it first appears
3. Add any unique behavior in the `update()` enemy loop (weaving patterns already exist)

### Tweaking Game Balance
- Edit `DIFFICULTIES` object for spawn rates and enemy speed
- Modify score base values in `addScore()` and boss kill bonus
- Adjust multiplier decay in `breakChain()`
- Edit weapon cooldown in `fireWeapon()` or damage via enemy HP values

### Adding Sound Effects
1. Create new function in the `sfx` object calling `beep()` or `noiseBurst()` 
2. Tune frequency, duration, waveform type, and filter parameters
3. Call `sfx.newEffect()` in relevant game event (spawn, hit, powerup, etc.)

### Accessibility & Reduced Motion
- Use `reducedMotion` flag to disable animations; already integrated into CSS and JS
- Update `announce(text)` calls for screen reader feedback on significant game events
- All buttons have `title` attributes for tooltips

## Rendering & Visual Effects

The render pipeline follows a specific order for correct depth:
1. Background (sky gradient, sun glow, clouds)
2. Player with weapon indicator
3. All bullets (player bullets, then enemy bullets)
4. All entities (enemies, bosses, power-ups)
5. Particles (explosions)
6. UI overlays (HUD, pause screen, effects like flash or vignette)

Screen effects:
- **Screen Shake**: Applies random translation to context; magnitude and duration vary by event
- **Flash**: White overlay with fade; used for hits and bombs
- **Vignette**: Red pulsing edge overlay; enabled when health is critical

## Testing & Validation

Since this is a browser-based game with no build step:
1. Open `index.html` in a browser
2. Click **Start Game** to begin
3. Test difficulty levels on title screen
4. Verify boss spawning after 18 enemy kills
5. Check sound synthesis works (ensure browser allows WebAudio)
6. Test touch controls by resizing to mobile viewport or using touch device
7. High score persistence: complete a game, reload page, verify best score is retained

## Browser & Platform Notes

- **Canvas Rendering**: Uses 2D context, tested on modern browsers (Chrome, Firefox, Safari)
- **WebAudio**: Requires browser WebAudio API support; gracefully degrades if unavailable
- **LocalStorage**: Required for high score and settings persistence
- **Touch**: Full touch support for mobile; joystick-based movement and button-based firing
- **Accessibility**: Screen reader announcements via `aria-live` region; respects `prefers-reduced-motion`
