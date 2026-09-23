# Sky Fighter - Codebase Documentation

## Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with vanilla JavaScript. It features a single-file implementation (`index.html`) with no external dependencies, making it highly portable and playable in any modern browser.

## Game Features

- **Three Difficulty Levels**: Easy, Normal, Hard with different spawn rates, enemy speeds, and boss HP multipliers
- **Player Progression**: Six stages with escalating difficulty and themed environments
- **Boss Battles**: Six unique bosses (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with distinct attack patterns
- **Weapon System**: Vulcan gun (spread shots) and Laser (piercing shots) with 3 upgrade levels
- **Power-ups**: Life restoration, bombs, and weapon upgrades dropped randomly by defeated enemies
- **Score Multiplier**: Chain-based combo system that increases score multiplier up to 10x for consecutive kills
- **Smart Bombs**: One-hit stage clear that damages bosses significantly
- **Touch Controls**: Full mobile support with virtual joystick and action buttons
- **Accessibility**: Reduced motion mode, screen reader support, and keyboard/mouse input options
- **Audio**: Procedurally generated sound effects using WebAudio API (no external audio files)
- **Adaptive Performance**: Dynamic quality scaling based on frame timing to maintain 60 FPS

## Architecture

### Single-File Structure
All game logic, rendering, and input handling is contained in one HTML file. This approach:
- Eliminates build complexity
- Ensures maximum portability
- Makes the game playable offline
- Simplifies distribution and setup

### Core Game Loop
The main game loop runs via `requestAnimationFrame` and executes at 60 FPS:
1. **Input Processing**: Keyboard, mouse, touch, and joystick input
2. **Game Update**: Player movement, bullet physics, collision detection, enemy AI
3. **Canvas Rendering**: Scene setup, entity drawing, UI updates
4. **Performance Tracking**: Frame time monitoring for adaptive quality

### State Management
- **Player**: Position, health, weapon, cooldown timers, invulnerability
- **Entities**: Enemies, bullets, particles, power-ups, boss
- **Game State**: Score, lives, stage, running/paused/gameOver flags, chain multiplier
- **Persistent Data**: Best score, mute setting, difficulty, reduced motion preference (localStorage)

## Key Systems

### Difficulty Settings (`DIFFICULTIES`)
Configurable per-difficulty parameters:
- `spawnBase/spawnMin`: Enemy spawn rate timing
- `enemySpeed`: Enemy movement speed multiplier
- `enemyShootChance`: Probability enemies fire projectiles
- `lives/bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Enemy Types (`ENEMY_TYPES`)
- **Grunt**: Basic enemy, low threat
- **Interceptor**: Fast, weaving movement
- **Tank**: High HP, low speed, high damage
  Each has color, score value, and minimum stage availability.

### Boss System (`BOSSES` & `BOSS_PATTERNS`)
Each boss has:
- Unique visual design (colors, hull/core/wing)
- HP multiplier
- Attack pattern set (3 patterns per boss, cycling)
- Patterns include: radial bursts, aimed spreads, spirals, columns, homing shots, etc.

Boss entry state prevents collision damage during arrival. Telegraph phase (white flash) before attacks.

### Weapon System
**Vulcan** (default):
- Spread shot pattern widens with level (1→2→3 bullets)
- Fixed bullet speed
- Faster fire rate

**Laser**:
- Pierce mechanic (bullets damage multiple enemies)
- Narrower fire rate but faster projectiles
- Glowing aura around player

Toggled via power-up at max level.

### Score Chain System
- Consecutive enemy kills extend chain timer (2.2s)
- Every 5 kills increases multiplier cap (max 10x at 50+ kills)
- Chain breaks on missed shots or taking damage
- Score = (base score × multiplier)

### Visual Themes (`THEMES`)
Six scrolling sky themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with:
- Gradient backgrounds
- Parallax cloud layers
- Theme-specific colors and atmosphere

Each stage cycles through themes automatically.

### Collision Detection
Rectangle-based AABB (axis-aligned bounding box) overlap testing via `rectsOverlap()`:
- Player ↔ Enemies
- Player ↔ Enemy bullets
- Player ↔ Power-ups
- Player bullets ↔ Enemies
- Player bullets ↔ Boss
- Boss ↔ Player

### Audio System
WebAudio API with:
- Frequency sweep synthesis for "beep" sounds
- Noise burst generation
- Stereo panning based on on-screen position
- Dynamic compressor for volume control
- Multiple SFX: shoot, laser, explosions (by type), hits, bomb, powerup, boss sounds, gameover

Audio can be muted and is gated by user interaction (autoplay policy).

### Input Handling
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move cursor to steer, click to fire
- **Touch**: Virtual joystick (left) and action buttons (right)
- **Priority**: Keyboard/joystick > mouse (no fighting)

### Touch Controls
- Left zone (55% width): Virtual joystick with base/stick visualization
- Right zone (45% width): Two large buttons (FIRE red, BOMB yellow)
- Joystick constraint radius prevents over-extension
- Full passive event handling to prevent scroll

### Accessibility Features
- Reduced motion mode: Disables all animations/effects (can be toggled)
- Screen reader support: `aria-live` status region for game events
- Respects system prefers-reduced-motion setting
- High contrast buttons and UI
- Icon-based controls with tooltips

### Performance Optimization
- **Quality Levels**: 0=full, 1=medium, 2=low
  - Adjusts particle counts
  - Disables glow effects at lower quality
  - Reduces shadow blur
- **Frame Time Tracking**: 45-frame rolling average
  - Scales up quality if avg < 1/56s
  - Scales down if avg > 1/40s
- **Pruning**: Bullets/particles/enemies removed when off-screen

## Development Guidelines

### Adding Features
When extending the game, consider:

1. **New Enemy Types**: Add to `ENEMY_TYPES`, define sprite function if needed
2. **New Boss**: Add to `BOSSES` array and corresponding pattern array in `BOSS_PATTERNS`
3. **New Power-up**: Add type check in `applyPowerUp()` and visual in draw loop
4. **New Audio**: Add to `sfx` object with beep/noiseBurst calls
5. **UI Changes**: Update overlay cards and HUD display elements

### Performance Considerations
- Avoid large particle counts (use `quality` check)
- Prune off-screen entities regularly
- Use requestAnimationFrame, not setInterval
- Canvas size is fixed at 480×720 (scaled on display)
- Shadow/glow effects are quality-dependent

### Debugging
- Check `frame` counter for timing issues
- Monitor `quality` variable for performance scaling
- Use console to inspect entity arrays (bullets, enemies, particles)
- Adjust difficulty multipliers to test balance

### Extensibility Points
- **Themes**: Add entries to `THEMES` for new skies
- **Enemy Patterns**: Modify weave/shoot logic in enemy update
- **Boss Patterns**: Add new pattern functions and append to `BOSS_PATTERNS`
- **Difficulty**: Add new difficulty presets to `DIFFICULTIES`
- **Weapons**: Implement new weapon types in `fireWeapon()`
- **Score Events**: Trigger milestone milestones in `addScore()`

## File Structure

```
index.html
├── HTML5 structure & meta tags
├── Global CSS styling & animations
│   ├── Canvas/overlay layout
│   ├── HUD elements (score, weapons, hearts, bombs)
│   ├── Boss health bar
│   ├── Combo/stage banners
│   ├── Pause & gameover overlays
│   ├── Touch controls (joystick, buttons)
│   ├── Icon buttons (mute, motion, pause)
│   └── Accessibility classes (sr-only)
└── JavaScript (IIFE)
    ├── DOM elements caching
    ├── Constants (storage keys, difficulties, themes, enemies, bosses)
    ├── Audio system (ensureAudio, beep, noiseBurst, SFX)
    ├── Game state variables
    ├── Input handlers (keyboard, mouse, touch, joystick)
    ├── Game loop (update & draw)
    ├── Entity logic (player, bullets, enemies, boss)
    ├── Collision detection
    ├── Particle & explosion system
    ├── Boss AI & attack patterns
    ├── UI rendering (hearts, bombs, HUD)
    └── Storage persistence (localStorage)
```

## Testing & Validation

- Test all three difficulty levels for balance
- Verify touch controls on mobile devices
- Check reduced motion mode disables all animations
- Validate boss patterns don't overlap on-screen
- Ensure particle counts don't tank FPS (check quality variable)
- Test persistence (reload page, check saved score/settings)

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support (test WebAudio autoplay policies)
- Mobile browsers: Full support with touch input

Requires: ES6+ (arrow functions, const/let, template literals)

## Common Modifications

### Adjust Boss Difficulty
Modify `baseHp` calculation in `spawnBoss()` or `bossHpMul` in difficulty presets.

### Change Stage Count
Increase `KILLS_PER_STAGE` to extend or reduce to shorten stages.

### Add New Boss
1. Add entry to `BOSSES` array
2. Add pattern array to `BOSS_PATTERNS` (must be same index)
3. Update `THEMES` if adding new stage
4. Test stage progression and boss selection

### Customize Controls
Modify key checks in `window.addEventListener('keydown')` or touch handlers.

### Tweak Enemy Spawn
Adjust `spawnTimer` math in update loop or modify difficulty spawn parameters.

## Performance Notes

The game runs in a single thread with no Web Workers. Adaptive quality scaling helps maintain 60 FPS even on lower-end devices. Most expensive operations:
- Particle rendering (mitigated by quality scaling)
- Canvas shadow/glow effects (disabled at low quality)
- Collision detection loops (O(n²) for bullets vs enemies, but n is typically small)

For significant feature additions, monitor frame time via `frameTimes` array.
