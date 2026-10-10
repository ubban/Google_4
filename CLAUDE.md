# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up arcade game built as a single-file HTML/JavaScript web application. Players control an aircraft to defeat waves of enemies and stage bosses while managing weapons, smart bombs, and health.

- **Repository**: https://github.com/ubban/Google_4
- **Type**: Browser-based game (HTML5 Canvas)
- **Status**: Active development

## Project Structure

### Main Files

- **index.html** (1669 lines) - Complete game implementation
  - HTML structure with game canvas and HUD
  - Embedded CSS for styling, animations, and responsive design
  - Embedded JavaScript game engine (1000+ lines)

### Directory Layout

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JS)
├── README.md           # Project readme
└── .git/               # Version control
```

## Game Architecture

### Core Components

1. **Canvas Rendering**
   - Uses HTML5 Canvas API for game graphics
   - Fullscreen responsive design with viewport scaling
   - Supports both desktop mouse and mobile touch controls

2. **Game Engine**
   - Frame-based update loop
   - Entity management (player, enemies, bullets, effects)
   - Collision detection system
   - State management (menu, playing, game over, stage complete)

3. **Player Controls**
   - Mouse/Touch: Move with cursor
   - Click/Tap: Fire weapon
   - Right-click/Long-press: Smart bomb

4. **Game Mechanics**
   - **Weapons System**: Multiple weapon types with power progression
   - **Smart Bombs**: Limited-use explosives that clear enemies
   - **Health System**: Multiple lives with critical state warnings
   - **Scoring**: Kill counters and multipliers
   - **Stages**: Progressive difficulty with boss encounters

### HUD Elements

- **Health**: Heart icons showing remaining lives
- **Bombs**: Bomb icons showing available smart bombs
- **Weapon**: Current weapon type indicator
- **Multiplier**: Score multiplier display
- **Wave/Enemy Count**: Current stage information

## Development

### How to Run

1. Open `index.html` in a modern web browser
2. Click to start the game
3. Use mouse/touch to aim and fire
4. Manage resources and progress through stages

### Browser Requirements

- Modern browser with HTML5 Canvas support (Chrome, Firefox, Safari, Edge)
- ES6+ JavaScript support
- Touch support for mobile devices (optional, fallback to mouse)

### Key Technologies

- **HTML5 Canvas**: Graphics rendering
- **JavaScript (ES6+)**: Game logic
- **CSS3**: Styling, animations, responsive design
- **Touch Events**: Mobile input handling

## Code Organization

The embedded JavaScript follows this structure:

1. **Configuration & Constants**: Game settings, difficulty, stage data
2. **Canvas & Rendering**: Setup, drawing functions, sprite rendering
3. **Entity Classes**: Player, Enemy, Bullet, Effect objects
4. **Game State**: State machine for menu, gameplay, game over, etc.
5. **Input Handling**: Mouse and touch event listeners
6. **Game Loop**: Update and render functions
7. **Initialization**: Game setup and event binding

## Development Notes

### Adding Features

- New weapons: Extend weapon system in player update logic
- New enemy types: Create enemy class with unique behavior
- New stages: Add stage configuration to stage data array
- Visual effects: Add to effect rendering system

### Performance Considerations

- Canvas rendering is optimized for 60 FPS
- Enemy pooling prevents excessive object creation
- Efficient collision detection using bounding boxes
- Mobile optimization with touch-action CSS properties

### Testing

- Manual testing in browser (desktop and mobile)
- Test on different screen sizes
- Verify keyboard and touch controls
- Check stage progression and boss behavior

## Git Workflow

- **Branch**: `claude/inspiring-planck-fafird` (development branch)
- **Remote**: `origin` (GitHub)
- **Main branch**: `main` (stable releases)

### Contributing

1. Develop features on the assigned branch
2. Test thoroughly in browser
3. Commit with clear messages
4. Push changes before creating pull requests

## Future Enhancements

Potential improvements for future development:

- Sound effects and background music
- Additional weapon types and upgrades
- More diverse enemy patterns and AI
- Leaderboard/score persistence
- Level editor or procedural generation
- Difficulty settings/selectable modes
- Visual polish and particle effects

## Resources & Links

- **Game Reference**: Raiden arcade series
- **Documentation**: See code comments in index.html
- **Browser APIs**: Canvas API, Touch Events, Fullscreen API

---

*Last updated: 2026-10-10*
*Maintained by: Claude Code agent*
