# Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game in the style of classic arcade shmups like Raiden. The entire game is implemented as a single-file HTML application with embedded CSS and JavaScript, using Canvas 2D for rendering.

- **Type**: Arcade-style shoot-'em-up game
- **Platform**: Browser (HTML5 Canvas)
- **Architecture**: Single-file monolithic design (index.html)
- **Rendering**: HTML5 Canvas 2D API
- **Input**: Keyboard + Mouse/Touch controls

## Core Game Features

### Player Mechanics
- **Movement**: WASD or Arrow keys + mouse/touch aiming
- **Weapons**: 
  - Vulcan (rapid-fire spread)
  - Laser (concentrated beam)
  - Missile (slow but powerful homing)
  - Weapon cycling via E key
- **Smart Bombs**: Limited-use special attacks (press SPACE)
- **Health System**: 5-heart health bar (lose all hearts = game over)

### Gameplay Elements
- **Enemies**: Various enemy types with different behaviors
- **Bosses**: Stage bosses with unique attack patterns
- **Scoring**: Points for kills and combos (multiplier system)
- **Stages**: Multiple stages with increasing difficulty
- **Power-ups**: Weapon upgrades, health restoration, bomb pickups

## Code Structure

The codebase is organized as a single HTML file with three main sections:

### 1. HTML Structure (`<body>`)
```html
<div id="container">
  <canvas id="gameCanvas"></canvas>
  <div id="hud">
    <!-- HUD elements: health, bombs, weapon, score, multiplier -->
  </div>
  <div id="vignette"></div>  <!-- Damage vignette effect -->
  <div id="flash"></div>       <!-- Hit/bomb flash effect -->
</div>
```

### 2. CSS Styles (`<style>`)
- **Canvas styling**: Full-screen layout, crosshair cursor
- **HUD layout**: Responsive stat displays (hearts, bombs, weapon, score)
- **Effects**: Vignette effect, flash animations, responsive font sizing
- **Dark theme**: Color scheme with blues (#0b1d3a, #05122b) and accent colors

### 3. JavaScript Game Engine

#### Core Game Loop
- **Initialization**: Set up canvas, context, input handlers
- **Main loop**: `requestAnimationFrame` for ~60 FPS updates
- **Update cycle**: 
  1. Process input
  2. Update game state (enemies, player, bullets, effects)
  3. Handle collisions
  4. Render everything to canvas

#### Key Data Structures

**Player Object**
```javascript
{
  x, y,              // Position
  width, height,     // Dimensions
  health,            // Current health (max 5)
  bomb,              // Bomb count
  weapon,            // Current weapon (0=Vulcan, 1=Laser, 2=Missile)
  firing,            // Is player shooting
  vx, vy,            // Velocity
  lastShotTime,      // For rate limiting shots
}
```

**Bullet Object**
```javascript
{
  x, y,
  vx, vy,            // Velocity
  damage,
  type,              // 'player' or 'enemy'
  alive,
}
```

**Enemy Object**
```javascript
{
  x, y,
  vx, vy,
  health,
  maxHealth,
  type,              // 'normal', 'heavy', 'boss', etc.
  pattern,           // Attack pattern object
  shootCooldown,
  alive,
}
```

**Particle Effect Object**
```javascript
{
  x, y,
  vx, vy,
  life, maxLife,
  type,              // 'explosion', 'smoke', etc.
  color,
}
```

#### Key Game Systems

**Weapon System**
- **Vulcan**: Fast spread shots (3 bullets per shot)
- **Laser**: Continuous beam with concentration
- **Missile**: Slow homing projectiles

**Collision Detection**
- Circle/rect-based collision detection
- Player-enemy, bullet-enemy, bullet-player collisions
- Power-up collection area

**Enemy AI**
- Pattern-based movement and shooting
- Boss patterns with phases
- Difficulty scaling per stage

**Scoring & Multiplier**
- Base points for enemy kills
- Multiplier increases on consecutive hits without being hit
- Combo text display

## Important Notes for Claude

### When Modifying the Code:

1. **Single File Architecture**: Changes must keep everything in the one `index.html` file
2. **Canvas Coordinates**: Origin (0,0) is top-left; Y increases downward
3. **Responsive Design**: Code uses `clamp()`, `vw`, and `vh` units for mobile support
4. **Performance**: Be mindful of Canvas draw order (render in back-to-front order)
5. **Touch Support**: Input system supports both keyboard and touch/mouse

### Common Tasks:

**Adding a New Enemy Type**
1. Create an enemy object in the spawn logic
2. Add drawing code in the render section
3. Define movement/attack pattern in the enemy update loop
4. Add collision handling

**Adding a New Weapon**
1. Add weapon type constant (index 3+)
2. Implement bullet spawn logic in weapon switch
3. Add rendering code for bullet type
4. Add particle effects for firing

**Tweaking Game Balance**
1. Enemy health/damage values
2. Weapon damage/rate of fire
3. Player movement speed
4. Bomb effect radius/power
5. Scoring multipliers

### File Locations:
- **Game Canvas**: `#gameCanvas` (main rendering target)
- **HUD Elements**: `#hud`, `#hearts`, `#bombs`, etc.
- **Effects**: `#vignette`, `#flash` (visual feedback)

## Game Flow

1. **Start Screen** → Player sees title and waits for input
2. **Stage 1-N** → Play through stages, destroy enemies and bosses
3. **Game Over** → When player loses all health
4. **Win State** → When all stages are completed

## Styling Reference

- **Primary Color**: #0b1d3a (dark blue background)
- **Accent Colors**: #1976d2 (blue), #ffd54f (yellow), #ff8a65 (orange), #7CFC00 (lime green)
- **Text Color**: #fff with text-shadow for contrast
- **Vignette**: Red radial gradient (active when damaged)

## Development Tips

- Use browser DevTools console for debugging game state
- Canvas rendering is CPU-bound; test performance on lower-end devices
- Touch input requires no extra libraries (native browser APIs)
- Ensure all game logic updates happen before canvas render
