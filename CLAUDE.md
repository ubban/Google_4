# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML file with embedded JavaScript. Players switch weapons, drop smart bombs, and battle stage enemies and bosses.

The entire game (HTML, CSS, JavaScript, and inline assets) is contained in `index.html` (~1,669 lines).

## Development and Testing

### Running the Game

- **Open locally**: Double-click `index.html` or open it in a browser with `file://` protocol
- **Local server** (recommended for testing): 
  ```bash
  python -m http.server 8000
  # Then open http://localhost:8000
  ```
- **Live testing**: Any modern browser (Chrome, Firefox, Safari, Edge) supports the full feature set

### Testing Modifications

1. Make changes to `index.html`
2. Refresh the browser (Ctrl+R or Cmd+R)
3. Game state persists in localStorage, so clear browser storage or use incognito mode to reset to defaults:
   - `skyfighter.bestScore` — high score
   - `skyfighter.muted` — mute state
   - `skyfighter.difficulty` — selected difficulty
   - `skyfighter.reducedMotion` — accessibility preference

### Key Testing Scenarios

- **Gameplay**: Launch game, complete at least one stage to verify enemy spawning, collision detection, and progression
- **Touch controls** (mobile): Test joystick movement and fire/bomb buttons on a mobile device or Chrome DevTools device emulation
- **Audio**: Unmute and verify sound effects play during gameplay
- **Accessibility**: Test reduced motion setting (Menu → Accessibility toggle) to verify animations respect user preferences
- **Performance**: Press the menu pause button and check console for FPS metrics (tracked internally)

## Code Architecture

### Game Loop and State Management

The game runs on `requestAnimationFrame` with delta-time handling:

```
requestAnimationFrame(loop) → trackPerf(dt) → update(dt) → draw()
```

Key game state variables:
- `running` — whether game is in active gameplay
- `paused` — manual pause state (separate from game-over)
- `score`, `multiplier`, `combo` — player progression metrics
- Arrays: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`
- `player` object — ship position, health, weapon, bombs
- `boss` — active boss (if stage has one)
- `stage` — current level number
- `difficulty` — game difficulty setting (0–3, affects enemy patterns and health)

### Game Systems

#### Update Logic (`update(dt)`)
- **Player movement**: Mouse/touch target tracking; joystick input on mobile
- **Weapon system**: Fires based on `weaponType` and `weaponLevel` (1–3)
- **Enemy spawning**: Waves determined by `KILLS_PER_STAGE` and stage themes
- **Collision detection**: Bullets vs. enemies, enemy bullets vs. player, power-ups pickup
- **Boss logic**: Special HP bar, unique firing patterns, stage progression
- **Power-up effects**: Life restore, bomb count increase, weapon upgrade
- **Particle effects**: Explosions, hits (100+ particles on screen)

#### Rendering (`draw()`)
- Canvas 2D context with quality-based optimizations:
  - `quality === 0`: Glow effects enabled (shadow blur)
  - `quality > 0`: Stripped for performance on weak devices
- Layered rendering: enemies → bullets → player → boss → UI
- HUD overlay: Score, multiplier, lives, bomb count, weapon status
- Vignette effect during damage (red pulse)
- Flash effects on hits and bomb detonation

#### Audio System
- Uses Web Audio API (`AudioContext`)
- Audio initialization deferred until user interaction (browser autoplay policy)
- Sound effects: laser, explosion, pickup, stage clear, boss defeat
- `muted` setting persists in localStorage
- Fallback graceful: game plays without audio if API unavailable

#### UI and Controls

**Desktop:**
- Mouse aim (crosshair) and click to fire
- Spacebar for bombs

**Mobile:**
- Virtual joystick in bottom-left (drag to move)
- Fire button (bottom-right)
- Bomb button (top-right)
- Toggle buttons for mute, motion, pause (top bar)

**Difficulty Selection:**
- Accessed via overlay menu
- 4 levels: Easy → Normal → Hard → Insane
- Affects enemy health, spawn rate, and bullet patterns
- Stored in localStorage for persistence

### Configuration and Balancing

Key configuration objects at top of script:

- **`DIFFICULTIES`**: Enemy health multipliers and spawn behavior per difficulty
- **`THEMES`**: Stage-specific enemy color sets and visual variations
- **`ENEMY_TYPES`**: Fighter, tank, and boss stats (health, speed, fire rate)
- **`BOSSES`**: 3 unique boss definitions with attack patterns and health pools
- **`KILLS_PER_STAGE`**: Enemies required to complete a stage (default: 18)
- Weapon levels: Increases bullet speed and spread; reaches cap at level 3
- Bomb mechanics: Limited count; recharge on power-up pickup

### Performance Considerations

- **Frame tracking**: `trackPerf(dt)` logs average FPS (3 recent frames)
- **Delta-time capping**: Max 50ms per frame to prevent spiral-death physics
- **Canvas optimization**: Quality setting reduces shadow/glow on low-end devices
- **Particle pooling**: Particles are created on-demand; expired particles are removed
- **Rendering culling**: Only visible entities are drawn (though canvas clipping is not used)

## Common Development Tasks

### Add a New Enemy Type

1. Add entry to `ENEMY_TYPES` object with `hp`, `speed`, `fireRate`, etc.
2. In `spawnEnemy()`, add a condition to spawn the new type
3. In `update()` enemy loop, handle the new type's behavior (e.g., unique firing pattern)
4. Update `drawPlane()` or add a new draw function for visual representation

### Adjust Difficulty Balancing

Edit `DIFFICULTIES` object to modify:
- Enemy health scale (relative to base)
- Spawn frequency and timing
- Bullet patterns and speeds

### Add a New Boss

1. Add entry to `BOSSES` array with `name`, `hp`, `speed`, `fireCycle`
2. In `update()`, implement the boss firing logic in the boss handling section
3. Add a corresponding draw function or use existing `drawBossShip()`
4. Test on each difficulty to ensure appropriate challenge

### Modify Game Balance (Speed, Damage, etc.)

These are hardcoded but easily found:
- Player bullet damage: Search for `enemies.forEach` collision logic
- Enemy bullet speed: `enemyBullets` push statements
- Player health: `player.hp` initialization
- Weapon upgrade thresholds: `weaponLevel` cap and increment logic

### Add Visual or Sound Effects

- **New particle effect**: Push to `particles` array with `{ x, y, vx, vy, life, maxLife, color }`
- **New sound**: Call `audio.playSound(frequency, duration)` in appropriate event handler
- **New animation**: Use CSS keyframes (in `<style>`) or modify canvas drawing logic

## Code Organization Notes

- **Inline constraints**: All code is in a single IIFE (`(function() { ... })()`) to avoid global pollution
- **No external dependencies**: Pure HTML, CSS, and vanilla JavaScript (ES6+)
- **Mobile-first responsive design**: Uses viewport meta tags, `clamp()` for scaling, and safe-area-inset
- **Accessibility**: Reduced motion support, keyboard controls, ARIA labels on buttons
- **No build process**: Edit and refresh; suitable for rapid iteration and prototyping

## Debugging Tips

- Open browser DevTools (F12) and check Console for any errors
- Use `console.log()` anywhere in the game loop (just refresh to see changes)
- The game tracks FPS internally; pause to view performance stats
- Check Network tab if audio fails to load (Web Audio API may require HTTPS in production)
- Use Chrome DevTools device emulation to test mobile controls without a physical device
