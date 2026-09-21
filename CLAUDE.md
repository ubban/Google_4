# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 Canvas application. The entire game—including HTML, CSS, and JavaScript—is contained in `index.html`.

### Key Technologies
- **HTML5 Canvas** for rendering
- **Web Audio API** for procedurally generated sound effects (no external audio files)
- **LocalStorage** for persistence (best score, settings)
- **Touch/Joystick support** for mobile play
- **Responsive design** with adaptive quality scaling based on frame time

## Development Commands

Since this is a standalone HTML file, most development is done directly in the editor:

### Running the Game
- **Browser**: Open `index.html` directly in any modern web browser
- **Local Server** (optional, for HTTPS-dependent features):
  ```bash
  # Python 3
  python -m http.server 8000
  # Then visit http://localhost:8000
  
  # Node.js (if available)
  npx http-server
  ```

### Testing
- No automated test suite exists; test manually by:
  - Playing through all three difficulty levels (Easy, Normal, Hard)
  - Testing touch controls on mobile or using browser dev tools
  - Verifying each weapon type and bomb effects
  - Confirming all audio effects play correctly
  - Checking LocalStorage persistence across page reloads

### Code Quality
- **No linting/build step**: Edit `index.html` directly
- **Browser DevTools Console**: Check for runtime errors
- **Performance**: Open DevTools → Performance tab to monitor frame times; the game has built-in adaptive quality that degrades if avg frame time > 1/40s

## Architecture & Code Structure

The `index.html` file follows this logical organization (line numbers approximate):

### 1. **DOM Setup & Constants** (lines 1–421)
- HTML markup: game canvas, HUD, pause/game-over overlays, touch controls, mobile buttons
- CSS styling: responsive layout, animations (heartBeat, vignettePulse, flashHit), dark theme
- Constant definitions: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, storage keys

### 2. **Game Initialization** (lines 422–750)
- IIFE wrapper to isolate scope
- DOM element caching
- Audio context setup with Web Audio API
- Mute/motion preferences from LocalStorage
- Difficulty selector and settings

### 3. **Audio System** (lines 542–676)
- `beep(freq, dur, type, vol, opts)`: Core oscillator synth
- `noiseBurst(dur, vol, opts)`: Noise buffer for percussion
- `sfx` object: Pre-tuned sound effects (shoot, laser, explode, bomb, powerup, etc.)
- **No external audio files**; all generated procedurally at runtime
- Panning support based on projectile/explosion X coordinate

### 4. **Input Handling** (lines 785–870)
- Keyboard input: Arrow keys (move), Space (fire), B (bomb), P (pause), Shift (reduce motion)
- Touch/Mobile: Virtual joystick with fire and bomb buttons
- Desktop: Mouse position tracked for aiming (cross-hair cursor)
- `joystickStart()`, `joystickMove()`, `joystickEnd()`: Touch input management

### 5. **Game State & Update Loop** (lines 875–1470)
- **Core state**: Player position/weapon/health, enemy array, projectiles, explosions, score, combo counter, boss state
- `resetGame()`: Initialize new run
- `update(dt)`: Main physics and logic update (lines 1148–1302)
  - Player movement and weapon firing
  - Enemy spawning (adaptive per difficulty)
  - Collision detection: bullets↔enemies, player↔enemies, projectiles↔player
  - Powerup application (weapons, lives, bombs)
  - Boss AI and attacks
  - Score combo multiplier (multiplier increases with consecutive kills)
- `fireWeapon()`: Player weapon dispatch based on current weapon type
- **Boss patterns** (lines 1324–1367): Multiple firing strategies
  - `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- `applyPowerUp(type)`: Weapon/health/bomb powerup logic
- `updateBoss(dt)`: Boss-specific AI and phase transitions

### 6. **Rendering Pipeline** (lines 1507–1635)
- `draw()`: Main render function
  - Cloud parallax background (adaptive speed per quality level)
  - Canvas clearing and transform for screen shake
  - Enemy rendering with shadows
  - Boss rendering (multi-part ship design)
  - Projectile rendering (bullets, bombs)
  - Particle effects (explosions)
  - HUD overlay (score, lives, weapon, bomb count)
  - Pause/game-over screens
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Player and enemy plane sprite (lines 913–968)
- `drawBossShip(b)`: Complex boss ship design with wings, core, and visual states (lines 970–1020)
- `drawClouds()`: Parallax cloud rendering with adaptive density

### 7. **Main Game Loop** (lines 1640–1655)
- `loop(now)`: RequestAnimationFrame callback
  - Calculates delta time
  - Calls `update(dt)` and `draw()`
  - Tracks performance for adaptive quality
  - Handles pause state

### 8. **UI/UX Features** (lines 1491–1505 & spread)
- `togglePause()`: Pause/resume game
- `endGame()`: Game over screen with final score/best score display
- `showStageBanner()`, `showComboPop()`: Feedback animations
- `flashHit()`, `flashBomb()`: Visual feedback (respects reduced-motion preference)
- `screenShake()`: Juice effect for impacts (disabled with reduced-motion)
- Accessibility: Live region announcements (`srLive`) for screen readers

## Key Gameplay Systems

### Difficulty Levels
Three predefined difficulty profiles in `DIFFICULTIES` object:
- **Easy**: Slower spawn rate, weaker enemies, more lives/bombs
- **Normal**: Balanced middle ground
- **Hard**: Aggressive spawn, higher enemy damage/speed, fewer lives

### Enemy Types
- **Grunt** (`#e53935`): Basic enemy, low health
- **Interceptor** (`#ff7043`): Fast, weaving movement
- **Tank** (`#5e35b1`): Slow, high health, appears from stage 2

### Boss System
Six named bosses with unique visual designs and `hpMul` (health multiplier):
- Color schemes (hullTop, hullBottom, wing, core) define appearance
- Each boss appears at the end of a stage
- Defeated by accumulating damage; triggers stage clear on defeat

### Weapon System
Multiple weapon types available as pickups; each has unique firing pattern:
- Standard weapon (default)
- Laser (fast, narrow)
- Spreadshot (wide, multiple projectiles)
- Dual cannons, missiles, etc. (defined in powerup logic)

### Combo Multiplier
- Increases for consecutive enemy kills without breaking
- Breaking condition: taking damage, missing for too long
- Displayed in HUD; affects score calculation

### Adaptive Quality System
- Monitors frame time over 45-frame window
- Adjusts `quality` level (0=full, 1=medium, 2=low) based on avg frame time
- Low quality disables particle count, reduces projectile rendering detail
- Keeps gameplay smooth on lower-end devices

## Performance & Optimization

1. **Canvas rendering**: Adaptive particle count based on quality level
2. **Audio**: Lazy-loads Web Audio context on first interaction
3. **Collision**: Simple AABB (rect overlap) checks; no spatial hashing (scale appropriate for single Canvas game)
4. **DOM**: Minimal DOM manipulation; most rendering on Canvas
5. **LocalStorage**: Used only for persistence, not per-frame

## Common Modification Points

When adding features, these are typical edit locations:

- **New enemy type**: Add to `ENEMY_TYPES` object (lines ~502–506), then update `spawnEnemy()` and collision logic
- **New boss**: Add to `BOSSES` array (lines ~508–515), add custom `boss.patternX()` firing function, update `updateBoss()` dispatch
- **New weapon**: Add firing pattern function (e.g., `fireNewPattern()`), dispatch from `fireWeapon()`, add sprite/color definition
- **Audio tweak**: Modify sfx object entries or beep/noiseBurst calls with different freq/duration/filter parameters
- **UI element**: Add HTML in the head section, add CSS rule, reference in DOM cache section
- **Game balance**: Adjust `DIFFICULTIES` object, `KILLS_PER_STAGE`, enemy HP/speed values, boss hpMul

## Mobile & Accessibility

- **Touch controls**: Virtual joystick in moveZone, fire/bomb buttons
- **Screen reader**: Live region (`srLive`) announces game state changes
- **Reduced motion**: Checkbox respects `prefers-reduced-motion` OS preference; disables shake/flash effects
- **Responsive canvas**: Scales to fit viewport while maintaining aspect ratio

## Browser Compatibility

- Requires: HTML5 Canvas, RequestAnimationFrame, Web Audio API, ES6 (arrow functions, const/let, template literals)
- Modern browsers (Chrome, Firefox, Safari, Edge) fully supported
- Mobile: iOS Safari, Chrome Android with touch event handling

## File Overview

- **`index.html`**: Single-file application (1669 lines)
  - Lines 1–421: HTML + CSS + Constants
  - Lines 422–1655: JavaScript game logic, rendering, audio, input
  - No external dependencies or build step

## Debugging Tips

1. **Check console errors**: Open DevTools → Console for runtime issues
2. **Visual debug**: Temporarily add `ctx.strokeRect()` calls to visualize collision boxes
3. **Audio issues**: Verify `audioCtx.state` in DevTools console; Web Audio must be resumed after user gesture
4. **Performance**: DevTools → Performance tab; record a play session and inspect frame times
5. **Input issues**: Check browser console for touch/keyboard events; ensure keyboard event listeners are attached to `window`
6. **LocalStorage**: DevTools → Application → LocalStorage to inspect saved best score and settings

## Testing Checklist

When making changes, manually verify:
- [ ] Game starts and canvas renders
- [ ] Player moves smoothly (keyboard + touch)
- [ ] Firing works and bullets move
- [ ] Enemies spawn and move correctly
- [ ] Collisions (player with enemy, bullet with enemy) register
- [ ] Scoring updates and combo multiplier works
- [ ] Powerups are collected and effects apply
- [ ] Boss appears and fights correctly
- [ ] Boss defeat ends stage and shows banner
- [ ] Game over screen shows correctly
- [ ] Best score is saved and persists across reloads
- [ ] Mute/motion settings save and apply
- [ ] Touch controls work on mobile device or DevTools device emulation
- [ ] Audio plays (or stays muted if toggled)
