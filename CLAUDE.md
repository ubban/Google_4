# Sky Fighter - CLAUDE.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas and JavaScript. Players control a fighter aircraft to shoot down enemies, switch weapons, drop smart bombs, and defeat stage bosses.

**Repository:** ubban/google_4  
**Development Branch:** claude/inspiring-planck-emq630

## Project Structure

```
Google_4/
├── index.html       # Single-file game implementation (HTML + CSS + JavaScript)
├── README.md        # Basic project metadata
└── .git/            # Git repository
```

## Key Features

- **Vertical Scrolling Shoot-em-up Gameplay**: Classic Raiden-style mechanics
- **Weapon System**: Multiple weapons that can be switched during gameplay
- **Smart Bombs**: Special bombs that clear enemy waves
- **Boss Battles**: Challenging stage bosses with unique patterns
- **Responsive Controls**: Keyboard and mouse/touch input support
- **HUD Display**: Real-time stats showing health, weapon type, kills, and stage progress
- **Dark Space Theme**: Visual design with gradient backgrounds and detailed sprites

## Technical Details

### Architecture
- **Single-file implementation**: All game logic, rendering, and styling in one index.html file
- **Canvas-based rendering**: Uses HTML5 Canvas 2D context for all graphics
- **Game loop**: RequestAnimationFrame for smooth 60fps gameplay
- **Event-driven input**: Keyboard events for movement and actions, mouse/touch for aiming

### Key Game Systems
1. **Player Controller**: Handles aircraft movement, weapon firing, bomb usage
2. **Enemy Management**: Spawns enemies in waves with varying patterns
3. **Collision Detection**: Detects player-bullet, enemy-bullet, and item collisions
4. **Physics**: Gravity, velocity-based movement, bullet trajectories
5. **UI/HUD**: Score, health, weapon indicator, stage information

### Controls
- **Arrow Keys / WASD**: Move aircraft
- **Space**: Fire primary weapon
- **Shift**: Drop smart bomb
- **Mouse/Touch**: Aim direction (if supported)

## Development Notes

### Styling & Theming
- Uses CSS custom properties for dark theme
- Dark background (#05122b) with gradient game area
- Safe area insets for mobile/notch support
- Responsive sizing with `clamp()` for fluid layouts

### Performance Considerations
- Single canvas element for all rendering
- Efficient sprite drawing and collision detection
- RequestAnimationFrame for optimal rendering cadence
- Touch action disabled to prevent browser defaults

### Browser Compatibility
- Requires modern browser with Canvas 2D support
- Works on desktop (keyboard) and mobile (touch) devices
- Viewport-fit=cover for notch support on mobile

## How to Run

1. Open `index.html` in a modern web browser
2. Game will start automatically
3. Use controls to play and defeat enemies

## Git Workflow

- Working branch: `claude/inspiring-planck-emq630`
- Commits should be meaningful and describe gameplay or feature additions
- Include co-author attribution in commits

## Future Enhancement Ideas

- Persistent high score tracking (localStorage)
- Sound effects and background music
- Additional enemy types and boss patterns
- Power-up items (health recovery, weapon upgrades)
- Progressive difficulty scaling
- Mobile app packaging (React Native, Electron)
- Leaderboard system
- Tutorial/difficulty levels

## Code Style Guidelines

- Keep game logic and rendering separated where practical
- Use descriptive variable and function names
- Add comments for complex algorithms (collision detection, AI patterns)
- Minimize external dependencies to keep single-file implementation
- Test gameplay changes in multiple browsers and device sizes

---

*Last updated: 2026-10-16*
