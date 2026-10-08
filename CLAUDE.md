# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with JavaScript. The entire game logic, UI, rendering, and audio synthesis lives in `index.html` (~1669 lines).

## Development Setup

### Running Locally

Since this is a static HTML file, you need a local HTTP server:

```bash
# Python 3
python3 -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Node.js (if installed)
npx http-server
```

Then open `http://localhost:8000` in a browser. The game will launch immediately on page load, and state (best score, difficulty, audio mute) persists in localStorage.

### Testing Changes

- Hard refresh (Ctrl+Shift+R or Cmd+Shift+R) to clear the browser cache after edits
- Use browser DevTools console to debug (F12)
- Test on mobile devices by navigating to your computer's IP (e.g., `http://192.168.x.x:8000`)

## Code Architecture

All game logic is wrapped in a single IIFE (Immediately Invoked Function Expression) starting at line 423. Key systems:

### Game State & Loop (lines 422–1120)
- Central `gameState` object holds all game data: player position/health, enemies, projectiles, score, stage
- `tick(dt)` is the main update loop (called every frame)
- `render()` draws all entities and UI to the canvas

### Player System (lines ~800–950)
- **Player** object: position, velocity, health, bombs, weapon level
- Input handling: keyboard (WASD/Arrows), mouse, and touch controls
- Movement clamped to canvas bounds
- Weapon switching via E/Q keys or number keys

### Enemy & Projectile System (lines ~1000–1250)
- **Enemies** array: each enemy has type (grunt/interceptor/tank), position, velocity, hp, target behavior
- **Projectiles** array: player bullets and enemy fire
- Collision detection: axis-aligned bounding boxes (AABB)
- Enemy spawning based on difficulty (spawn rate increases each stage)

### Boss System (lines ~1300–1450)
- Bosses spawn every 18 kills per stage
- Boss object: multi-phase behavior, special movement patterns, color-coded hull/wing/core
- Boss state tracked separately; defeated bosses advance to next stage
- Boss health bar displayed in top HUD

### Weapon System (lines ~550–750)
- **WEAPONS** object defines Vulcan, Spread, Missile, Laser with unique fire patterns
- Weapon level increases with pickups; affects damage, fire rate, projectile count
- Power-up drops randomly from killed enemies

### Difficulty & Configuration (lines ~485–515)
- **DIFFICULTIES** object: spawn rates, enemy speed, shoot chance, starting lives/bombs, HP multiplier
- **ENEMY_TYPES**: defines grunt, interceptor, tank stats (size, hp, speed, score)
- **BOSSES**: 6 boss patterns with unique colors and HP multipliers
- **THEMES**: 6 visual themes (sky gradients, cloud/sun colors) for stage variety

### Audio System (lines ~542–750)
- WebAudio synth (no external audio files) — all sounds generated procedurally
- `playSound(type, volume)` plays effects: bullet, explosion, select, damage, etc.
- Master gain node; mute state persists in localStorage

### UI & HUD (lines ~1500–1650)
- Canvas rendering for game elements (player, enemies, particles)
- DOM elements for stats HUD: score, best, multiplier, health, bombs, weapon
- Overlay system: title screen, game-over screen, difficulty select
- Accessibility: screen-reader announcements via `#srLive` element

### Input Handling (lines ~1100–1200)
- **Keyboard**: WASD/Arrows to move, Space/X/Shift to fire/bomb, P/Esc to pause, E/Q for weapon switch
- **Mouse**: click to fire, move to steer
- **Touch**: drag left half to move, tap buttons for fire/bomb (on mobile)
- Joystick base shown on touch devices for visual feedback

## Making Changes

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line ~504):
   ```javascript
   speedy: { w: 20, h: 20, hp: 1, speedMul: 2.5, color: '#00ff00', dark: '#00aa00', score: 20, minStage: 3 }
   ```
2. Set `minStage` to control when it first appears
3. Enemy spawning automatically includes all types in the pool (line ~1050)

### Adjusting Difficulty

Edit the `DIFFICULTIES` object (line ~485):
- `spawnBase` / `spawnMin`: controls spawn frequency (lower = more enemies)
- `enemySpeed`: multiplier for all enemy velocity
- `enemyShootChance`: 0–1 probability each frame an enemy fires
- `lives` / `bombs`: starting player resources
- `bossHpMul`: multiplier for boss HP

### Adding a Visual Theme

1. Add to `THEMES` (line ~493):
   ```javascript
   { sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
2. Themes cycle through stages (modulo 6); see line ~1400

### Adding a New Boss

1. Add to `BOSSES` (line ~508):
   ```javascript
   { name: 'BOSS_NAME', hullTop: '#color1', hullBottom: '#color2', wing: '#color3', core: '#color4', hpMul: 1.5 }
   ```
2. hpMul controls boss difficulty relative to normal
3. Boss rendering uses these colors (line ~1550)
4. Boss attack patterns are hardcoded in `updateBoss()` (line ~1380)

### Modifying Player/Weapon Balance

- Player speed: `PLAYER_SPEED` (line ~600)
- Player size (collision): `playerSize` (line ~620)
- Weapon fire rate: each weapon object has `rate` field (line ~660)
- Weapon damage/spread: modify projectile arrays in weapon objects

## Performance Notes

- **Adaptive quality** (line ~474): performance tracking drops particle effects / render resolution on low-end devices
- **Canvas size** scales to window; see `resizeCanvas()` (line ~1650)
- **Frame cap**: game runs at native refresh rate (60fps typical)
- **Collision checks**: O(n²) between enemies and projectiles; acceptable for ~50 entities on-screen

## Browser Compatibility

- Requires HTML5 Canvas 2D context
- Uses localStorage (persists state)
- Uses WebAudio API (audio synthesis)
- Tested on Chrome, Firefox, Safari, mobile browsers
- Mobile: touch events + DeviceOrientation API optional

## Debugging Tips

- `console.log(gameState)` in DevTools to inspect all game data at pause
- Set `quality = 0` to force full rendering quality
- Comment out `tick()` in RAF loop to freeze the game for inspection
- Enemy spawn threshold: change `KILLS_PER_STAGE` (line ~491) to test boss encounters
- Add `if (gameState.player.debug) { /* draw hitboxes */ }` to visualize collision boxes

## Accessibility

- Screen-reader announcements: `announce('text')` updates `#srLive` aria-live region
- Reduced motion preference respected: see `updateMotionBtn()` (line ~535)
- Keyboard controls fully support mouse-free play
- High contrast: colors chosen for visibility in daylight + dark
