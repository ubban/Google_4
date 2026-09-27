# Claude Code Guidance for Google_4 Project

## Project Overview

**Sky Fighter** — a Raiden-style vertical shoot-em-up game implemented as a single-page HTML5 application. The game features player-controlled aircraft, weapon switching, smart bombs, enemy waves, and boss encounters.

**Repository**: `ubban/google_4`  
**Primary Branch**: `claude/inspiring-planck-x9ior9`  
**Current State**: Active development with core game loop implemented

## Architecture & Key Files

### Main Files
- **index.html** (58KB) — Complete game implementation including:
  - Canvas-based rendering engine
  - Game state management
  - Player ship controller
  - Enemy spawning and AI
  - Collision detection
  - Weapon systems
  - UI/HUD

### Project Structure
```
Google_4/
├── index.html          # Single-file game implementation
├── README.md           # Project metadata
├── Claude.md           # This file — development guidance
└── .git/               # Version control
```

## Development Guidelines for Claude Code Agents

### Code Style & Conventions
- **Format**: Single HTML file with embedded CSS and JavaScript
- **Language**: Vanilla JavaScript (no frameworks)
- **Canvas API**: Used for all rendering
- **Performance**: Optimized for 60 FPS mobile/desktop gameplay
- **Mobile-First**: Support touch controls with fallback to keyboard

### Game Systems Architecture
The game is organized into logical systems:

1. **Rendering Pipeline**
   - Canvas setup with device pixel ratio scaling
   - Sprite drawing and animation frames
   - HUD text rendering with safe area insets

2. **Game Loop**
   - Update: Player input, entity positions, collisions
   - Render: Draw to canvas
   - 60 FPS target via requestAnimationFrame

3. **Entity System**
   - Player ship with health/powerups
   - Enemy types with individual behavior patterns
   - Projectiles (player bullets, enemy fire)
   - Explosions and visual effects

4. **Weapon System**
   - Multiple weapon types with different fire patterns
   - Ammo management
   - Weapon switching via UI

5. **Collision Detection**
   - Rectangle-based AABB collision checks
   - Projectile-entity collisions
   - Boundary wrapping for entities

6. **Input Handling**
   - Keyboard controls (Arrow keys, Z/X for weapons, C for bomb)
   - Touch controls with position tracking
   - Mouse aiming for desktop players

### Common Tasks & Approaches

#### Adding a New Enemy Type
1. Define enemy parameters (health, speed, spawn pattern)
2. Create spawn function in enemy spawning section
3. Add behavior logic in enemy update loop
4. Implement animation frames
5. Test collision and damage behavior
6. Balance difficulty progression

#### Modifying Weapon Behavior
1. Locate weapon fire pattern in player update section
2. Adjust fire rate, damage, or projectile velocity
3. Update ammo consumption if needed
4. Test feel and balance
5. Verify weapon switching works correctly

#### UI/HUD Changes
1. Update HUD DOM element structure if needed
2. Modify CSS in style block (clamp() for responsive sizing)
3. Update value display in game loop render section
4. Ensure safe area insets respected for notched devices

#### Performance Optimization
- Monitor entity count (keep under 1000 active)
- Profile canvas drawing operations
- Use requestAnimationFrame for smooth rendering
- Cache DOM element references
- Minimize reflow/repaint operations

### Testing & Validation

#### Gameplay Testing Checklist
- [ ] Player movement responds smoothly
- [ ] Weapon firing works on all weapon types
- [ ] Enemy spawning matches difficulty curve
- [ ] Collision detection is accurate
- [ ] Boss encounters are challenging but fair
- [ ] Mobile touch controls are responsive
- [ ] Desktop keyboard controls work properly
- [ ] Game runs at consistent frame rate
- [ ] No memory leaks on extended play

#### Cross-Platform Testing
- Desktop: Chrome, Firefox, Safari
- Mobile: iOS Safari, Chrome mobile, Samsung Internet
- Tablet: iPad, Android tablets
- Test with various screen sizes and aspect ratios

### Branching & Commits

- **Branch**: Always work on `claude/inspiring-planck-x9ior9`
- **Commits**: Use clear, descriptive messages referencing the feature/fix
- **Push**: After meaningful changes, push with `-u origin claude/inspiring-planck-x9ior9`
- **PR**: Follow GitHub PR template if creating pull requests

### Game Design Principles

1. **Difficulty Curve**: Gradually increase enemy complexity and spawn rate
2. **Feedback**: Visual/audio feedback for player actions (hits, weapon fire, damage)
3. **Pacing**: Mix intense combat with breathing room
4. **Balance**: Ensure all weapons are viable, no overpowered strategies
5. **Mobile Experience**: Ensure playable on smaller screens without cramped controls

## Common Pitfalls to Avoid

- ❌ Don't break the existing game loop structure
- ❌ Don't introduce external dependencies (keep vanilla JS)
- ❌ Don't optimize prematurely — profile first
- ❌ Don't forget mobile testing after desktop changes
- ❌ Don't leave console errors or warnings
- ❌ Don't hardcode values that should be configurable

## Resources & References

### Canvas API
- MDN Canvas Documentation: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game loop patterns and requestAnimationFrame

### Game Development
- Collision detection algorithms (AABB, SAT)
- Entity-component patterns
- Sprite animation techniques

### Mobile Optimization
- Touch event handling and preventDefault
- Device pixel ratio scaling
- Safe area insets (notched devices)

## Future Enhancement Ideas

- [ ] Sound effects and music system
- [ ] Persistent highscore tracking (localStorage)
- [ ] Additional boss encounters
- [ ] Power-up variations
- [ ] Difficulty settings
- [ ] Leaderboard integration
- [ ] Mobile app wrapper (Cordova/Capacitor)
- [ ] Procedural enemy pattern generation

## Questions & Decisions

When working on this codebase:

1. **Performance vs Features** — Profile before optimizing; avoid premature optimization
2. **Code Organization** — Keep inline; resist urge to extract separate files
3. **Game Balance** — Test thoroughly; get feedback from multiple testers
4. **Accessibility** — Ensure controls are remappable and gameplay is colorblind-friendly

---

**Last Updated**: 2026-09-27  
**Maintained By**: Claude Code Agent  
**Version**: 1.0
