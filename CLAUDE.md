# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in vanilla HTML5 Canvas and JavaScript. Players control a fighter plane, manage different weapon types, use smart bombs, and engage stage bosses. The game is fully self-contained in a single `index.html` file.

### Key Features
- Vertical scrolling shoot-em-up gameplay
- Player health and damage system with visual feedback
- Weapon system with multiple weapon types and upgrades
- Smart bomb mechanics with visual effects
- Stage-based progression with boss battles
- HUD displaying health, bombs, multiplier, and weapon info
- Responsive canvas that adapts to different screen sizes
- Touch and keyboard support for controls
- Visual feedback: hit flashes, vignette effects, explosions

## Project Structure

### Repository Files
- **index.html** - Complete game implementation (58KB)
  - Embedded CSS for styling and animations
  - Complete JavaScript game engine
  - All game assets (generated procedurally or as SVG)

- **README.md** - Minimal documentation (timestamp only)

- **CLAUDE.md** - This file (codebase documentation)

## Technical Architecture

### Single-File Design
The entire game is contained in one HTML file for simplicity and portability. The file structure includes:

1. **HTML Head** - Metadata, viewport configuration, favicon as embedded SVG
2. **CSS** - Game styling, layout, animations (heartbeat, vignette pulse, flash effects)
3. **JavaScript** - Complete game engine and logic
4. **HTML Body** - Single canvas element + HUD elements

### Core Game Systems

#### Canvas & Rendering
- `#gameCanvas` - Main game rendering surface (400×600 resolution, scaled to fit screen)
- Canvas context used for all graphics (no sprite sheets, procedural rendering)
- Gradient background for sky effect (#0d2f5e to #7fc1ff)

#### Game State Management
The JavaScript section manages:
- Player position, health, weapons, bombs
- Enemies and projectiles (bullets, waves)
- Stage progression and boss spawning
- Multiplier tracking for scoring

#### HUD (Heads-Up Display)
Located in `#hud` with stats wrapped in `.stat` divs:
- **Hearts** (`#hearts`) - Visual health indicator using emoji
- **Bombs** (`#bombs`) - Count of available smart bombs
- **Weapon** (`#weaponStat`) - Current weapon type display
- **Multiplier** (`#multStat`) - Scoring multiplier value

#### Visual Effects
- **Vignette** (`#vignette`) - Red pulsing edge effect when player takes damage
- **Flash** (`#flash`) - Full-screen white flash on hit or bomb explosion
- **Animations** - CSS keyframe animations for heartbeat (critical health) and vignette pulse

## Game Mechanics

### Player Controls
- **Movement** - Arrow keys or touch/mouse to move plane
- **Shooting** - Automatic fire or space bar for manual fire
- **Weapon Switch** - Keys to change active weapon type
- **Smart Bombs** - Activate bomb to clear enemies and damage boss (limited quantity)

### Weapon System
Multiple weapon types available:
- Different firing patterns (straight, spread, laser)
- Damage values per weapon type
- Weapon upgrades/multipliers applied during gameplay

### Enemy System
- Spawning waves of enemies at different stages
- Boss encounters at stage progression points
- Boss health tracking and defeat detection

### Scoring & Progression
- Multiplier system that increases with successful hits
- Stage-based progression
- Boss-based level advancement

## Code Organization

The JavaScript is organized into functional sections:
1. **Initialization** - Canvas setup, event listeners
2. **Game Loop** - Update and render cycle
3. **Player Logic** - Movement, weapon firing, damage
4. **Enemy Logic** - Spawning, movement, AI
5. **Projectile Logic** - Bullet tracking, collision detection
6. **Boss Logic** - Boss spawning, behavior, health management
7. **HUD Updates** - Score, health, bomb display
8. **Input Handling** - Keyboard and touch event listeners
9. **Utility Functions** - Collision detection, random spawning

## Development Notes

### Browser Compatibility
- Requires HTML5 Canvas support
- Touch support for mobile devices
- Crosshair cursor (`:cursor: crosshair` on canvas)
- Uses `touch-action: none` for proper touch input handling

### Mobile Optimization
- Responsive canvas scaling (max-width/height: 100%)
- Safe area insets for notched devices (`env(safe-area-inset-top)`)
- Viewport configured for full-screen experience without zoom

### Visual Polish
- Dark theme (color-scheme: dark)
- Glass-morphism HUD elements (backdrop-filter: blur)
- Consistent color scheme:
  - Primary background: #0b1d3a (dark blue)
  - Sky gradient: #0d2f5e to #1e6fd8
  - Accent colors: #ffd54f (yellow), #7CFC00 (lime green), #ff8a65 (orange)

### Performance Considerations
- Procedural rendering (no image assets)
- Canvas-based animations
- Efficient collision detection
- Single game loop update/render cycle

## Common Modifications

To extend the game, you might:
1. **Add new weapon types** - Define new firing patterns in weapon logic
2. **Create new stages** - Add enemy waves and boss definitions
3. **Adjust difficulty** - Modify enemy spawn rates, health values, multipliers
4. **Change visuals** - Modify CSS animations, canvas drawing code, or color scheme
5. **Add sound** - Web Audio API for SFX and music

## Running the Game

Simply open `index.html` in a web browser. The game:
- Automatically detects canvas size
- Responds to touch and keyboard input
- Maintains responsive design across all screen sizes
- Stores game state in JavaScript variables (non-persistent)

---

**Last Updated:** 2026-10-03
**Project Type:** Single-file HTML5 Canvas Game
**Dependencies:** None (vanilla JavaScript)
