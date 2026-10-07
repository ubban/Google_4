# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-page HTML5 vertical shoot-'em-up game (Raiden-style). The entire game is implemented in a single 1669-line `index.html` file.

**Key Features:**
- Vertical scrolling shooter with multiple weapon types (Vulcan, Laser)
- Smart bomb mechanic
- Wave-based enemy spawning with boss encounters
- Stage progression system with difficulty scaling
- Combo/chain multiplier system for scoring
- Mobile-friendly touch controls and keyboard input
- Web Audio API sound effects
- Responsive canvas scaling
- Accessibility features (ARIA labels, screen reader support)

## Repository Structure

```
/
├── index.html        # Complete game (HTML + CSS + JavaScript)
├── README.md         # Basic project info
└── CLAUDE.md         # This file
```

## Running & Development

**To play the game:**
- Open `index.html` in any modern web browser
- Use keyboard controls or mouse to aim
- On mobile: touch controls appear automatically

**Keyboard Controls:**
- Arrow keys / WASD: Move player ship
- Space: Fire weapon
- X / Shift: Drop bomb
- P / Escape: Pause
- Click mute/pause/motion buttons in top-right corner

**Mobile Controls:**
- Left zone: Virtual joystick for movement
- Right zone: Fire button (continuous firing when held)
- Bottom center: Bomb button

**Difficulty Selection:**
- Available on the title screen before starting: Easy, Normal, Hard
- Affects enemy count, player health, bomb count, and enemy behavior
- Can be changed while paused (except during boss fights on some difficulties)

## Architecture & Code Organization

The entire game is wrapped in an **IIFE (Immediately Invoked Function Expression)** at line 423 for scope isolation.

### High-Level Structure (Reading Order)

1. **HTML Structure** (lines 353–391)
   - Game canvas container
   - HUD elements (score, lives, weapon status)
   - Overlay/title screen with difficulty selection
   - Touch control overlays (joystick and fire/bomb buttons)
   - Screen reader live region

2. **CSS Styling** (lines 10–322)
   - Dark sci-fi theme with gradient backgrounds
   - Responsive design using `clamp()` for flexible sizing
   - Animations for UI elements (card entry, heart beat, screen shake)
   - Accessibility-first with safe-area insets for notched devices
   - Reduced motion support (`prefers-reduced-motion`)

3. **JavaScript Game Logic** (lines 423–1666)

### Key Game Systems

#### Game State (lines 863–869)
```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score, lives, maxLives, running, paused, gameOver;
let bombCount, maxBombs, bombFlashTimer;
let spawnTimer, frame, kills, stage, stageKills;
let chain, chainTimer, multiplier;
```

All game entities are managed as plain objects/arrays. No class-based entities.

#### Game Loop (lines 1640–1652)
The main game loop runs via `requestAnimationFrame(loop)` and is time-step based:
- Calculates delta time (`dt`) in seconds
- Calls `update(dt)` if game is running and not paused
- Calls `draw()` to render the canvas
- Handles screen shake timing

#### Update Phase (lines 1148–1410)
Central `update(dt)` function that each frame:
1. Handles player input (keyboard/mouse/joystick)
2. Updates player position, weapon cooldown, invulnerability
3. Spawns enemies based on current wave (KILLS_PER_STAGE = 18 per stage)
4. Updates all enemy positions and shooting behavior
5. Updates all projectiles (player bullets + enemy bullets)
6. Updates particles and power-ups
7. Handles collision detection between:
   - Player and enemy bullets → damage
   - Player bullets and enemies → kill/score
   - Power-ups and player → weapon/health/bomb pickup
   - Player and enemies → collision damage
8. Tracks combo/chain multiplier (decays over time)
9. Manages stage progression and boss encounters
10. Detects game over conditions (lives = 0)

#### Draw Phase (lines 1520–1637)
The `draw()` function each frame:
1. Renders parallax scrolling cloud background
2. Draws all game entities: player, enemies, bullets, particles, power-ups
3. Renders HUD (score, lives, weapon, bombs) dynamically
4. Shows boss health bar when boss is active
5. Applies screen shake offset to canvas

#### Audio System (lines 545–677)
Web Audio API sound effects management:
- `beep(freq, dur, type, vol, opts)`: Generates tones (weapon fire, hits)
- `noiseBurst(dur, vol, opts)`: Generates noise (explosion sounds)
- Lazy initialization of audio context on first user interaction
- Dynamic audio ducking with compressor
- Stereo panning for spatial effects
- Mute toggle with persistent storage

### Key Constants & Configuration (lines 463–519)

**Difficulty Tiers** (line 485):
```javascript
const DIFFICULTIES = {
  easy: { lives: 5, bombs: 3, ... },
  normal: { lives: 3, bombs: 2, ... },
  hard: { lives: 2, bombs: 1, ... }
}
```

**Stage Themes** (line 493): Different background gradients per stage

**Enemy Types** (line 502): Different enemy archetypes with unique behavior

**Boss Data** (line 508): Boss ships with health, attack patterns, and rewards

### Important Functions to Know

**Collision Detection:**
- `rectRect(a, b)`: line 1080 – Simple AABB collision check
- No external physics library; all collision is custom

**Enemy Spawning:**
- `spawnEnemy()`: lines 1046–1065 – Creates random enemy with position/velocity/type
- Wave-based: enemies spawn until `stageKills >= KILLS_PER_STAGE`

**Boss Management:**
- `updateBoss(dt)`: lines 1411–1506 – Handles boss AI and attack patterns
- Boss appears after each stage's wave is defeated
- Boss health bar shown at top of screen

**Scoring & Combo:**
- Chain multiplier increases each time an enemy is killed
- Chain timer (line 1366): Resets if too long between kills
- Multiplier applied to all score gains (enemies, power-ups)

**Visual Effects:**
- `screenShake(mag, dur)`: line 727 – Triggers screen vibration (can be disabled via motion toggle)
- `flashHit()` / `flashBomb()`: White screen flash on damage or bomb
- Particles spawned on enemy death and explosions

**Storage & Persistence:**
- Best score, mute setting, difficulty, reduced-motion preference saved to localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Common Development Tasks

### Adding a New Enemy Type
1. Add new entry to `ENEMY_TYPES` object (line 502)
2. Define `w`, `h`, `speed`, `fireRate`, `firePattern`, `health`, `scoreReward`
3. Update `spawnEnemy()` (line 1046) to include new type in random selection
4. Implement firing logic in `update()` enemy loop if unique behavior needed

### Modifying Weapon Behavior
- Player weapon state: `player.weapon` (vulcan or laser), `player.level`, `player.cooldown`, `player.rapid`
- Firing logic in `update()` around line 1169 for Vulcan, line 1203 for Laser
- Each weapon has different cooldown/fire rate and audio

### Adding Sound Effects
1. Call `beep()` with frequency, duration, and type
   - Example: `beep(800, 0.1, 'sine', 0.3)` for a medium-pitch, short beep
2. Or use `noiseBurst()` for explosion-like sounds
3. Consider stereo panning with `panFor(x)` for positional audio

### Adjusting Game Balance
- Enemy spawn count: `KILLS_PER_STAGE` (line 491)
- Enemy behaviors: Edit `ENEMY_TYPES` and `updateBoss()`
- Player speed: `player.speed` (line 877)
- Weapon cooldowns: Player firing section in `update()`
- Boss health and attacks: `BOSSES` array and `updateBoss()` function
- Difficulty scaling: `DIFFICULTIES` object and how it's applied in `resetGame()` and `update()`

### Reducing Performance Impact
- Frame rate tracking available in `trackPerf()` (line 476)
- Particle count affects draw calls; consider pooling instead of creating/destroying
- Canvas is 480x720 (BASE_W, BASE_H at line 753) – keep in mind for large sprite counts

## Mobile & Accessibility

- **Responsive Design**: Uses `clamp()` and viewport meta tags for proper scaling on all screen sizes
- **Safe Area Insets**: Respects notches on modern phones (iPhone, etc.)
- **Touch Support**: Virtual joystick + fire/bomb buttons appear automatically on touch devices
- **Reduced Motion**: Users can toggle motion effects (screen shake, animations)
- **Keyboard Navigation**: All controls support keyboard; pause/mute/motion buttons are keyboard-accessible
- **Screen Reader Support**: Live region (id="srLive") for status announcements via `announce()` function

## Debugging Tips

- Open browser DevTools (F12) and check Console for any errors
- Use `console.log()` in the game loop with caution (impacts performance)
- Check localStorage to inspect saved settings: `localStorage.getItem('skyfighter.bestScore')`
- Frame time tracking in `trackPerf()` can help identify performance bottlenecks
- Adjust `canvas.width/height` values to test different resolutions

## Performance Notes

- The game runs at 60 FPS target but uses delta-time stepping for frame-rate independence
- Max delta time clamped to 50ms (line 1644) to prevent huge jumps on tab refocus
- Canvas is hardware-accelerated 2D context (not WebGL)
- No external dependencies – pure vanilla JavaScript
- Audio context is created lazily on first user interaction to respect browser autoplay policies
