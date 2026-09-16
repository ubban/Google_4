# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1670 lines). No build step, bundler, or external dependencies are required—just open the file in a browser to play.

### Key Features
- **Controls**: Keyboard (arrows/WASD + Space/X), mouse (move to steer, click to fire), and touch (drag to move, tap FIRE/BOMB)
- **6 Themed Stages**: Each with unique visual themes (daylight, sunset, night, storm, arctic, volcanic)
- **6 Bosses**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN—each with unique attack patterns
- **3 Difficulty Levels**: Easy, Normal, Hard (affects spawn rates, enemy speed, boss HP)
- **Weapon System**: Vulcan (spread shot) and Laser (piercing beam) with 3 upgrade levels each
- **Power-ups**: Life restore, bomb ammo, weapon upgrades
- **Accessibility**: Reduced motion mode, screen reader announcements, safe-area-inset support for notched devices
- **Persistence**: Best score, difficulty, mute state, motion preference via localStorage

## Running the Game

### Local Development
```bash
# Simplest: open directly in a browser (file:// protocol works fine)
open index.html

# Or serve via simple HTTP server (required for some audio features)
python -m http.server 8000
# Then visit http://localhost:8000
```

### Browser Compatibility
- Requires HTML5 Canvas, WebAudio, and localStorage
- Modern browsers (Chrome, Firefox, Safari, Edge) from ~2020 onwards
- Mobile: iOS Safari, Chrome on Android

## Code Architecture

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) starting at line 422. The code flows in this order:

### 1. **DOM & State Setup** (lines 426–527)
   - Cache all DOM elements for HUD, overlay, buttons, canvas
   - Initialize localStorage keys and settings (mute, difficulty, reduced motion, best score)

### 2. **Utilities & Core Systems** (lines 529–710)
   - **Motion & Shake**: `updateMotionBtn()`, `toggleReducedMotion()`, `screenShake()`
   - **Audio (WebAudio Synth)**: `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` object with all sound effects
   - **Visual Feedback**: `flashHit()`, `flashBomb()`, `showComboPop()`, `showStageBanner()`, hearts/bombs rendering

### 3. **Game Configuration** (lines 485–515)
   - `DIFFICULTIES`: Parameters for easy/normal/hard (spawn rates, enemy speed, lives, bomb count, boss HP multiplier)
   - `THEMES`: 6 color schemes for backgrounds and sky effects
   - `ENEMY_TYPES`: grunt, interceptor, tank with stats (HP, speed multiplier, score value)
   - `BOSSES`: 6 boss definitions with visual colors and HP multiplier
   - `BOSS_PATTERNS`: Attack pattern functions for each boss type

### 4. **Canvas Rendering** (lines 752–765)
   - `resize()`: Maintains BASE_W=480, BASE_H=720 internal resolution, scales to fit screen
   - Handles viewport orientation changes

### 5. **Input Handling** (lines 767–861)
   - **Keyboard**: Arrow keys / WASD for movement, Space/X for fire/bomb, P/Esc to pause
   - **Mouse**: Move cursor to steer, click to fire; mouse target glides player smoothly
   - **Touch**: Left zone for joystick movement, right zone for FIRE and BOMB buttons
   - **Pause Button**: Top-right menu button

### 6. **Game State & Logic** (lines 863–1302)
   - `resetGame()`: Initializes all entities and state for a new game
   - `update(dt)`: Main game loop update—handles movement, spawning, collisions, scoring
   - `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`: Entity creation
   - **Collision System**: `rectsOverlap()` for AABB hit detection
   - **Scoring & Chains**: `addScore()`, `chain` multiplier that resets after 2.2s of no kills
   - **Weapon System**: `fireWeapon()` spawns bullets (vulcan spread or laser pierce); level 1–3 affects projectile count
   - **Power-ups**: `applyPowerUp()` handles life/bomb/weapon effects
   - **Particle System**: Explosions and visual feedback

### 7. **Boss AI** (lines 1324–1439)
   - **Attack Patterns**: Radial burst, aimed spread, columns, spiral, double rings, homing missiles, etc.
   - `BOSS_PATTERNS`: Array of 6 arrays (one per boss type); each contains 3 unique attack functions
   - `updateBoss()`: Boss movement, telegraph phase (warning before attack), attack execution
   - Each boss cycles through its 3 patterns and bounces horizontally

### 8. **Game Over & Pause** (lines 1441–1504)
   - `killBoss()`: Boss defeat → next stage
   - `loseLife()`: Collision damage → invulnerability frames
   - `endGame()`: Game over screen with final stats; save best score to localStorage
   - `togglePause()`: Show/hide overlay; pause/resume game

### 9. **Rendering** (lines 1507–1637)
   - `drawClouds()`: Parallax scrolling cloud layers
   - `drawPlane()`: Player and enemy ship shapes (color fills + dark accents)
   - `drawBossShip()`: Boss rendering with animated core glow and wing movement
   - Main `draw()`: Sky gradient, clouds, projectiles, enemies, boss, particles, HUD updates
   - **Screen Shake**: Random translation applied if `shakeTime > 0`

### 10. **Main Loop** (lines 1640–1665)
   - `loop(now)`: Runs via `requestAnimationFrame`
   - Calculates `dt` (delta time), caps at 0.05s to prevent large jumps
   - Tracks performance via `trackPerf()` to adjust quality level (particle counts, glow effects)
   - Calls `update()` and `draw()` only if game is running and not paused

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line ~503) with w, h, hp, speedMul, color, dark, score, minStage
2. Add spawn conditions in `spawnEnemy()` (line ~1022) if desired
3. Add explosion SFX entry to `EXPLODE_SFX` (line ~678) pointing to existing `sfx.explode*` function

### Add a New Boss Attack Pattern
1. Create a function like `fireRadial()`, `fireAimedSpread()` etc. (lines 1324–1370)
2. Add pattern function to the relevant boss's array in `BOSS_PATTERNS` (lines 1372–1409)
3. Patterns receive the boss object; call `enemyBullets.push(...)` to spawn bullets
4. For radial bullets, set `radial: true` and use `dx`, `dy` instead of `vx`, `speed`

### Adjust Difficulty
Edit `DIFFICULTIES` (line 485):
- `spawnBase` / `spawnMin`: Lower = more frequent spawns
- `enemySpeed`: Enemy movement multiplier
- `enemyShootChance`: 0–1 probability of enemy firing
- `bossHpMul`: Boss HP multiplier

### Add a New Theme/Stage
1. Add to `THEMES` array (line 493): `{ sky: [color1, color2, color3], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }`
2. Themes cycle via `(stage - 1) % THEMES.length` (line ~1529)
3. Sky colors are CSS gradient stops; cloud/sun colors control parallax layers

### Add Sound Effects
- Sounds use WebAudio synth (no external files)
- Add function to `sfx` object (line 624) using `beep(freq, duration, type, volume, opts)` or `noiseBurst()`
- Call via `sfx.yourSoundName()` when needed
- `opts` can include: `sweepTo` (frequency ramp), `filterFreq`, `filterType`, `pan`, `delay`

### Test Locally
Since there's no build step:
1. Edit `index.html`
2. Refresh browser (hard refresh if cached: Ctrl+Shift+R or Cmd+Shift+R)
3. Use browser DevTools (F12) to check console for errors
4. Test all three difficulty levels, all bosses, and both weapons

### Performance Tuning
- Adaptive quality level tracked in `quality` variable (0=full, 1=medium, 2=low)
- Affects particle counts in `spawnExplosion()` and glow effects in `draw()`
- Automatically adjusts based on frame time average (checks frame times over ~45 frames)
- Reduced motion mode is user-controlled and caps screen shake + animations

## Key Design Patterns

### Delta Time
All physics use `dt` (seconds elapsed), not frames. Frame-based timers divide by 60: `timer -= dt * 60`

### Collision Detection
Simple AABB: `rectsOverlap(a, b)` checks if rectangles overlap (x and y center distance vs. half-width/height)

### Entity Arrays
Entities (bullets, enemies, powerups, particles) stored in arrays and updated/rendered in order. Out-of-bounds entities filtered after update step.

### State Machine
Game has states: `running`, `paused`, `gameOver`. Update and render behave differently for each.

### Procedural Bullet Patterns
Boss attack functions directly push bullets; patterns are stateless and deterministic per boss type.

### Chain Multiplier
Score multiplier increases for consecutive kills (1 multiplier per 5 kills). Resets if 2.2s elapses without a kill.

## Deployment

To deploy:
1. Commit and push `index.html` to your hosting (GitHub Pages, Netlify, AWS, etc.)
2. Serve as a static file—no server-side logic needed
3. HTTPS recommended for WebAudio and mobile fullscreen support

## Future Enhancement Ideas

- Add more boss types or attack patterns
- Implement level progression (boss health scaling by stage)
- Add combo multiplier visual effects
- Procedural level generation
- Leaderboard (requires backend)
- Mobile app wrapper (React Native / Capacitor)
- Local multiplayer (split screen)
- Persistent progression (stage/weapon unlocks)
