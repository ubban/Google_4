# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with vanilla HTML5, Canvas, and JavaScript. The game features multiple stages with increasingly difficult enemies, a dynamic weapon system, smart bombs, and boss encounters across six themed stages.

## Game Architecture

The entire game is contained in a single `index.html` file with:
- Embedded CSS for UI styling and animations
- Embedded JavaScript (IIFE pattern) for game logic
- Canvas rendering for all graphics

### Key Game Systems

#### 1. **Game State Management**
- `gameActive`: Boolean tracking whether the game is currently running
- `difficulty`: Current difficulty level (easy/normal/hard)
- `score`, `mult`: Score tracking with combo multiplier system
- `stage`: Current stage number (1-6)
- Persistent storage: best score, muted state, difficulty preference

#### 2. **Player (Airplane)**
- Position-based movement (keyboard/mouse/touch)
- Health system (configurable by difficulty: 3-4 lives)
- Weapon switching between VULCAN, LASER, and NOVA types
- Weapon levels (1-3) affecting fire rate and damage
- Bomb system (smart bombs with area effect)

#### 3. **Enemy System**
- Waves spawn based on difficulty-determined intervals
- Enemies have health, movement patterns, and shooting behavior
- Enemies drop weapon pickups and health/bomb powerups
- Collision detection for bullets and player

#### 4. **Weapon System**
Three weapon types with escalating power:
- **VULCAN**: Rapid-fire basic weapon (fastest, low damage)
- **LASER**: Medium fire rate, piercing shots
- **NOVA**: Slow, high-damage projectiles

Weapon levels (1-3) are obtained from enemy drops and improve fire rate/damage.

#### 5. **Difficulty Levels**
Configured in the `DIFFICULTIES` object:

| Aspect | Easy | Normal | Hard |
|--------|------|--------|------|
| Spawn Base | 75 | 58 | 40 |
| Spawn Min | 40 | 26 | 18 |
| Enemy Speed | 1.0× | 1.4× | 1.9× |
| Shoot Chance | 45% | 70% | 95% |
| Lives | 4 | 3 | 3 |
| Bombs | 3 | 2 | 2 |
| Boss HP | 0.8× | 1.0× | 1.3× |

#### 6. **Stage System**
- Six themed stages with unique visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Each stage requires 18 kills to progress
- Culminates in a boss encounter unique to each stage
- Score multiplier increases with stage progression

#### 7. **Visual Effects**
- **Vignette**: Red pulsing overlay when taking damage
- **Flash**: Screen flash on hit or bomb detonation
- **Combo Popup**: Large floating text showing combo multiplier
- **Stage Banner**: Animated stage name announcement
- **Boss Health Bar**: Visual indication of boss HP

#### 8. **Control Systems**

**Desktop Controls:**
- Mouse movement for aiming/steering
- Click or Space to fire
- X or Shift for bomb
- Arrow keys/WASD for directional movement
- P or Esc to pause

**Mobile Controls:**
- Drag on left 55% of screen to move (joystick feedback)
- Tap FIRE button (right side, red)
- Tap BOMB button (top-right, yellow)

**Accessibility Features:**
- Mute button (top-right, speaker icon)
- Reduced motion toggle for animations/effects
- Screen reader support via `#srLive` aria-live region

## Code Structure

### Canvas Rendering Loop
- Main game loop runs via `requestAnimationFrame`
- Adaptive performance tracking adjusts quality based on frame times
- Renderer handles: player, enemies, bullets, particles, UI

### Game Logic Flow
1. **Input Processing**: Capture keyboard/mouse/touch input
2. **Update**: Move entities, check collisions, update state
3. **Render**: Draw to canvas with current theme/quality settings
4. **Effects**: Manage animations (damage vignette, combos, stage banner)

### Key Functions & Managers
- **Spawner**: Controls enemy wave timing and types
- **Collision System**: Bullet-enemy, bullet-player, powerup-player detection
- **Particle System**: Explosion and hit effect particles
- **UI Manager**: Score, HUD, health display, weapon indicator
- **Audio Manager**: Mute toggle persistence

## Customization & Extension Points

### Adding New Weapons
Extend the weapon fire logic in the player update section:
```javascript
// Define new weapon type with rate, speed, damage values
// Add case in weapon switch logic
// Update weapon name display
```

### Creating New Bosses
Each stage defines its boss with:
- Initial health (multiplied by difficulty)
- Movement pattern
- Attack behavior (bullet spawn, frequency)
- Animation during defeat

### Modifying Difficulty Levels
Edit `DIFFICULTIES` object to adjust spawn rates, speeds, and health pools. The difficulty affects:
- Spawn timing (lower = more frequent)
- Enemy speed multiplier
- Enemy shoot chance (0-1)
- Player resources (lives, bombs)
- Boss health scaling

### Adding Visual Themes
`THEMES` array defines stage appearance:
- `sky`: Gradient colors [top, middle, bottom]
- `cloud`: RGB color for cloud rendering
- `sun`: RGB color for sun/light effect
- `name`: Stage display name

## Performance Optimization

**Adaptive Quality System:**
- Tracks last 45 frame times
- Reduces quality if avg frame time > 1/40s (25ms)
- Increases quality if avg frame time < 1/56s (~17ms)
- Quality levels: 0 (full) → 1 (medium) → 2 (low)

**Rendering Optimizations:**
- Canvas cleared and redrawn each frame
- Particle pool reuse for explosions
- Texture-free rendering (geometric/color-based)

## Storage & Persistence

LocalStorage keys:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## Known Quirks & Design Notes

1. **Single HTML File**: Entire game is self-contained for easy deployment
2. **No External Assets**: All graphics rendered via Canvas (no image/audio files)
3. **Mobile-First Responsive**: Uses `clamp()` for fluid sizing across devices
4. **Touch-Area Zones**: Left 55% for movement, right 45% for actions (optimized for thumbs)
5. **Prefers Dark Mode**: Uses CSS `color-scheme: dark` and provides reduced-motion support

## Testing & Debugging

- Open DevTools console to check for errors
- Test on different screen sizes (mobile/tablet/desktop)
- Verify touch controls on actual touch devices
- Check localStorage in DevTools Application tab for persistence
- Frame timing visible in performance profiler

## Future Enhancement Ideas

- Sound effects and music (currently muted capability exists)
- Local high score leaderboard expansion
- Additional weapon types
- Enemy formation patterns
- Power-up variety expansion
- Difficulty scaling between stages
- Replay/spectate system
