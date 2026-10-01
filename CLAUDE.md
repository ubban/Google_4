# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. It features classic arcade gameplay with weapon switching, smart bombs, stage bosses, and progressive difficulty.

## Architecture

The entire game is contained in a single `index.html` file with:
- **HTML structure**: A canvas container and HUD elements for displaying score, lives, ammo, etc.
- **CSS styling**: Responsive dark theme with gradient backgrounds, safe-area-inset support for mobile
- **JavaScript game engine**: Canvas-based 2D rendering with collision detection, sprite management, and game state

### Key Game Systems

1. **Player Ship**: Positioned at bottom-center, controlled via mouse/touch/keyboard
2. **Weapons**: Multiple weapon types with ammo management (normal bullets, spread shots, lasers, etc.)
3. **Enemies**: Wave-based spawning with different enemy types and behaviors
4. **Projectiles**: Player bullets and enemy fire with collision detection
5. **Explosions & Effects**: Visual feedback for impacts and destroyed enemies
6. **Scoring System**: Points based on enemies destroyed, with combo multipliers
7. **Stage/Boss System**: Progressive difficulty with boss encounters

## File Structure

```
Google_4/
├── index.html        # Complete game (HTML + CSS + JS)
├── README.md         # Project overview
├── CLAUDE.md         # This file
└── .git/             # Git repository
```

## How to Run

1. **Local**: Open `index.html` directly in a modern web browser
2. **Server**: Serve via any HTTP server (e.g., `python -m http.server`)
3. **Web**: Deploy to any static hosting (GitHub Pages, Netlify, etc.)

## Game Controls

- **Mouse/Touch**: Move and aim
- **Left Click/Touch**: Shoot
- **Space / Right Click**: Switch weapon
- **E / Double-tap**: Deploy smart bomb (limited uses)
- **Arrow Keys** (optional): Alternative movement controls

## Development Notes

### Code Organization
The `index.html` file is organized with:
- Meta tags and external resources (favicon, viewport, etc.)
- CSS stylesheet (embedded)
- HTML structure (canvas + HUD)
- JavaScript game code (organized into logical sections)

### Browser Compatibility
- Requires modern browser with HTML5 Canvas support
- Touch events supported for mobile gameplay
- Uses CSS Grid and Flexbox for responsive layout
- Safe-area-inset for notch/safe area support on mobile

### Performance Considerations
- Single-threaded Canvas rendering at 60 FPS target
- Object pooling recommended for bullets and explosions at scale
- Collision detection uses simple rectangle/circle bounds
- No external dependencies - entirely self-contained

### Asset Strategy
- No external image files - all graphics rendered via Canvas
- All sprites and effects generated procedurally
- Icon created as inline SVG data URI in favicon

## Common Development Tasks

### Adding New Enemy Types
1. Define enemy object in the enemy spawning section
2. Implement movement pattern in the game loop
3. Add collision handling with player bullets
4. Update scoring for defeated enemies

### Adjusting Game Balance
- Enemy spawn rates: Modify wave configuration
- Weapon damage: Adjust bullet properties
- Player movement speed: Update player.speed constant
- Difficulty progression: Modify stage completion conditions

### Adding New Weapon Types
1. Add weapon config to player.weapons array
2. Implement firing pattern in weapon switching logic
3. Add ammo management for the new weapon
4. Update HUD display if needed

## Testing

- **Manual testing**: Open in browser and play through stages
- **Cross-browser**: Test on Chrome, Firefox, Safari, and mobile browsers
- **Mobile**: Test touch controls and layout on various screen sizes
- **Performance**: Monitor frame rate (should target 60 FPS)

## Deployment

1. Push changes to designated branch
2. Deploy `index.html` to hosting
3. Test on target platforms (desktop, mobile, tablets)
4. Monitor for performance or compatibility issues

## Future Considerations

- Extract game logic into separate module files for better maintainability
- Implement state management system for save/load functionality
- Add sound effects and music (audio API)
- Create level editor for custom stages
- Add leaderboard/highscore persistence (localStorage/backend)
- Mobile app wrapper (React Native, Flutter, or native)

## Attribution

This project uses single-file design for maximum portability. All code is self-contained in `index.html` for easy deployment and modification.
