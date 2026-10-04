# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely with HTML5 Canvas and vanilla JavaScript. The game features dynamic weapon switching, smart bomb mechanics, stage-based progression, and boss encounters with progressive difficulty.

**Repository:** ubban/Google_4  
**Primary File:** `index.html` (1669 lines, single-file monolithic structure)  
**Tech Stack:** HTML5 Canvas, Vanilla JavaScript (ES6+), CSS3

## Game Features

- **Vertical Scrolling Gameplay:** Classic top-down shooting mechanics
- **Weapon System:** Multiple weapon types with switching capability
- **Power-ups:** Health, weapon upgrades, multiplier boosts
- **Smart Bombs:** Special offensive ability with cooldown mechanics
- **Stage-Based Progression:** Multiple levels with increasing difficulty
- **Boss Encounters:** Unique boss patterns per stage
- **Score & Multiplier:** Dynamic scoring with multiplier tracking
- **Touch & Keyboard Support:** Mobile and desktop controls
- **HUD System:** Real-time display of health, bombs, weapon, and multiplier stats
- **Visual Effects:** Hit flash, vignette damage indicator, particle effects

## Code Architecture

### Structure
The codebase uses a monolithic single-file approach with clear separation of concerns through:
- **40 Classes:** Game entities and managers
- **52 Functions:** Utility, calculation, and helper functions
- **148 Constants:** Configuration values for game balance, dimensions, and thresholds

### Major Component Categories

#### Game State & Manager Classes
- `Game` - Main game loop controller
- `Player` - Player character with health and weapon management
- `Enemy` - Individual enemy units
- `Boss` - Boss entity with health bar and unique attack patterns
- `Particle` - Visual effects and debris

#### Rendering & Input
- Canvas-based 2D rendering
- Keyboard input handling (WASD, Arrow keys)
- Touch/mouse support for mobile
- HUD rendering with stats display

#### Game Systems
- **Weapon System:** Different weapon types with unique firing patterns
- **Power-up System:** Health, weapon, and multiplier pickups
- **Collision Detection:** Player/enemy, projectile/enemy, player/pickup
- **Scoring:** Point calculation with multiplier
- **Difficulty Scaling:** Progressive enemy patterns and boss attacks

### Key Constants & Config
- **Canvas Dimensions:** Responsive to container
- **Physics:** Speed, acceleration, collision radii
- **Enemy Patterns:** Spawn rates, behavior configurations
- **Boss Data:** Health pools, attack patterns, rewards
- **Weapon Stats:** Damage, fire rate, projectile speed

## Development Workflow

### Setup
1. Clone the repository
2. Open `index.html` in a modern web browser
3. Game starts automatically on page load

### Running & Testing
- No build step required
- All code is in `index.html`
- Open in any browser with Canvas support (Chrome, Firefox, Safari, Edge)
- Mobile testing: Open on mobile device or emulate in browser DevTools

### Making Changes
1. Edit `index.html` directly
2. Refresh browser to see changes
3. Use browser DevTools console for debugging
4. Test across different screen sizes/devices

## File Organization

```
/home/user/Google_4/
├── index.html        (Main game - 1669 lines)
├── README.md         (Basic project info)
└── .git/             (Git repository)
```

## Git Workflow

- **Current Branch:** `claude/inspiring-planck-udw7wd`
- **Commits:** 
  - `c4dc8f6` - Add Sky Fighter: Raiden-style vertical shoot-em-up game
  - `4efdad1` - Initial commit
- **Push Strategy:** Use `git push -u origin <branch-name>`
- **PR Template:** Check `.github/pull_request_template.md` if creating PRs

## Common Tasks

### Adding a New Weapon Type
1. Define weapon properties in weapon configuration section
2. Create weapon firing logic in `Player` class
3. Update weapon switching logic
4. Test fire rate and damage

### Adjusting Game Difficulty
1. Modify enemy spawn rates in Game class
2. Adjust boss health and attack patterns
3. Tweak projectile speeds and patterns
4. Balance power-up drop rates

### Tweaking Game Balance
- Enemy health/damage in `ENEMY_*` constants
- Weapon damage values in weapon definitions
- Player speed and invulnerability timing
- Score multiplier thresholds

### Adding Visual Effects
1. Create new Particle subclass if needed
2. Instantiate particles at effect location
3. Update particle rendering and animation
4. Adjust emitter properties (size, duration, color)

## Performance Considerations

- **Canvas Rendering:** Optimized 2D rendering at ~60 FPS
- **Entity Management:** Dynamic spawning/despawning to reduce draw calls
- **Collision Detection:** Efficient spatial checks for entity interactions
- **Memory:** Pooling pattern for frequently created objects (particles, projectiles)

## Browser Compatibility

- Chrome/Chromium 80+
- Firefox 75+
- Safari 13+
- Edge 80+
- Mobile browsers (iOS Safari, Chrome Mobile)

## Known Limitations

1. Single HTML file structure - consider splitting for larger projects
2. No persistent storage (scores not saved between sessions)
3. Monolithic game loop - state management could benefit from refactoring
4. Limited audio (no sound effects or music implemented)

## Future Enhancement Ideas

1. Sound effects and background music
2. Persistent high score tracking (localStorage)
3. More varied enemy types and patterns
4. Additional weapon types and special abilities
5. Difficulty modes (easy, normal, hard)
6. Pause and resume functionality
7. Mobile-optimized touch controls
8. Leaderboard system

## Code Style & Conventions

- **ES6+ Features:** Classes, const/let, arrow functions
- **Naming:** camelCase for variables/functions, PascalCase for classes
- **Comments:** Minimal - code is self-documenting through clear naming
- **Indentation:** 2 spaces

## Testing Notes

### Gameplay Testing Checklist
- [ ] Player movement works in all directions
- [ ] Weapon switching changes bullet type and behavior
- [ ] Enemy spawning follows intended patterns
- [ ] Boss encounters trigger at correct stage
- [ ] Power-ups drop and provide expected benefits
- [ ] Collision detection is accurate
- [ ] HUD displays correct stats
- [ ] Score calculation and multiplier work
- [ ] Bombs activate and damage enemies
- [ ] Game handles mobile/touch input
- [ ] Game handles keyboard input

## Attribution & Authorship

This codebase was created and maintained by Claude Code agents. Development focused on creating a complete, playable vertical shoot-em-up game with modern web technologies.

---

**Last Updated:** 2026-10-04  
**Version:** 1.0 (Initial release with Raiden-style gameplay)
