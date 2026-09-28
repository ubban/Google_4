# Claude Development Guide for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:
- Player-controlled spaceship with weapon switching mechanics
- Enemy AI and boss encounters
- Smart bomb mechanics for crowd control
- Score multiplier system
- Responsive touch and keyboard controls

## Technology Stack

- **Frontend**: Vanilla HTML5, CSS3, Canvas API
- **Game Engine**: Custom JavaScript game loop
- **Platform**: Web browser (desktop and mobile)
- **No Build Tools**: Single HTML file with embedded styles and scripts

## Project Structure

```
Google_4/
├── index.html        # Main game file (complete application)
├── README.md         # Project metadata
└── Claude.md         # This development guide
```

## Key Development Guidelines

### 1. Single File Architecture
The entire game is contained in `index.html`. When making changes:
- Keep all CSS in the `<style>` section
- Keep all JavaScript in the final `<script>` section
- Maintain clear separation between logic layers

### 2. Game State Management
- **Player state**: Position, health, weapons, bomb count
- **Game state**: Score, wave number, difficulty, pause state
- **Entities**: Enemies, bullets, explosions
- Store state in the game controller object

### 3. Canvas Rendering Pipeline
1. Clear canvas with background gradient
2. Update entity positions and states
3. Render entities (player, enemies, bullets, effects)
4. Render HUD overlay
5. Apply post-process effects (vignette, flash)

### 4. Input Handling
- **Keyboard**: WASD for movement, Space for shoot, E for weapon switch, Q for bomb
- **Touch/Mobile**: On-screen controls with fallback mouse aim
- **Accessibility**: Consider touch-friendly hit zones

### 5. Game Loop Timing
- Use `requestAnimationFrame` for smooth 60fps rendering
- Decouple game logic from rendering when needed
- Handle variable frame rates gracefully

## Common Development Tasks

### Adding New Enemy Types
1. Define enemy class with properties: position, health, speed, behavior
2. Implement movement pattern and attack logic
3. Create spawn logic in wave generation
4. Add collision and damage handling

### Modifying Weapons
1. Update weapon data in the weapons system
2. Adjust bullet spawn position and velocity
3. Modify fire rate and damage values
4. Test weapon balancing with different enemy waves

### Adjusting Game Difficulty
- Enemy spawn rate and wave progression
- Boss health and attack patterns
- Score multiplier mechanics
- Bomb availability

### Performance Optimization
- Use object pooling for frequently created entities (bullets, enemies)
- Minimize canvas state changes
- Batch render calls when possible
- Profile with browser DevTools Performance tab

## Testing & Validation

### Manual Testing Checklist
- [ ] Game starts and runs without errors
- [ ] Player movement responds correctly (keyboard + touch)
- [ ] Weapons fire and hit enemies
- [ ] Health system works (damage, death, revival)
- [ ] Bomb mechanics function correctly
- [ ] Score and multipliers update properly
- [ ] Enemy waves progress as expected
- [ ] Boss encounters work
- [ ] Game over / win states trigger correctly
- [ ] HUD displays accurate information
- [ ] Responsive on mobile devices
- [ ] No memory leaks over extended play

### Browser Compatibility
- Chrome/Edge (primary)
- Firefox
- Safari (desktop and iOS)
- Mobile browsers (Chrome, Safari)

### Performance Targets
- Maintain 60fps on desktop
- Maintain 30fps minimum on mobile
- No frame drops during heavy action

## Code Style & Conventions

- Use clear, descriptive variable and function names
- Group related code together (entity classes, game logic, rendering)
- Add comments for non-obvious game mechanics or complex algorithms
- Use consistent indentation (2 spaces)
- Prefer vanilla JavaScript over external libraries

## Debugging Tips

- Use browser console for logging game state
- Check Canvas context state (fill style, stroke, etc.)
- Use DevTools Performance tab to identify bottlenecks
- Enable requestAnimationFrame in DevTools to see frame timing
- Test with different screen sizes and DPR values

## Security Considerations

- Input validation for all user actions
- Prevent accidental player state manipulation
- Be mindful of timing attacks in multiplayer context
- Keep client-side score validation for offline play

## Git Workflow

- Develop on feature branches: `claude/*`
- Use clear, descriptive commit messages
- Test thoroughly before pushing changes
- Link commits to related issues where applicable

## Updating This Guide

- Update this Claude.md when adding new major features
- Document new mechanics and design decisions
- Keep sections up-to-date with actual codebase structure
- Record performance baselines and optimization notes
