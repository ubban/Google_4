# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application with embedded CSS and JavaScript. The game features:
- Multiple difficulty levels (easy, normal, hard)
- 6 unique boss encounters with distinct attack patterns
- Dynamic stage progression with enemy spawning
- Weapon power-up system with progression levels
- Score multiplier chains and combo system
- Touch controls for mobile devices
- Accessibility features (screen reader support, reduced-motion preferences)
- Persistent high score tracking via localStorage

## Quick Start

**Playing the game:**
- Open `index.html` in any modern web browser
- Use mouse cursor to aim, click to fire, Space/X to drop bombs
- Keyboard: Arrow keys/WASD to move, Space to fire, X or Shift for bombs
- Mobile: Drag left half of screen to move, tap FIRE/BOMB buttons

**Development:**
- No build process required
- Edit `index.html` directly
- Changes take effect on refresh
- Test in browser with DevTools console open for debugging

## Code Architecture

The entire game is a single ~1670-line JavaScript IIFE (Immediately Invoked Function Expression) wrapped in HTML/CSS, running inside `<script>` tags at the end of the document (line 422+).

### High-Level Structure

1. **DOM Element References** (lines 426–461)
   - Canvas, UI containers, HUD elements, buttons
   
2. **Game Constants & Configuration** (lines 463–519)
   - Storage key strings for localStorage
   - `DIFFICULTIES`: defines easy/normal/hard parameters (spawn rates, enemy speed, health, lives, bombs)
   - `KILLS_PER_STAGE`: number of enemies to kill before boss spawns (18)
   - `THEMES`: 6 different sky gradients with visual themes
   - `ENEMY_TYPES`: defines enemy variants (grunt, interceptor, tank) with their properties
   - `BOSSES`: 6 boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
   - `BOSS_PATTERNS`: 3-pattern attack sequences for each boss
   
3. **Audio System** (lines 545–677)
   - Web Audio API procedural sound generation (no audio files)
   - `ensureAudio()`: initializes audio context on first user interaction
   - `beep()`: generates tones (frequency, duration, wave type)
   - `noiseBurst()`: generates explosion/impact sounds
   - `sfx` object: named sound effects for gameplay events
   
4. **UI & Visual Effects** (lines 680–752)
   - `renderHearts()`, `renderBombs()`: HUD display
   - `flashHit()`, `flashBomb()`: screen flash effects
   - `showComboPop()`, `showStageBanner()`: floating text animations
   - `screenShake()`: camera shake effect
   - `updateMuteBtn()`, `setDifficulty()`: UI state management
   
5. **Rendering** (lines 754–1519)
   - `resize()`: scales canvas to fit viewport while maintaining 480×720 aspect
   - `drawPlane()`: renders player ship with different colors and animation states
   - `drawBossShip()`: renders boss sprite with animated wings/patterns
   - `spawnEnemy()`: creates random enemy at top of screen
   - `spawnBoss()`: creates boss when stage progresses
   - `drawClouds()`: parallax background clouds
   - `draw()`: main render function called each frame
   
6. **Game Loop & Input** (lines 768–1146)
   - Keyboard/mouse/touch input handling
   - `joystickStart/Move/End()`: mobile joystick controls
   - `mouseTarget`: mouse position tracking for aiming
   - `update(dt)`: main game logic per frame (player movement, collision, damage, scoring)
   
7. **Boss & Combat Systems** (lines 1148–1510)
   - `updateBoss()`: boss movement and attack patterns
   - Boss pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireHomingSeeded()`, etc.
   - `killBoss()`: boss defeat sequence
   - `loseLife()`, `endGame()`: player damage and game over
   
8. **Game State** (scoped within IIFE)
   - `gameRunning`, `paused`, `gameOver`: game state flags
   - `player`: player position, health, weapon, invulnerability, bomb count
   - `enemies[]`, `bullets[]`, `enemyBullets[]`, `particles[]`: entity arrays
   - `boss`: current boss object
   - `score`, `lives`, `stage`, `combo`: game metrics
   - Difficulty parameters applied from `DIFFICULTIES`

9. **Main Loop** (lines 1652–1665)
   - `requestAnimationFrame(loop)` calls `update()` then `draw()` each frame
   - `dt` (delta time) scales movement for frame-rate independence

## Key Game Mechanics

### Weapon System
- **Weapon progression**: 4 types (Vulcan, Piercer, Spreader, Laser) with 3 levels each
- Weapon pick-ups appear when enemies drop them
- Cycles through types on pickup; increases level if same type

### Difficulty Settings
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

### Boss Encounters
- Each of 6 bosses has unique sprite, health, movement, and 3-attack patterns
- Bosses telegraph attacks before firing (22 frame delay)
- Pattern cycles on loop; increases in difficulty as battle progresses

### Scoring & Combos
- Base kill score varies by enemy type
- Combo multiplier (1.0–3.0×) applied on consecutive kills within ~2 seconds
- Boss kills worth 500 points + multiplier
- High score persisted to localStorage

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant with `w`, `h`, `hp`, `score`, `dropChance`
2. Add explosion type to `EXPLODE_SFX` if custom sound needed
3. Update enemy drawing logic in `draw()` if unique sprite required
4. Adjust `spawnEnemy()` spawn weights if needed

### Modifying Boss Attack Pattern
1. Find desired boss in `BOSS_PATTERNS` array
2. Edit one of the 3 pattern functions (fire functions like `fireRadial()`, `fireColumns()`, etc.)
3. Test in browser to verify visual feedback and difficulty

### Tweaking Difficulty
Edit `DIFFICULTIES` object:
- `spawnBase/spawnMin`: control enemy spawn frequency
- `enemySpeed`: bullet/movement velocity multiplier
- `enemyShootChance`: probability enemy fires each frame
- `lives`: starting player health
- `bombs`: starting bomb count
- `bossHpMul`: boss health multiplier

### Adding Sound Effects
1. Add new function in audio system section using `beep()` or `noiseBurst()`
2. Add named function to `sfx` object
3. Call `sfx.effectName()` where needed

## Debug Tips

- Open browser DevTools (F12)
- Game state variables are scoped in IIFE; use breakpoints in Sources tab
- Check Console for `announce()` output (accessibility announcements)
- Performance: `trackPerf()` automatically adjusts quality level (visible in `quality` variable)
- Audio context requires user interaction first—click/tap screen before sound plays

## File Locations & Storage Keys

- **Game data storage** (localStorage):
  - `skyfighter.bestScore`: high score
  - `skyfighter.muted`: mute state
  - `skyfighter.difficulty`: selected difficulty
  - `skyfighter.reducedMotion`: accessibility preference

- **Viewport rendering**: Canvas scales to fit window (max 100% × 100%) while maintaining 480×720 game world ratio
- **Safe area**: Uses CSS `env(safe-area-inset-*)` for notch/home-indicator awareness on mobile

## Performance Considerations

- Adaptive quality system reduces particle/rendering detail if frame time exceeds 1/40 second
- Bullet pools and entity arrays are recycled, not created/destroyed per-bullet
- Web Audio synth sounds are generated once per call (no streaming)
- No external assets—all graphics drawn procedurally on canvas
