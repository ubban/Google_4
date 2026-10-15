# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, CSS, and JavaScript. The game runs in a single `index.html` file (1669 lines) with embedded styles and JavaScript. It features:

- Classic top-down scrolling shooter gameplay
- Progressive difficulty with multiple bosses and stages
- Weapon upgrade system with 4 weapon types
- Smart bomb mechanic for clearing the screen
- Touch and keyboard controls
- Responsive design (mobile and desktop)
- Accessibility features (reduced motion, screen reader support)

## File Structure

```
Google_4/
├── index.html          (Main game file - 1669 lines, contains all HTML, CSS, and JS)
├── README.md           (Project title and timestamp)
└── CLAUDE.md           (This documentation)
```

## Architecture Overview

### Single-File Application Pattern

The entire game is contained in `index.html`:
- **HTML (lines 1-420)**: DOM structure for game canvas, HUD, controls, and UI overlays
- **CSS (lines 10-359)**: Responsive styling, animations, and visual effects
- **JavaScript (lines 422-1669)**: Game logic and rendering

### Core Game Systems

#### 1. **Game State Management**
- `resetGame()` - Initializes game state (player, enemies, bosses, stage)
- `update(dt)` - Main game loop update function (called each frame)
- `draw()` - Renders all game elements to canvas

#### 2. **Rendering & Graphics**
- `drawPlane(x, y, w, h, color, darkColor, flame)` - Renders player ship
- `drawBossShip(b)` - Renders boss enemy
- `drawClouds(speed, fillColor, rx, ry)` - Scrolling background
- Canvas 2D context used for all graphics (no sprite sheets)

#### 3. **Audio System**
- `ensureAudio()` - Initializes Web Audio API context
- `beep(freq, dur, type, vol, opts)` - Synthesized sound effects
- `noiseBurst(dur, vol, opts)` - White noise effects for bombs
- `panFor(x)` - Stereo panning based on screen position
- Mute state stored in localStorage (`STORAGE_MUTE`)

#### 4. **Input Handling**
- **Keyboard**: Arrow keys/WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
- **Mouse**: Movement + click to fire
- **Touch**: Joystick on left, action buttons on right (Fire/Bomb buttons)
- Joystick implementation with visual feedback

#### 5. **Entity Management**
```javascript
Game entities tracked in arrays:
- player: Single player plane object
- enemies: Array of regular enemies
- bosses: Array of boss enemies (max 1 active per stage)
- playerBullets: Array of player projectiles
- enemyBullets: Array of enemy projectiles
- particles: Array of explosion effects
- powerUps: Array of collectable pickups
```

#### 6. **Weapon System** (4 weapons with upgrade levels)
- **VULCAN** - Rapid-fire straight shots
- **LASER** - Instant-hit beam weapon
- **SPREAD** - Multi-directional shots
- **HOMING** - Tracking projectiles

Firing patterns:
- `fireRadial(b, count, speed, offset)` - Circular burst pattern
- `fireAimedSpread(b, count, speed, spreadStep)` - Fan pattern
- `fireColumns(b, speed)` - Vertical columns
- `fireSpiralBurst(b, speed)` - Spiral pattern
- `fireCross(b, speed)` - Cross pattern
- `fireDoubleRing(b, speedA, speedB)` - Double ring pattern
- `fireHomingSeeded(b, count, speed)` - Homing missiles

#### 7. **Difficulty System**
```javascript
DIFFICULTIES = {
  easy: { enemy spawn rate, bullet patterns, boss HP },
  normal: { ... },
  hard: { ... }
}
```
- Selected at start screen
- Stored in localStorage (`STORAGE_DIFF`)
- Affects enemy behavior and damage

#### 8. **Score & Progression**
- Score multiplier based on combo (consecutive hits without taking damage)
- Multiplier resets on hit (`breakChain()`)
- Best score saved to localStorage (`STORAGE_BEST`)
- Stage progression: Complete all enemies on stage → boss appears
- Kill boss → advance to next stage with new enemy waves

#### 9. **Visual Effects**
- `screenShake(mag, dur)` - Camera shake on hit/bomb
- `flashHit()` - White flash on damage taken
- `flashBomb()` - Yellow flash on bomb detonation
- `showComboPop(text)` - Floating combo indicator
- `showStageBanner(text)` - Stage transition banner
- Red vignette animation when player health is critical

#### 10. **UI System**
- **HUD (always visible)**: Score, best score, multiplier, weapon, health, bombs
- **Overlay**: Title screen with difficulty selector and start button
- **Boss bar**: Shows boss health during boss fights
- **Screen reader support**: Aria live region for announcements

## Key Constants & Configuration

```javascript
STORAGE_BEST = 'skyfighter.bestScore'
STORAGE_MUTE = 'skyfighter.muted'
STORAGE_DIFF = 'skyfighter.difficulty'
STORAGE_REDUCED = 'skyfighter.reducedMotion'

DIFFICULTIES = {
  easy: { spawnInterval: 2.5, ... },
  normal: { spawnInterval: 1.8, ... },
  hard: { spawnInterval: 1.2, ... }
}

THEMES = [ /* Array of 5 color themes for enemy variety */ ]

BOSSES = [ /* Array of boss definitions with names and patterns */ ]
```

## Game Loop

```
Initialize Game
↓
Show Title Screen (difficulty selector)
↓
Game Loop (each frame):
  1. update(dt) - Process input, update entities, check collisions
  2. draw() - Clear canvas, render background, entities, HUD
  3. Check win/lose conditions
↓
Show Results Screen (with new best score notification)
```

## Performance Optimization

- **Adaptive quality system** (`trackPerf()`) - Reduces particle effects on low-end devices
- **Frame time tracking** - Maintains 45-frame rolling average
- **Quality levels**: 0=full, 1=medium, 2=low (affects particle count)
- **LocalStorage caching** - Best score, settings cached locally

## Controls & Accessibility

### Keyboard/Mouse Controls
- **Arrow keys/WASD** - Move plane
- **Space/X/Shift** - Fire/Bomb
- **P or Esc** - Pause/Resume
- **Mouse movement** - Steer (if using mouse mode)
- **Mouse click** - Fire

### Touch Controls
- **Left side drag** - Move plane (joystick)
- **Right side tap FIRE** - Fire weapon
- **Right side tap BOMB** - Detonate smart bomb

### Accessibility
- **Reduced motion mode** - Disables animations and screen effects
- **Screen reader support** - Lives in HUD, announcements in aria-live region
- **Color contrast** - High contrast UI and text
- **Focus management** - Proper focus on interactive elements

## Data Structures

### Player Object
```javascript
{
  x, y,              // Position
  w, h,              // Width/height
  vx, vy,            // Velocity
  health,            // Current HP
  maxHealth,         // Max HP
  bomb,              // Smart bomb count
  maxBomb,           // Max bombs
  weapon,            // Current weapon type (0-3)
  weaponLevel,       // Upgrade level (1-5)
  fireTimer,         // Cooldown for weapon
  invulnTimer        // Invulnerability frames after hit
}
```

### Enemy Object
```javascript
{
  x, y,              // Position
  vx, vy,            // Velocity
  health,            // Current HP
  fireTimer,         // Cooldown for weapon
  radius,            // Collision radius
  color,             // Visual color theme
  isBoss,            // If true, this is a boss
  bossName,          // Boss identifier string
  pattern,           // Attack pattern function
  progress           // Stage progress (0-1)
}
```

### Bullet Object
```javascript
{
  x, y,              // Position
  vx, vy,            // Velocity
  radius,            // Collision radius
  color,             // Visual color
  isPlayer,          // Owner (player or enemy)
  isPiercing,        // If true, passes through enemies
  lifeTimer,         // Remaining lifetime
  homingTarget       // For homing bullets
}
```

## Development Notes

### Adding New Features
1. **New weapon type**: Add to weapon firing functions, update `weaponLabel()` and `DIFFICULTIES`
2. **New boss**: Add to `BOSSES` array with firing pattern function
3. **New effect**: Add animation to CSS or implement drawing in `draw()`
4. **New difficulty**: Update `DIFFICULTIES` object

### Testing
- Test across browsers (Chrome, Firefox, Safari)
- Test touch controls on mobile devices
- Test keyboard accessibility with screen readers
- Verify audio context initialization (some browsers require user interaction first)
- Test localStorage availability and quota

### Known Limitations
- Single canvas element (2D context, not WebGL)
- All assets are procedurally generated (no external image files)
- Audio is synthesized (no audio files, uses Web Audio API)
- No server-side features (pure client-side game)

## Browser Compatibility

Requires:
- HTML5 Canvas API
- Web Audio API (for sound)
- ES6+ JavaScript (arrow functions, template literals, const/let)
- localStorage API

Tested on:
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+

## Local Development

1. No build step required - open `index.html` in a browser
2. For local testing: `python3 -m http.server 8000` (avoids CORS issues)
3. Developer tools: Use browser DevTools to inspect canvas and Web Audio
4. Disable cache while developing (DevTools → Settings → Disable cache)

## Future Enhancement Ideas

- Score leaderboard with cloud sync
- Customizable control schemes
- Additional weapon types and power-ups
- Persistent progression/campaign mode
- Sound volume controls
- Mobile app wrapper (Electron, Cordova)
- Multiplayer mode
- Level editor

## Attribution & License

Project: Sky Fighter
Created: 2026-02-23
