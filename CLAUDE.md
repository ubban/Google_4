# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The entire game runs in a single `index.html` file with embedded CSS and JavaScript.

### Features

- **Vertical scrolling arcade shooter** with dynamic difficulty levels
- **Multiple weapon systems**: Vulcan, Laser, and Cannon with progressive levels
- **Smart bomb mechanics** for clearing the screen
- **Combo system** rewarding consecutive enemy eliminations
- **Dynamic weather/visual themes** that rotate per stage
- **Boss battles** at stage transitions with health bars
- **Responsive design** supporting both desktop (mouse/keyboard) and mobile (touch)
- **Accessibility features**: Screen reader announcements, reduced motion support
- **Audio system** with mute toggle and adaptive performance scaling

## Game Architecture

### Main Structure

The entire game logic is contained in a single IIFE (Immediately Invoked Function Expression) within the `<script>` tag, organized into clear sections:

1. **DOM References** (lines 426-461): Canvas and UI element caching
2. **Storage Keys** (463-466): LocalStorage identifiers for persistence
3. **Accessibility** (468-471): Screen reader announcement function
4. **Performance Tracking** (473-483): Frame time monitoring and quality adjustment
5. **Game Configuration** (485-499): Difficulty settings and visual themes
6. **Game State Objects** (500+): Entities, systems, and game flow

### Core Game Objects

- **Player**: Ship with position, health, bombs, current weapon, and multiplier
- **Enemies**: Various enemy types with health, AI, shooting patterns
- **Projectiles**: Bullets and bombs with collision detection
- **Weapons**: Vulcan, Laser, Cannon with different fire rates and damage
- **Boss**: Special enemy with large health pool and complex attack patterns
- **Particles/Effects**: Visual feedback for hits, explosions, combo numbers

### Game Loop

The main game loop:
1. Handles input (mouse, keyboard, touch)
2. Updates game state (player, enemies, projectiles, effects)
3. Performs collision detection
4. Renders all entities to canvas
5. Updates HUD (score, health, weapons, boss bar)
6. Checks win/lose conditions

## Key Systems

### Difficulty Settings

Three difficulty levels with distinct parameters:

```
Easy:   Slower spawning, weaker enemies, 4 lives, 3 bombs, 0.8x boss HP
Normal: Standard challenge, 3 lives, 2 bombs, 1.0x boss HP
Hard:   Fast spawning, aggressive enemies, 3 lives, 2 bombs, 1.3x boss HP
```

Difficulty preference is persisted to localStorage.

### Visual Themes

Six rotating background themes that change per stage:
- Daylight Front
- Sunset Corridor
- Night Approach
- Storm Front
- Arctic Wastes
- Volcanic Ridge

Each theme has distinct sky gradient colors and cloud/sun styling.

### Weapon System

Three weapon types with 3 levels each:
- **Vulcan**: High fire rate, low damage, spread pattern
- **Laser**: Medium fire rate, medium damage, piercing
- **Cannon**: Low fire rate, high damage, explosive

Weapon powerups appear randomly and cycle through available weapons.

### Combo Multiplier

Consecutive hits build a multiplier (up to 5x) for bonus points. Score display updates in real-time. When combo breaks, the multiplier resets.

### Boss System

- Appears after killing 18 enemies per stage
- Boss has increased health (1.0-1.3x multiplier by difficulty)
- Boss bar displays at top center when active
- Boss defeat triggers stage banner and progresses to next stage

## Running and Testing

### Running the Game

1. Open `index.html` in a web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"
4. Play using:
   - **Desktop**: Mouse to move, click/Space to fire, X/Shift for bomb
   - **Mobile**: Drag left side to move, tap FIRE button, tap BOMB button

### Performance Features

- **Adaptive quality scaling**: Monitors frame times and automatically reduces rendering quality on slower devices
- **Mute toggle**: Persists audio preference
- **Reduced motion support**: Respects prefers-reduced-motion media query and disables animations for accessibility

### Controls

**Keyboard:**
- Arrow keys or WASD: Move ship
- Space or Click: Fire weapon
- X or Shift: Drop bomb
- P or Escape: Pause
- Mute button: Toggle sound
- Motion button: Toggle reduced motion effects
- Pause button: Pause game

**Mouse:**
- Move cursor to steer
- Click to fire
- (Keyboard for bomb)

**Touch:**
- Drag left side to move
- Tap FIRE button
- Tap BOMB button

## Development Notes for Claude Code

### Code Organization

- **Pure vanilla JavaScript**: No dependencies or frameworks
- **Single file architecture**: All code in one HTML file makes it self-contained
- **Canvas rendering**: Direct 2D context drawing for all game graphics
- **Game state management**: Clear separation between state objects and rendering

### Key Variables and Constants

- `DIFFICULTIES`: Configuration object for three difficulty levels
- `KILLS_PER_STAGE`: Number of enemy kills needed to trigger boss (18)
- `THEMES`: Array of 6 visual theme configurations
- `quality`: Adaptive performance level (0=full, 1=medium, 2=low)
- `frameTimes`: Array for performance tracking
- `player`, `enemies`, `projectiles`: Main game entity arrays/objects

### Accessibility Considerations

- Uses `announce()` function for screen reader announcements
- `.sr-only` class for screen reader-only content
- Reduced motion animations disabled with `prefers-reduced-motion` query
- Button elements with proper labels and ARIA attributes
- High contrast HUD elements for visibility

### Performance Optimization

- **Frame time tracking**: Automatically reduces quality if FPS drops below 40
- **Object pooling patterns**: Reuses projectiles and effects to reduce garbage collection
- **Canvas optimization**: Uses efficient rendering calls
- **Mobile optimization**: Touch controls only enabled on touch devices

### Persistence

LocalStorage keys:
- `skyfighter.bestScore`: Highest score achieved
- `skyfighter.muted`: Audio preference (true/false)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference

## Common Tasks

### Adding a New Enemy Type

1. Define enemy properties in a configuration object
2. Create spawn logic in the enemy spawning section
3. Define AI and shooting behavior
4. Add collision and damage handling
5. Test against player and bomb collision

### Modifying Difficulty

Edit the `DIFFICULTIES` object with new spawn rates, speeds, and health multipliers.

### Adding Visual Theme

Add a new entry to the `THEMES` array with sky colors, cloud, and sun values.

### Adjusting Weapon Balance

Modify weapon properties in the weapon system section (fire rate, damage, spread patterns).

## Browser Compatibility

Requires:
- HTML5 Canvas API
- ES6 JavaScript (arrow functions, template literals, const/let)
- LocalStorage API
- Touch events API (for mobile)
- CSS Flexbox
- CSS gradients and filters

Tested and optimized for modern browsers (Chrome, Firefox, Safari, Edge).
