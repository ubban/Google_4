# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Procedurally spawned waves of enemies across multiple stages
- Six unique boss encounters with distinct attack patterns
- Multiple weapon types with upgrade levels
- Three difficulty modes with adjustable parameters
- Procedural audio synthesis (no external sound files)
- Responsive canvas rendering with adaptive performance
- Touch controls and keyboard input support
- Combo multiplier system for consecutive hits

## Architecture

### Single-File Structure
The entire game is contained in `index.html` (1669 lines):
- Lines 1-421: HTML markup and embedded CSS
- Lines 422-1666: JavaScript game logic in IIFE scope

### Core Game Systems

#### 1. Game State Management
Key global variables for game state:
- `running`, `paused`, `gameOver`: Game control flags
- `score`, `bestScore`, `multiplier`: Player metrics
- `lives`, `bombCount`, `maxBombs`: Player resources
- `stage`, `kills`, `stageKills`: Progression tracking
- `difficulty`: Current difficulty setting (easy/normal/hard)
- `player`: Player ship object with `{x, y, weapon, level, invuln}`

#### 2. Entity Systems
- **Player**: Single controllable ship with weapon state
- **Enemies**: Array of enemy objects spawned per stage, each with `{x, y, type, hp, speed, pattern, canShoot}`
- **Boss**: Single boss per stage after 18 enemy kills, with telegraphing and attack patterns
- **Bullets**: Separate arrays for `bullets` (player) and `enemyBullets` (enemy/boss)

#### 3. Rendering Pipeline
```
draw() {
  - Clear canvas with theme-based sky gradient
  - Draw clouds and sun glow
  - Draw player ship (with weapon flame effect)
  - Draw enemies and bullets
  - Draw boss and boss health bar
  - Apply screen shake offset
  - Render UI overlays (HUD, combos, stage banners)
}
```

#### 4. Update Cycle
Main game loop (`update(dt)`) in frame-rate-independent delta time:
- Update player position/weapon state from input
- Spawn new enemies based on difficulty curve
- Update enemy positions and AI (targeting, weaving)
- Handle collision detection (bullets vs entities)
- Update visual effects (shake, flashes, explosions)
- Boss behavior when stage progress reaches threshold

#### 5. Audio System
Procedural sound synthesis using Web Audio API:
- `audioCtx`: Web Audio context (lazy-initialized on first game)
- `masterGain`: Master volume with compressor
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with sweeps and filters
- `noiseBurst(dur, vol, opts)`: White noise for explosion effects
- Pre-composed SFX library in `sfx` object (shoot, laser, explode, hit, bomb, etc.)
- No external audio files; all synthesis inline

#### 6. Difficulty Configuration
Three presets in `DIFFICULTIES`:
```javascript
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```
These control spawn rate curves, enemy behavior, and boss scaling.

#### 7. Themes & Visuals
Six themes in `THEMES` array, each defining:
- Sky gradient colors (3 stops)
- Cloud color
- Sun color
- Display name
Themes cycle through stages automatically.

#### 8. Enemy & Boss Definitions
- **Enemy Types** in `ENEMY_TYPES`: grunt, interceptor, tank with varying HP, speed, patterns
- **Bosses** in `BOSSES`: 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique color schemes and HP scaling
- **Boss Patterns** in `BOSS_PATTERNS`: Array of 6 pattern sets, one per boss. Each pattern uses firing functions like `fireRadial()`, `fireColumns()`, `fireHomingSeeded()`, etc.

#### 9. Weapon System
Three weapons with level upgrades:
- `bullet`: Straight-fire basic weapon
- `laser`: Rapid-fire narrow beam (weapon level affects fire rate)
- `missile`: Homing projectiles
Weapon data: `{name, firingLogic, upgradeEffect}`

#### 10. Persistence & Settings
Local storage keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility toggle (respects OS prefers-reduced-motion)

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w, h, hp, speedMul, color, dark, score, minStage, weave`
2. Update `spawnEnemy()` to include the new type in `choices.includes()` check
3. Assign the SFX type in `EXPLODE_SFX` if it should use a custom explosion sound

### Adding a New Boss
1. Add object to `BOSSES` array with: `name, hullTop, hullBottom, wing, core, hpMul`
2. Create a new pattern set in `BOSS_PATTERNS` array (7 patterns match the 7th boss, cycling through old designs if more added)
3. Define attack patterns using helper functions: `fireRadial()`, `fireColumns()`, `fireAimedSpread()`, `fireDoubleRing()`, `fireHomingSeeded()`

### Adding a New Theme
1. Add object to `THEMES` with: `sky` (3-color array), `cloud` (RGB string), `sun` (RGB string), `name`
2. Themes cycle automatically; stage N uses `THEMES[(stage - 1) % THEMES.length]`

### Adjusting Difficulty
Modify `DIFFICULTIES` object properties:
- `spawnBase` / `spawnMin`: Lower = more aggressive spawn rate
- `enemySpeed`: Multiplier on enemy velocity
- `enemyShootChance`: Probability enemy fires (0.0-1.0)
- `lives` / `bombs`: Player starting resources
- `bossHpMul`: Multiplier on boss HP calculation

### Audio & Effects
- Sound effects live in `sfx` object; edit `beep()` and `noiseBurst()` calls to adjust pitch, duration, filter
- Screen shake: Call `screenShake(magnitude, duration)` in collision or impact code
- Flash effects: Call `flashHit()` or `flashBomb()` for visual feedback

### UI Text & Messaging
- HUD updates through `textContent` of DOM elements (score, weapon name, etc.)
- Screen reader announcements: `announce(text)` updates `#srLive` for accessibility

## Running & Development

Since this is a single-file HTML5 game with no build step:

### Testing
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Test on desktop with keyboard (`Arrow Keys` for movement, `Z` for fire, `X` for bomb)
3. Test on mobile by opening in browser; touch controls appear automatically

### Performance Monitoring
The game includes adaptive quality adjustment:
- `quality` variable tracks frame rate over 45-frame window
- If FPS drops below 40, quality increases (fewer particles, less shadow blur)
- If FPS exceeds 56, quality decreases (more visual detail)
- Adjust thresholds in `trackPerf()` function

### Accessibility
- **Reduced Motion**: Toggle with motion button; respects OS `prefers-reduced-motion` setting
- **Sound Mute**: Persistent toggle; all audio can be disabled
- **Screen Reader**: Live region `#srLive` announces game state changes
- **Difficulty Selector**: Three presets before game start; locked during play

### Key Canvas Constants
- `BASE_W = 480, BASE_H = 720`: Game canvas dimensions (4:6 aspect ratio)
- Scaling respects device size up to 1.6× magnification
- Safe area insets handled for notched displays

## Code Organization Patterns

### Common Event Listeners
- Keyboard/touch: `keys` object tracks pressed state; polled in `update()`
- Mouse/touch movement: `moveZone` drag handler updates joystick position
- Buttons: Click handlers for mute, motion, difficulty, start, pause, resume

### Math & Geometry
- Vector operations: Inline calculations for distance, angle, lerp
- Collision: Rectangle overlap (AABB) for bullets vs entities
- Targeting: Boss enemy-bullet spread uses angle-based firing

### State Transitions
- **Game Start**: `resetGame()` initializes all entities and UI
- **Pause**: `togglePause()` shows/hides overlay without resetting
- **Stage Clear**: Boss kill triggers `killBoss()`, increments stage, spawns new boss on next threshold
- **Game Over**: `endGame()` stops loop, saves high score, shows final stats overlay

## Common Edits & Extensions

### Changing Stage Progression
- `KILLS_PER_STAGE = 18`: Enemy kills before boss appears
- Modify `if (kills >= KILLS_PER_STAGE && !boss)` in `update()` to change trigger

### Custom Particle Effects
- Explosions use `spawnExplosion(x, y, color, count)`: Creates radial burst of circles
- Add new effect functions following the same pattern (object with position, velocity, lifetime)

### Weapon Rebalancing
- Fire rate, damage, spread: Controlled via `firingLogic` in weapon objects and bullet generation in `update()`
- Weapon level scaling: Edit coefficient in weapon upgrade logic

## Performance Notes

- **Canvas rendering**: No off-screen buffer; direct draw to main canvas
- **Collision detection**: O(n·m) where n=enemies, m=bullets; optimize by spatial hashing if entities exceed ~200
- **Audio**: Web Audio nodes created per sound; no pooling, but brevity of SFX mitigates GC pressure
- **Animation**: Frame counter `frame` increments every loop; used for wave, pulse, rotation animations
- **Reduced motion**: Affects animation duration, shake magnitude, and flash opacity—does not disable effects entirely

## Browser Compatibility

- Modern ES6+ (const/let, arrow functions, template literals)
- Canvas 2D context
- Web Audio API (with webkit fallback)
- LocalStorage
- CSS Grid/Flexbox for UI
- Supports both `keydown/keyup` and `touchstart/touchend/touchmove` input
