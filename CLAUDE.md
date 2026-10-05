# Sky Fighter - Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas. It's a single-file web game featuring:

- Vertical scrolling gameplay with enemy waves and stage bosses
- Player ship with weapon switching mechanics
- Smart bomb power-ups
- Score multiplier system
- Health/heart system with damage feedback
- Touch and keyboard controls
- Responsive design with mobile support

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (all HTML, CSS, and JS)
├── README.md           # Project timestamp and basic info
├── CLAUDE.md          # This file - documentation for Claude Code
└── .git/              # Git repository
```

## Technology Stack

- **Frontend**: HTML5, CSS3, Vanilla JavaScript (no frameworks)
- **Graphics**: Canvas 2D API
- **Input**: Keyboard and touch events
- **Deployment**: Static HTML file (no build process required)

## How to Run

1. Open `index.html` in any modern web browser
2. The game runs entirely client-side with no server requirements
3. Responsive design adapts to desktop, tablet, and mobile screens

## Key Game Features

### Gameplay Mechanics

- **Player Ship**: Controlled via arrow keys or WASD (desktop) or touch input (mobile)
- **Weapons**: Switch between rapid-fire, spread shot, and laser weapons
- **Smart Bombs**: Limited-use screen-clearing special attack
- **Enemies**: Progressive waves with increasing difficulty and boss encounters
- **Score System**: Points multiplied based on kill streak
- **Health**: 3-heart system with damage indication

### UI Elements

- **HUD**: Top-of-screen display showing hearts, bombs, weapon type, and score multiplier
- **Vignette Effect**: Red pulsing effect indicates critical health
- **Flash Effects**: Screen flashes on hits and bomb detonations

### Visual Design

- Dark blue gradient background simulating sky
- Modern glassmorphism UI elements with backdrop blur
- Color-coded HUD stats (yellow score, green weapons, orange multiplier)
- Mobile-optimized viewport with safe area insets

## Game Configuration

Key game settings that may be adjusted in `index.html`:

- Canvas size: Dynamically sized to viewport (max 800x600 recommended)
- Difficulty: Progressive enemy wave spawning and boss patterns
- Player speed: Movement velocity and acceleration
- Weapon stats: Fire rates, damage, and spread patterns
- Health system: Heart count and damage amounts

## Development Notes

### Single-File Architecture

All code exists in one HTML file for simplicity:
- HTML structure (container, canvas, HUD elements)
- CSS styles (including responsive design and animations)
- JavaScript game logic (engine, entities, input handling)

This approach:
- ✅ Easy deployment and sharing
- ✅ No build process required
- ✅ Works offline with no dependencies
- ⚠️ Requires refactoring if splitting into modules

### Code Organization (within index.html)

The JavaScript is organized logically:

1. **Game Engine**: Canvas setup, game loop, update/render cycle
2. **Entity System**: Base entity class, player ship, enemies, projectiles
3. **Weapon System**: Different weapon types and firing logic
4. **Collision Detection**: Hit detection between entities
5. **Input Handling**: Keyboard and touch event listeners
6. **UI/HUD**: Score, health, and weapon display
7. **Game State**: Win/lose conditions, score tracking

### Canvas Scaling and Responsive Design

- Canvas scales to fit viewport while maintaining aspect ratio
- Touch controls use viewport-relative coordinates
- CSS `max-width: 100%; max-height: 100%` ensures responsive scaling
- Safe area insets support notched devices

## Common Tasks

### Adding New Enemy Types

1. Extend entity system with new enemy class
2. Define movement pattern and AI behavior
3. Add spawn logic to wave system
4. Adjust difficulty scaling

### Adjusting Difficulty

- Enemy spawn rate in game loop
- Enemy health/damage values
- Boss pattern complexity
- Wave progression speed

### Adding New Weapons

1. Create weapon object with fire rate, damage, spread pattern
2. Add weapon switching logic to input handler
3. Update HUD display for new weapon
4. Balance damage vs. fire rate vs. resource cost

### Mobile Optimization

- Test touch controls across different devices
- Verify viewport scaling on tablets
- Check safe area insets for notched phones
- Ensure HUD readability on small screens

## Testing Checklist

- [ ] Game starts and displays correctly
- [ ] Player can move with keyboard (arrow keys/WASD)
- [ ] Player can move with touch input (drag to move)
- [ ] Weapons fire and damage enemies
- [ ] Weapon switching works (cycle through types)
- [ ] Enemies spawn in waves with increasing difficulty
- [ ] Boss encounters occur at stage end
- [ ] Smart bombs work and clear screen
- [ ] Score increases and multiplier tracks kills
- [ ] Health system works (damage reduces hearts)
- [ ] Game ends on defeat and shows final score
- [ ] Game responds to victory conditions
- [ ] Responsive design works on various screen sizes
- [ ] Audio/visual feedback is clear (if sound effects added)

## Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires Canvas 2D API support
- Touch event support for mobile devices
- CSS backdrop-filter for HUD blur effect (gracefully degrades)

## Performance Considerations

- Canvas is rendered every frame (~60 FPS target)
- Entity pooling could improve performance with many enemies
- Current implementation uses simple spatial hashing for collision detection
- For large entity counts, consider quadtree or grid-based optimization

## Future Enhancement Ideas

- Sound effects and background music
- Particle effects for explosions and weapon fire
- Additional weapon types and power-ups
- More complex boss patterns
- Leaderboard with local storage
- Different difficulty modes
- Cosmetic ship/weapon skins
- Stage/level progression system

## Attribution

- Developed with Claude Code assistance
- Game design inspired by classic Raiden series
- Built as a single-file HTML5 game for maximum accessibility

## Git Workflow

- Main branch: `main` (stable releases)
- Development: Feature branches for new content/fixes
- Commits should describe gameplay or technical changes clearly
- Keep HTML file as single source of truth

---

*Last updated: 2026-10-05*
