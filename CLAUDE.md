# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shooter game built entirely in a single HTML file (`index.html`) with embedded CSS and JavaScript. It features:
- Canvas-based 2D rendering
- Synthesized audio (WebAudio API, no external assets)
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive enemy spawning with 6 boss types
- Weapon upgrade system
- Bomb mechanics
- Touch and keyboard controls
- Persistent score tracking via localStorage
- Accessibility features (screen reader announcements, reduced motion support)

## Running the Game

### Local Development
```bash
# Serve the file directly with any HTTP server
python3 -m http.server 8000
# or
npx http-server
# or
python -m SimpleHTTPServer 8000  # Python 2
```
Then open `http://localhost:8000` in a browser.

### Production
The game is a static HTML file that can be served directly from any CDN or static host. No build step is required.

## Code Architecture

### Single-File Structure
The entire game logic is in `index.html`:
- **Lines 1-359**: CSS (styling for UI, canvas, overlays, animations)
- **Lines 360-420**: HTML markup (canvas, HUD, controls, overlay menu)
- **Lines 422+**: JavaScript game engine (wrapped in IIFE, strict mode)

### Game Loop & State Management
```javascript
(function () { 'use strict'; ... })()
```
The game runs in an IIFE with strict mode. The game state is maintained in module-scoped variables:
- `gameActive` - whether game is currently running
- `paused` - pause state
- `player` - player object (position, health, weapon, bombs, score)
- `enemies` - array of active enemy objects
- `bullets` - arrays for player bullets and enemy bullets
- `explosions` - array of active explosion particles

### Key Game Subsystems

#### 1. Rendering Pipeline
- **Canvas setup**: 2D context with `antialias` disabled for crispness
- **Draw order**: Background gradient → enemies → bosses → player → effects (explosions, flashes, vignette)
- **Performance adaptation**: `quality` variable (0-2) adjusts particle counts and effect detail based on frame time tracking

#### 2. Game Loop (`gameLoop()` function)
Called via `requestAnimationFrame`:
1. Calculate delta time
2. Update player position (from input)
3. Update enemy positions & fire
4. Update all bullets
5. Update boss state
6. Handle collisions (bullets vs enemies, player vs bullets, enemy vs player)
7. Spawn new enemies (based on spawn timer)
8. Render frame
9. Track performance metrics

#### 3. Input Handling
Three input modes:
- **Keyboard**: Arrow keys, WASD for movement; Space/X/Shift for fire/bomb; P/Esc for pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Left zone for movement (joystick visual), right zone for fire/bomb buttons

Input state is accumulated in an object, checked each frame to avoid input lag.

#### 4. Collision Detection
- **AABB (Axis-Aligned Bounding Box)**: All collision checks use simple rectangle intersection
- **Player bullets vs enemies**: Deals damage, triggers explosion effects
- **Enemy bullets vs player**: Reduces health, triggers damage flash/vignette
- **Enemy proximity to player**: Immediate damage

#### 5. Difficulty System
Three preset configurations in `DIFFICULTIES` object:
- **Easy**: Slower spawns, slower/weaker enemies, 4 lives
- **Normal**: Medium intensity, 3 lives (default)
- **Hard**: Fast spawns, aggressive enemies, 1.3x boss HP

Each config controls: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

#### 6. Enemy & Boss System
- **Enemy types** (in `ENEMY_TYPES`): Grunt, Interceptor, Tank
- **Boss types** (in `BOSSES`): 6 unique bosses with different visual themes and HP multipliers
- **Spawning**: Progressive increase in difficulty based on kills-per-stage counter; bosses appear every 18 kills
- **Boss rendering**: Custom drawn with gradient fills (hull, wings, core)

#### 7. Audio System
Web Audio API synthesis with no external files:
- **Beep function**: Oscillator-based tones with optional frequency sweep, filtering, panning, ADSR envelope
- **Noise bursts**: Filtered white noise for bomb/impact sounds
- **Muting**: Controlled via `muted` flag stored in localStorage

#### 8. Visual Polish
- **Animations**: CSS keyframe animations for card entrance, combo popups, stage banners, heart beat pulse
- **Effects**:
  - Flash overlay on hit/bomb
  - Red vignette pulse during damage
  - Combo counter display
  - Stage banner display
  - Boss health bar with smooth transitions
- **Themes**: 6 different sky gradients and color palettes selectable by stage

#### 9. Persistence & Preferences
All stored in browser localStorage:
- `skyfighter.bestScore` - high score
- `skyfighter.muted` - audio mute state
- `skyfighter.difficulty` - last selected difficulty
- `skyfighter.reducedMotion` - reduced motion preference

#### 10. Accessibility
- Screen reader announcements via `announce()` function and hidden `#srLive` element
- Reduced motion support: Disables all animations when `prefers-reduced-motion` is set or manually toggled
- Semantic HTML with ARIA labels
- High contrast UI elements

## Adding New Content

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.3, color: '#xxxx', dark: '#xxxx', score: 25, minStage: 2 }
   ```
2. Modify `spawnEnemy()` to include new type in random selection (check `Math.random()` branches)
3. Adjust `minStage` so new type appears only in later stages

### Adding a New Boss
1. Add entry to `BOSSES` array (line ~508):
   ```javascript
   { name: 'NEWBOSS', hullTop: '#xxx', hullBottom: '#xxx', wing: '#xxx', core: '#xxx', hpMul: 1.2 }
   ```
2. Boss is drawn by `drawBoss()` function - modify that function if custom rendering needed
3. Boss cycles automatically through array; no other changes needed

### Adding a New Weapon
Look for weapon upgrade system (search for `VULCAN`, `LASER`, etc.):
1. Add weapon config to data structure
2. Modify firing logic in game loop to handle new weapon spread/fire rate
3. Update HUD display (`weaponNameEl`, `weaponLvlEl`)

### Adding New Visual Theme
1. Add to `THEMES` array (line ~493):
   ```javascript
   { sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
2. Themes cycle automatically; background gradient is applied at start of each stage

## Performance Considerations

### Adaptive Quality System
Frame time is tracked (`frameTimes` array). If average frame time exceeds ~25ms (40 FPS), quality is reduced:
- `quality = 0`: Full details (all particles, effects)
- `quality = 1`: Medium (reduced particle counts)
- `quality = 2`: Low (minimal visual effects)

Check `quality` variable before expensive rendering operations.

### Optimization Techniques Used
- **RequestAnimationFrame**: Syncs to display refresh rate
- **Antialiasing disabled**: Crisp pixel-art style rendering
- **Lazy audio initialization**: WebAudio context created on first sound play
- **Sparse collision checks**: Only active entities (no quadtree, but manageable entity counts)
- **Canvas clearing**: Single `fillRect` per frame instead of `clearRect`

### Browser Compatibility
- Modern browsers with support for:
  - HTML5 Canvas 2D
  - Web Audio API (or webkit prefix fallback)
  - LocalStorage
  - `requestAnimationFrame`
  - CSS3 animations
  - CSS Grid/Flexbox

Touch support requires: Pointer Events or Touch Events API (widely supported).

## Common Tasks

### Running the game locally
```bash
python3 -m http.server 8000
# Visit http://localhost:8000
```

### Testing a specific difficulty
- Open dev console, set:
  ```javascript
  localStorage.setItem('skyfighter.difficulty', 'hard');
  location.reload();
  ```

### Tweaking game balance
Edit the `DIFFICULTIES` object constants:
- `spawnBase` / `spawnMin`: Enemy spawn frequency
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemies fire each frame
- `lives` / `bombs`: Starting count
- `bossHpMul`: Boss HP scaling

### Clearing saved data
```javascript
localStorage.clear();
location.reload();
```

### Adding audio muting toggle
Audio muting is already implemented - check `muteBtn` click handler and `muted` flag.

## Code Style & Conventions

- **Strict mode**: Enforced via IIFE wrapper
- **Variable naming**: Camelcase for variables/functions, SCREAMING_SNAKE for constants
- **DOM queries**: Stored in variables (e.g., `const canvas = document.getElementById(...)`)
- **No external dependencies**: Pure vanilla JavaScript
- **Comments**: Minimal; code uses clear naming (e.g., `updateMuteBtn()`, `drawEnemy()`)
- **Magic numbers**: Where used, grouped near definitions (e.g., `DIFFICULTIES` object, `KILLS_PER_STAGE`)

## Debugging Tips

### Enable/disable reduced motion in console
```javascript
reducedMotion = !reducedMotion;
localStorage.setItem('skyfighter.reducedMotion', reducedMotion ? '1' : '0');
updateMotionBtn();
```

### Inspect enemy state
```javascript
console.log('Enemies:', enemies);
console.log('Player:', player);
```

### Check current game state
```javascript
console.log('GameActive:', gameActive, 'Paused:', paused, 'Quality:', quality);
```

### Monitor frame times
```javascript
console.log('Avg frame time:', frameTimes.reduce((a,b) => a+b)/frameTimes.length);
```

## Future Enhancement Ideas

- Progressive wave-based enemy patterns (vs. random spawning)
- Power-up drops with visual feedback
- Local leaderboard with replay capability
- Screen shake intensity based on difficulty
- Weapon combo chains for high-multiplier bonuses
- Boss attack patterns with telegraph animations
- Background parallax scrolling
- Procedural theme generation
