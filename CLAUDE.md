# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas 2D, and Web Audio API. The entire game is self-contained in a single `index.html` file (~1669 lines) with no external dependencies or build process.

### Key Features
- Three difficulty levels (Easy, Normal, Hard) with progressive spawning and enemy behavior
- 6 unique boss encounters with distinct attack patterns
- Dual weapon system: Vulcan (spread shot) and Laser (piercing) with 3 upgrade levels each
- Smart bomb mechanic for crowd control
- Power-up system (extra lives, bombs, weapons)
- Procedurally generated sound effects via Web Audio API
- Touch/mobile controls alongside keyboard and mouse input
- Adaptive quality settings based on frame performance
- LocalStorage persistence for best score and settings

## Architecture & Code Organization

### Single-File Structure
All code is in `index.html`:
- **Lines 1-359**: HTML structure & CSS styling
- **Lines 361-420**: HTML body and layout
- **Lines 422-end**: JavaScript game logic

### Code Sections (within `<script>`)

#### 1. **Initialization & DOM Setup** (lines 423-462)
Canvas and UI element references, constants setup.

#### 2. **Game Constants** (lines 485-515)
- `DIFFICULTIES`: Difficulty parameters (spawn rates, enemy behavior, lives)
- `THEMES`: Visual theme data (sky colors, cloud colors, sun colors) — 6 stages cycle through these
- `ENEMY_TYPES`: Enemy definitions (grunt, interceptor, tank) with HP, speed multiplier, score values
- `BOSSES`: 6 boss designs with visual styling and HP multipliers
- `BOSS_PATTERNS`: Attack patterns for each boss type

#### 3. **Audio System** (lines 542-676)
Web Audio synth-based SFX generation:
- `ensureAudio()`: Initializes AudioContext on first user interaction
- `beep()`: Creates tonal sound effects with optional frequency sweep and filters
- `noiseBurst()`: Creates noise-based impacts (explosions)
- `sfx` object: Maps game events to sound functions (e.g., `sfx.shoot()`, `sfx.bomb()`)

**Note**: No external audio files; all sounds synthesized in real-time via oscillators and filters.

#### 4. **Input Handling** (lines 767-861)
- Keyboard: Arrow keys + WASD for movement, Space to fire, X/Shift to bomb, P/Esc to pause
- Mouse: Move to steer, click to fire
- Touch: Left zone for joystick movement, right buttons for fire/bomb
- `joyVec`: Normalized joystick vector (x, y both in [-1, 1])
- Mouse input uses `mouseTarget` position when no keyboard/joystick input active

#### 5. **Game State** (lines 863-912)
Core game objects and tracking:
- `player`: Player aircraft object with position, speed, weapon, cooldown, health (lives), invulnerability timer
- `bullets`: Array of player bullets (position, velocity, damage properties)
- `enemies`: Array of spawned enemies (position, type, HP, shooting pattern, weaving behavior)
- `enemyBullets`: Array of projectiles fired by enemies and bosses
- `particles`: Explosion particles (position, velocity, lifetime)
- `powerUps`: Collectible items (position, type: 'life'/'bomb'/'weapon')
- `boss`: Current boss object (or null); tracks HP, attack pattern, telegraph state
- Score tracking: `score`, `chain`, `multiplier`, `stage`, `lives`, `bombCount`

#### 6. **Game Loop & Update** (lines 1148-1302)
Main `update(dt)` function runs each frame:
1. Player movement input handling (keyboard/joystick takes priority over mouse)
2. Weapon firing with cooldown
3. Bullet physics and filtering (remove off-screen)
4. Enemy spawning based on difficulty and frame count
5. Enemy behavior (movement, shooting, weaving)
6. Boss update if present
7. Collision detection (bullets vs enemies/boss, enemies vs player, enemy bullets vs player, power-ups vs player)
8. Particle animation
9. Boss health bar update

#### 7. **Rendering** (lines 1520-end)
Main `draw()` function:
- Background (sky gradient) with parallax clouds
- Player aircraft with weapon aura (laser only)
- All bullets, enemy bullets, particles
- Boss health bar and enemy/boss visuals
- HUD (score, multiplier, weapon level, hearts, bombs)

#### 8. **Boss System** (lines 1043-1439)
- `spawnBoss()`: Creates boss with HP scaled to difficulty and stage
- `updateBoss(dt)`: Handles boss movement, telegraph timings, and pattern execution
- `BOSS_PATTERNS`: Array of pattern functions; each boss type has unique patterns
  - Each pattern fires enemy bullets in distinct shapes (radial, spread, columns, spiral, homing, etc.)
  - Telegraph mechanic: Boss telegraphs attack 22 frames before firing; visual flash and SFX cue
- Boss patterns indexed per attack cycle; cycles through all patterns

#### 9. **Collision & Damage** (lines 1232-1294)
- `rectsOverlap(a, b)`: Simple AABB collision check
- Bullet-enemy collision: Reduces enemy HP, consumes bullet (unless laser with pierce > 0)
- Enemy-player collision: Triggers invulnerability timer, camera shake, damage
- Power-up collection: Upgrades weapon, restores life, adds bombs

#### 10. **Difficulty Ramping**
- Spawn rate decreases over time: `spawnTimer = Math.max(d.spawnMin, d.spawnBase - difficultyRamp)`
- Enemy type availability gated by stage: `minStage` property filters which enemies spawn
- Boss HP scales by difficulty multiplier (`d.bossHpMul`) and boss design multiplier

## Key Data Structures

### Player Object
```javascript
{
  x, y,              // Center position
  w, h,              // Collision bounds (40×40)
  speed,             // Movement speed (260 px/s)
  cooldown,          // Weapon fire cooldown
  weapon,            // 'vulcan' or 'laser'
  level,             // 1-3 for weapon upgrades
  invuln             // Invulnerability timer in seconds
}
```

### Bullet Object
```javascript
{
  x, y,              // Center position
  w, h,              // Bounds
  speed,             // Upward speed (500 vulcan, 640 laser)
  vx,                // Horizontal velocity (spread for vulcan)
  pierce,            // Remaining pierce count (laser only)
  laser              // Boolean: true for laser bullets
}
```

### Enemy Object
```javascript
{
  x, y,              // Center position
  w, h,              // Collision bounds
  type,              // 'grunt', 'interceptor', 'tank'
  hp,                // Current health
  speed,             // Downward speed (difficulty + type scaled)
  color, dark,       // Fill and shadow colors
  shootTimer,        // Countdown to next shot
  canShoot,          // Boolean: will this enemy shoot?
  pattern,           // 'aimed' or 'spread' bullet pattern
  weave,             // Boolean: sine-wave movement?
  weaveSeed,         // Random offset for weave phase
  baseX              // X position without weave
}
```

### Boss Object
```javascript
{
  x, y,              // Center
  w, h,              // Bounds (90×80)
  hp, maxHp,         // Health tracking
  design,            // Reference to BOSSES[i] for visuals
  entering,          // Boolean: in spawn animation?
  vx,                // Horizontal velocity (bounces at edges)
  telegraph,         // Countdown to next attack (frames)
  patternIndex,      // Which pattern in cycle
  attackTimer        // Countdown to telegraph trigger
}
```

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant with `w, h, hp, speedMul, color, dark, score, minStage`
2. Optional: Add explosion sound mapping to `EXPLODE_SFX`
3. Spawn logic in `spawnEnemy()` already uses `minStage` gating

### Adding a New Boss
1. Add design object to `BOSSES` array: `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Create attack pattern functions (see `fireRadial`, `fireAimedSpread`, etc.)
3. Add pattern array to `BOSS_PATTERNS` with same index as new boss in `BOSSES`

### Adjusting Game Balance
- **Difficulty**: Modify `DIFFICULTIES` object (spawn rates, enemy speed, shoot chance, lives, boss HP)
- **Enemy HP/Damage**: Modify `ENEMY_TYPES` (hp field) and `bullets.pierce` in weapon fire
- **Boss HP**: Controlled by `baseHp * d.bossHpMul * design.hpMul` in `spawnBoss()`
- **Spawn Rate**: `d.spawnBase` and `d.spawnMin` in `DIFFICULTIES`

### Adding Visual Effects
- Particles: Modify `spawnExplosion(x, y, color, count)` for size, color, lifetime
- Screen shake: Call `screenShake(magnitude, duration)` — automatically scaled by `reducedMotion` setting
- Flash effect: Call `flashHit()` or `flashBomb()` — adds CSS animation class
- Boss telegraph: Visual flash controlled by `b.telegraph > 0` in `drawBossShip()`

### Audio/Sound Effects
- Add new sound: Add function to `sfx` object in audio section
- Use `beep(freq, duration, type, volume, opts)` for tonal sounds
  - `type`: 'sine', 'square', 'sawtooth', 'triangle'
  - `opts`: `{ sweepTo, filterType, filterFreq, pan, delay }`
- Use `noiseBurst(duration, volume, opts)` for explosions/impacts
- Test with `muted` flag: Mute button (🔊) in HUD stores preference to localStorage

### Modifying Input Controls
- Keyboard handling: Modify event listeners in input section or `keys` map in `update()`
- Touch buttons: Edit `#fireBtn` and `#bombBtn` styles or behavior
- Mouse steering: Adjust `mouseTarget` lerp factor in player movement logic
- Joystick: Modify `joystickStart`, `joystickMove`, `joystickEnd` functions and `#joystickBase` styling

### Performance Tuning
- Adaptive quality: Game automatically reduces glow/shadow/particle effects (`quality` variable)
- Frame rate tracking: `trackPerf(dt)` logs frame times; quality adjusts if avg > 1/40s or < 1/56s
- Reduce particle count: Scale `spawnExplosion()` count parameter
- Disable effects: Set `reducedMotion = true` to skip screen shake, flash, vignette animations

### UI/HUD Updates
- Score display: Update `scoreEl.textContent` in `addScore()`
- Weapon display: `weaponNameEl` and `weaponLvlEl` update in `applyPowerUp()`
- Lives/bombs: Call `renderHearts()` and `renderBombs()` to regenerate heart/bomb visuals
- Boss bar: Controlled by `bossBarWrap.classList` and `bossBarFill.style.width`

## Testing Tips

### Run the Game
Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge).

### Test by Difficulty
Use HUD buttons (Easy/Normal/Hard) to change difficulty before starting. Settings persist to localStorage.

### Debug Game State
In browser DevTools console:
```javascript
console.log(player);    // Check player position, weapon, health
console.log(boss);      // Check boss HP, pattern index
console.log(enemies);   // Check enemy array size
```

### Audio Testing
Click 🔊 button to mute/unmute. Sounds are synthesized; no assets to load.

### Performance Profiling
Open DevTools Performance tab and record a ~10s gameplay session. Frame rate and rendering time visible.

### Reduced Motion Testing
Press 🎆 button to toggle reduced-motion mode. Disables screen shake, flash, and vignette animations.

## Browser Compatibility
- **Requires**: Canvas 2D, Web Audio API, localStorage, CSS animations, ES6+ features
- **Tested**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile**: Full touch support on iOS/Android; landscape orientation recommended

## File Structure
```
Google_4/
├── index.html          # Single-file game (HTML + CSS + JS)
├── README.md           # Project readme
└── CLAUDE.md           # This file
```

## Common Debugging

### Game won't start
- Ensure browser supports Canvas 2D and Web Audio API
- Check browser console for errors
- Try muting audio (click 🔊) in case audio context initialization fails

### Visuals tearing or performance drops
- Press 🎆 to toggle effects if needed
- Quality setting (`quality` var) auto-adjusts; check DevTools console: `quality`
- Check for excessive particle count during big explosions

### Boss patterns not firing
- Verify boss is not in `entering` state (should resolve after ~2 seconds)
- Check `BOSS_PATTERNS` array index matches boss index in `BOSSES`
- Ensure telegraph timer reaches 0 to trigger pattern: check `b.telegraph` value

### Touch controls not responding
- Check `isTouchDevice` is true: `console.log(isTouchDevice)`
- Ensure `touchControls` has `enabled` class: `touchControls.classList.contains('enabled')`
- Test on actual mobile device or use Chrome DevTools device emulation
