# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file with embedded JavaScript and CSS. Players control a fighter jet, switch weapons, drop smart bombs, and defeat stage bosses.

- **Type**: Browser-based game (single HTML file)
- **Technology**: Vanilla JavaScript, Canvas API, CSS
- **Platform**: Web (responsive, touch-friendly)
- **Target**: Desktop and mobile browsers

## Project Structure

The entire game is contained in `index.html` with three main sections:

### 1. **HTML Structure** (lines 1-200)
- Meta tags for viewport, description, theme color
- Single `#container` div with `#gameCanvas` for game rendering
- HUD (Heads-Up Display) for stats: Lives, Score, Weapon type, Bombs
- Mobile-friendly responsive design with safe area inset support

### 2. **CSS Styling** (lines 10-100+)
- Dark space theme with gradient background (`#05122b` to `#1e6fd8`)
- Game canvas centered with crosshair cursor
- HUD positioned at top center with white text and shadow
- Responsive sizing using `clamp()` for mobile/desktop
- Touch-action and user-select disabled for game input

### 3. **JavaScript Game Engine** (bulk of file)
Contains:
- **Canvas initialization**: 2D context, viewport handling
- **Game state**: Lives, score, current weapon, bomb count
- **Entity system**: Player, enemies, bullets, bombs, effects
- **Game loop**: Input handling, physics, rendering, collision detection
- **Weapon system**: Multiple weapon types with different behaviors
- **Boss mechanics**: Stage progression with boss encounters
- **Audio/Visual effects**: Explosions, impacts (visual only, no audio tags)

## Key Game Systems

### Player
- Positioned at bottom center of screen
- Moves with mouse (desktop) or touch (mobile)
- Fires bullets continuously in current weapon mode
- Can drop bombs (limited supply) with spacebar

### Weapon Types
- Default: Standard bullets
- Spread: Multiple bullets at angles
- Laser: Concentrated beam
- Missiles: Homing projectiles
- Toggle with arrow keys or on-screen buttons

### Enemies
- Random spawn from top of screen
- Basic movement patterns (straight down, sine wave, etc.)
- Drop items on defeat: weapon pickups, bomb pickups
- Enemies drop bullets when destroyed

### Bosses
- Appear at specific score milestones
- More health, complex attack patterns
- Stage progression increases difficulty
- Boss defeat unlocks next stage

### Collision System
- Player hitbox: center-based, small radius
- Bullet hitbox: per-bullet detection
- Explosion radius for splash damage
- Item pickup detection (wide radius)

## How to Run

1. **Open in browser**: Simply open `index.html` in any modern web browser
2. **No build required**: Runs directly, no dependencies or build step
3. **Mobile friendly**: Works on touch devices (phone/tablet)

## Controls

| Control | Action |
|---------|--------|
| Mouse move | Aim/move player |
| Left click / Touch drag | Move player (alternative) |
| Spacebar | Drop bomb |
| Arrow keys ← → | Switch weapon |
| On-screen buttons | Mobile weapon/bomb controls |

## Development Notes

### Single File Architecture
- All code is in `index.html` for maximum portability
- Easy to deploy (just one file)
- Challenge: File size and organization within one file

### Responsive Design
- Uses viewport meta tags for mobile
- `clamp()` CSS for flexible sizing
- Safe area insets for notched devices
- Touch-action disabled to prevent browser zoom

### Performance Considerations
- Canvas rendering at 60 FPS target
- Sprite-based collision detection (no pixel-perfect)
- Enemy pooling to reduce garbage collection
- Efficient draw order: background → entities → HUD

### Asset Management
- All graphics drawn procedurally (no image files)
- Shapes use `fillRect`, `fillPath`, `arc`, etc.
- Color theme: Blue sky, white UI, colorful explosions

## Future Enhancement Opportunities

- **Audio**: Add sound effects (lasers, explosions, boss music)
- **Score persistence**: Save high scores to localStorage
- **Difficulty modes**: Easy/Normal/Hard with adjustable spawning
- **More boss variety**: Unique boss patterns per stage
- **Power-ups**: Shield, rapid fire, slow time, etc.
- **Mobile UI**: On-screen joystick + dedicated buttons
- **Leaderboard**: Track top scores online

## Code Style

- **Vanilla JS**: No frameworks or build tools
- **Procedural**: Imperative game loop style
- **Canvas API**: Direct drawing, no scene graph
- **Responsive**: Mobile-first CSS with media queries
- **Accessibility**: ARIA labels for HUD, semantic HTML

## Testing

Currently no automated tests. To test:
1. Open `index.html` in browser
2. Play through multiple stages
3. Test weapon switches
4. Verify bomb mechanics
5. Check mobile touch controls

## Git Workflow

- Branch: `claude/inspiring-planck-hqjtr1` for development
- Commit messages should describe game mechanics or fixes
- This CLAUDE.md should be updated when major systems change
