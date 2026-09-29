# Sky Fighter Codebase

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built entirely in vanilla HTML5 Canvas and JavaScript. Players fly a fighter aircraft through stages, collect weapon power-ups, and face boss squadrons. The game features responsive controls (mouse/keyboard/touch), multiple difficulty levels, and adaptive graphics quality based on performance.

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (1669 lines)
├── README.md           # Project metadata
└── Claude.md           # This file - codebase documentation
```

## Architecture

The entire game is implemented in a single `index.html` file with:
- **HTML Structure**: Game canvas, HUD elements, overlay UI, touch controls
- **CSS Styling**: Dark theme, responsive layout, animations and transitions
- **JavaScript (1200+ lines)**: Game logic, entity management, collision detection, audio synthesis

### Core Game Loop

```javascript
requestAnimationFrame(gameLoop)
  ├── Update logic (player, enemies, bullets, effects)
  ├── Collision detection
  ├── Render to canvas
  └── Performance tracking
```

## Key Components

### Game State Management

| Variable | Purpose |
|----------|---------|
| `gameActive` | Whether game is currently running |
| `gamePaused` | Whether game is paused |
| `quality` | Adaptive graphics quality (0=full, 1=medium, 2=low) |
| `difficulty` | Current difficulty setting (easy/normal/hard) |
| `stage`, `kills`, `stageKills` | Game progression tracking |
| `player` | Player aircraft object with position, health, weapon state |
| `enemies[]` | Array of active enemy entities |
| `bullets[]` | Array of active projectiles |
| `effects[]` | Visual effects (explosions, popups, etc.) |

### Difficulty Settings

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### Entity Types

#### Player Aircraft
- Position, velocity, health (lives system)
- Weapon system with three upgradeable types:
  - **Vulcan**: Standard rapid-fire (default)
  - **Laser**: Fast, thin beams
  - **Spread**: Wide cone attack
- Smart bomb ability (limited uses per stage)
- Speed: Fixed velocity with screen-bound constraints

#### Enemies

| Type | Behavior | Spawn Rate |
|------|----------|-----------|
| `grunt` | Slow, straight descent, basic fire | Common |
| `interceptor` | Medium speed, diagonal pattern, frequent fire | Medium |
| `tank` | Slow, armored (2 hits), strong projectiles | Moderate |
| `boss` | Stage boss, high HP, complex patterns, large bullets | 1 per stage |

Enemy spawn rate scales with stage progression (capped at KILLS_PER_STAGE = 18 kills per stage).

### Visual Theme System

Six visual themes rotate per stage:
1. **Daylight Front** - Blue sky, white clouds
2. **Sunset Corridor** - Orange/purple gradient
3. **Night Approach** - Dark blue, subtle clouds
4. **Storm Front** - Green tones, dark clouds
5. **Arctic Wastes** - Light blue, icy feel
6. **Volcanic Ridge** - Red/orange, intense heat

Each theme has custom sky gradient, cloud color, and sun tint.

## Gameplay Systems

### Weapon System

Weapons upgrade through three levels (1→3):
- **Vulcan Lv1/2/3**: Increasing fire rate and bullet damage
- **Laser Lv1/2/3**: Faster projectile speed and damage scaling
- **Spread Lv1/2/3**: Wider cone and additional bullets

Weapon pickups drop from defeated enemies and last until end of stage.

### Scoring System

- **Base Kill Points**: 100 points per enemy
- **Combo Multiplier**: Increases for consecutive kills without taking damage
  - Resets to 1x on player hit
  - Visual feedback via "×N" popup
- **Boss Bonus**: Large score bonus on stage boss defeat
- **Best Score Tracking**: Persisted to localStorage

### Collision Detection

- Pixel-perfect collision for projectiles vs entities
- Bounding box collision for player movement
- Explosion radius checks for bomb damage
- Screen bounds enforcement

### Audio System

**Web Audio Synthesis** - All sound effects generated procedurally:
- **Beeps/Tones**: `beep(freq, duration, type, volume, options)`
- **Noise Bursts**: `noiseBurst(duration, volume, options)` - random noise for explosions
- **Mute State**: Persisted to localStorage (`skyfighter.muted`)

Sound effects for:
- Weapon fire (different per weapon type)
- Enemy explosions (type-specific sounds)
- Player damage (warning beep)
- Bomb detonation (loud noise burst)
- Stage clear, game over, etc.

## User Interface

### HUD (Heads-Up Display)

**Top Center Elements**:
- Current Score
- Best Score
- Combo Multiplier (×N)
- Weapon Name and Level
- Health Hearts (❤️)
- Smart Bombs (💣)

### Control Buttons

**Desktop**:
- **Mouse**: Move aircraft to follow cursor, click to fire
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause

**Mobile/Touch**:
- **Left Zone**: Drag to control aircraft position with virtual joystick
- **FIRE Button**: Tap to shoot
- **BOMB Button**: Tap to detonate smart bomb

### Overlay Screens

| Screen | Purpose |
|--------|---------|
| **Title** | Main menu with difficulty selection and controls hint |
| **Game Over** | Final score, best score comparison, stage reached |
| **Stage Banner** | "Stage N - [Theme Name]" announcement on stage start |
| **Boss Bar** | Health bar for boss encounters |

## Performance Optimization

### Adaptive Quality System

Monitors frame times (rolling 45-frame average):
- **Quality 0** (Full): All visual effects, particle count
- **Quality 1** (Medium): Reduced particle effects
- **Quality 2** (Low): Minimal visual effects, optimized rendering

Automatically adjusts if FPS drops below 40 or rises above 56.

### Rendering Optimization

- Single canvas context used throughout
- Layered rendering: background → entities → UI → effects
- Efficient sprite-like drawing (no external image assets)
- Cloud parallax effect for depth

### Reduced Motion Mode

Accessibility setting to disable:
- Screen shake on impact
- Flash effects on bomb/hit
- Pulse animations
- Persistent in localStorage (`skyfighter.reducedMotion`)

## Data Persistence

Uses browser `localStorage`:

| Key | Content |
|-----|---------|
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | Audio mute state |
| `skyfighter.difficulty` | Last selected difficulty |
| `skyfighter.reducedMotion` | Accessibility preference |

## Development Guide

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` with stats (hp, speed, fireChance, etc.)
2. Implement spawn logic in `spawnEnemy()`
3. Add rendering in enemy draw section
4. Add explosion sound mapping in `EXPLODE_SFX`

### Modifying Difficulty

Edit `DIFFICULTIES` object values:
- `spawnBase/spawnMin`: Enemy spawn frequency (lower = harder)
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Probability enemies fire each frame
- `bossHpMul`: Boss health multiplier

### Changing Themes

Modify `THEMES` array:
- `sky`: [bottom, middle, top] gradient RGB colors
- `cloud`: Cloud color (255,255,255 = white)
- `sun`: Sun tint (255,255,220 = yellowish)
- `name`: Display name shown in stage banner

### Adjusting Game Balance

Key tuning parameters:
- `KILLS_PER_STAGE`: Enemies needed to clear stage
- Weapon damage values (search `player.bullet`)
- Enemy HP values in `ENEMY_TYPES`
- Bomb damage radius and cooldown

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Web Audio API for sound synthesis
- Touch Events for mobile controls
- localStorage for persistence

## Accessibility Features

- Screen reader support with `aria-live` region (`#srLive`)
- High-contrast colors and text shadows
- Keyboard and touch alternatives to mouse controls
- Reduced motion option for vestibular issues
- Semantic HTML structure

## Known Limitations

- Single-file structure limits code organization
- No external assets (all graphics drawn programmatically)
- Audio synthesis may not work in all environments (e.g., iOS background apps)
- Performance on very low-end devices may require quality adjustment

## Future Enhancement Ideas

- Multiple game modes (survival, time attack, score chase)
- Weapon customization between stages
- Achievements/medals system
- Leaderboard with online sync
- Power-up variety (shield, rapid-fire duration, invincibility)
- Enemy patterns and formations
- Stage-specific backgrounds and obstacles
- Networking for competitive play
