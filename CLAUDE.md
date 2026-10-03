# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 Canvas-based web application. It's a browser-based arcade game where players control an aircraft, destroy waves of enemies, and defeat stage bosses while collecting power-ups and managing resources.

**Repository**: `ubban/google_4`  
**Type**: Interactive Web Game  
**Technology Stack**: HTML5, Canvas API, Vanilla JavaScript, CSS3

---

## Architecture

### File Structure

```
/home/user/Google_4/
├── index.html       (Single-file game: HTML + CSS + JavaScript)
├── README.md        (Project metadata)
└── .git/            (Version control)
```

### Code Organization

The game is implemented as a monolithic HTML file with three main sections:

1. **HTML Structure** (`<head>` - `<body>`): DOM elements for game canvas and UI overlays
2. **CSS Styling** (`<style>`): Game visuals, animations, responsive design
3. **JavaScript Game Engine** (`<script>`): Game logic, entity management, input handling

---

## Core Game Features

### Game Mechanics

1. **Player Aircraft**
   - Single controllable player ship positioned at bottom-center of screen
   - 3 lives by default (adjustable per difficulty)
   - Takes damage from enemy projectiles and collisions
   - Destructible when health reaches zero

2. **Enemy System**
   - Waves of enemies spawn at top of screen
   - Enemy types: basic fighters with varying behaviors
   - Enemies shoot projectiles at player
   - Stage completion: destroy 18 enemies per stage (KILLS_PER_STAGE = 18)
   - Progressive difficulty: spawn rates decrease, enemy speed increases with stage

3. **Boss Encounters**
   - One boss per stage
   - Boss appears after defeating all regular enemies
   - Boss has health bar displayed at top of screen
   - Boss defeat triggers stage completion and score bonus

4. **Weapon System**
   - Multiple weapon types with upgrade levels (1-3)
   - Vulcan: basic rapid-fire weapon (lime green)
   - Other weapons: varied firing patterns and damage
   - Weapon upgrades obtained from downed enemies
   - Current weapon and level displayed in HUD

5. **Smart Bombs**
   - Limited-use screen-clearing weapon
   - Number displayed in HUD (heart emojis indicate remaining bombs)
   - Activate with Space (keyboard) or tap BOMB (mobile)
   - Bombs clear all on-screen enemies and projectiles

6. **Scoring System**
   - Base points per kill
   - Combo multiplier for consecutive kills without taking damage
   - Combo counter resets on player damage
   - Score multiplier displays current combo (x1, x2, x3, etc.)
   - Best score persisted in browser localStorage

7. **Combo/Streak System**
   - Multiplier increases for consecutive enemy kills
   - Visual feedback: "COMBO x2" / "COMBO x3" pop-ups
   - Combo resets on player damage
   - Encourages aggressive, damage-free play

8. **Visual Feedback**
   - Screen flash on hit (white flash)
   - Screen flash on bomb use (brighter flash)
   - Red vignette (edge darkening) with pulsing animation during damage
   - Stage banners announce progression ("STAGE 2", etc.)
   - Particle effects and animations

### Difficulty Settings

Three difficulty presets with customizable spawn/speed/damage parameters:

| Parameter | Easy | Normal | Hard |
|-----------|------|--------|------|
| Spawn Base (frames) | 75 | 58 | 40 |
| Spawn Minimum | 40 | 26 | 18 |
| Enemy Speed | 1.0x | 1.4x | 1.9x |
| Shoot Chance | 45% | 70% | 95% |
| Starting Lives | 4 | 3 | 3 |
| Starting Bombs | 3 | 2 | 2 |
| Boss HP Multiplier | 0.8x | 1.0x | 1.3x |

---

## Controls & Input Handling

### Desktop Controls

| Action | Input |
|--------|-------|
| Move | Mouse movement or Arrow keys / WASD |
| Fire Weapon | Left mouse click or Space bar |
| Detonate Bomb | X key or Shift |
| Pause | P key or Esc |
| Mute | Click 🔊 button (top-right) |
| Toggle Motion Effects | Click 🎆 button |
| Pause Toggle | Click ⏸ button |

### Mobile/Touch Controls

- **Movement**: Drag left side of screen (joystick emulation)
- **Fire**: Tap red FIRE button (bottom-right)
- **Bomb**: Tap yellow BOMB button (bottom-right)
- On-screen joystick base and buttons visible on touch devices

### Input Processing

- Keyboard event listeners: `keydown`, `keyup`
- Mouse listeners: `mousemove`, `click`, `contextmenu`
- Touch listeners: `touchstart`, `touchmove`, `touchend`
- All input updates game state; actual movement/firing processed in game loop

---

## UI Elements & HUD

### In-Game HUD (Top Center)

- **Score**: Current game score
- **Best**: Highest score achieved (persisted)
- **x (Multiplier)**: Current combo multiplier (orange color)
- **Weapon**: Current weapon name and level (lime green)
- **Hearts**: Player health indicator (❤️ = 1 life)
- **Bombs**: Smart bombs remaining (💣)

### Top-Right Control Buttons

- 🔊 Mute toggle (persisted)
- 🎆 Reduced motion toggle (disables animations for accessibility)
- ⏸ Pause button

### Boss Health Bar

- Appears when boss is active
- Shows boss name and remaining health
- Red-to-orange gradient fill
- Positioned below HUD at top of screen

### Overlay Screens

- **Start Screen**: Game title, instructions, difficulty selection, score display
- **Game Over**: Final score, stage reached, best score, option to restart
- Smooth card entrance animation (unless reduced-motion enabled)

---

## Visual Themes & Rendering

### Sky Themes

Six different visual themes that cycle through stages:

1. **Daylight Front**: Blue sky, white clouds, yellow sun
2. **Sunset Corridor**: Purple-orange gradient, warm tones
3. **Night Approach**: Dark blue, cool tones, purple-blue clouds
4. **Storm Front**: Green-teal gradient, storm atmosphere
5. **Arctic Wastes**: Cyan-blue gradient, cool ice tones
6. **Volcanic Ridge**: Red-orange gradient, volcanic atmosphere

Each theme defines:
- Sky gradient colors (3-stop gradient)
- Cloud color (RGB tuple)
- Sun color (RGB tuple)
- Visual name for display

### Rendering Pipeline

Canvas-based 2D rendering:
1. Clear canvas
2. Draw sky background with gradient
3. Draw parallax clouds (scrolling effect)
4. Draw sun/celestial body
5. Draw game entities (player, enemies, projectiles, effects)
6. Draw HUD overlay (score, health, etc.)

### Performance Optimization

**Adaptive Quality System**:
- Tracks frame render time over 45-frame window
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Degrades when average frame time > 1/40s
- Upgrades when average frame time < 1/56s
- Affects particle density and visual effects

---

## Game Loop & State Management

### Frame Rate & Timing

- Target frame rate: 60 FPS
- Delta-time calculation for frame-rate independent movement
- `requestAnimationFrame` for smooth updates

### Game States

- **MENU**: Overlay open, waiting for player input
- **PLAYING**: Game active, entities updating, rendering
- **PAUSED**: Game frozen, overlay shows pause menu
- **GAME_OVER**: Player defeated, overlay shows results

### Entity System

**Player Ship**:
- Position (x, y)
- Velocity (for smooth deceleration)
- Health/lives
- Current weapon and level
- Bomb count
- Invincibility frames (after damage)

**Enemy Aircraft**:
- Position and velocity
- Health
- AI behavior (spawn point, movement pattern)
- Shoot timer and behavior
- Weapon type and damage value

**Projectiles**:
- Position and velocity
- Damage value
- Owner (player or enemy)
- Lifetime/despawn logic

**Bomb Explosions**:
- Position
- Radius and damage
- Frame counter for animation

---

## Data Persistence

Uses browser `localStorage` for:

| Key | Data | Default |
|-----|------|---------|
| `skyfighter.bestScore` | Best score achieved | 0 |
| `skyfighter.muted` | Audio mute state | false |
| `skyfighter.difficulty` | Selected difficulty | "normal" |
| `skyfighter.reducedMotion` | Reduced motion preference | false |

Load on startup, save on update. Survives page refresh and browser restart.

---

## Accessibility Features

1. **Keyboard Navigation**: All controls available via keyboard
2. **Touch Support**: Full mobile/tablet control scheme
3. **Reduced Motion Mode**: Disables animations (keyframes) for motion-sensitive users
   - Affects: vignette pulse, heart beat, combo pop-ups, stage banners, card entrance
   - Controlled by 🎆 button in-game
4. **Screen Reader Support**: 
   - Live region (`#srLive` with `aria-live="polite"`)
   - Semantic announcements for major events
5. **High Contrast Support**: Responds to `prefers-color-scheme: dark`

---

## Development & Customization

### Modifying Game Balance

Edit `DIFFICULTIES` object to adjust spawn rates, enemy speeds, and resource counts:

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
```

### Adding New Weapons

Weapon types defined in game logic. To add a new weapon:
1. Define weapon properties (name, fire rate, damage, visual)
2. Add firing logic in weapon system
3. Register weapon drop chance

### Adding Visual Themes

Add entry to `THEMES` array with sky gradient, cloud/sun colors, and name.

### Customizing UI Text

All text is in HTML or JavaScript string literals. Search for specific text to customize:
- Game title: `<h1 id="overlayTitle">`
- Subtitle: `id="overlaySub"`
- Control hints: `id="controlsHint"`

---

## Running & Deployment

### Local Development

1. Open `index.html` in a web browser
2. Game starts with menu overlay
3. Select difficulty and click "Start Game"
4. No build step required

### Browser Compatibility

- Modern browsers with HTML5 Canvas support (Chrome, Firefox, Safari, Edge)
- Requires ES6+ JavaScript support
- Touch API for mobile support
- `localStorage` for score persistence

### Deployment

- Single static HTML file – no server required
- Can be hosted on any web server or static hosting (GitHub Pages, Netlify, etc.)
- No external dependencies or build process

---

## Key Constants & Configuration

| Constant | Value | Purpose |
|----------|-------|---------|
| `KILLS_PER_STAGE` | 18 | Enemies to defeat before boss spawns |
| `PLAYER_SPEED` | Variable | Player acceleration/movement rate |
| `ENEMY_SPEED` | Variable | Enemy base velocity (difficulty-adjusted) |
| `SCREEN_WIDTH` / `SCREEN_HEIGHT` | Dynamic | Canvas dimensions (responsive) |

---

## Known Limitations & Future Enhancements

### Current Scope
- Single-player gameplay
- 6 visual themes (one per stage, cycles)
- 3 difficulty levels
- Procedurally varied enemy waves
- Simple geometric/sprite-based graphics

### Potential Improvements
- Sound effects and music
- Additional weapon types
- More complex boss patterns
- Level/stage design with patterns
- High score leaderboard
- Mobile app wrapper
- Multiplayer modes
- Endless survival mode

---

## Commit History

- `c4dc8f6`: Add Sky Fighter (current version)
- `4efdad1`: Initial commit

---

## Notes for Contributors

- **Branch**: `claude/inspiring-planck-kbybpv` - designated development branch
- **Code Style**: Vanilla JS, single-file architecture, modular logic sections
- **Testing**: Manual play-testing recommended for balance changes
- **Performance**: Monitor frame times during intensive waves; consider quality degradation
- **Mobile**: Test touch controls on various device sizes and orientations
- **Accessibility**: Maintain keyboard/touch parity and respect `prefers-reduced-motion`

---

*Last Updated: 2026-10-03*
