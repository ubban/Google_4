# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. Players control an aircraft, switch between weapons, drop smart bombs, and defeat stage bosses in an arcade-style gameplay experience.

## Technical Stack

- **Runtime**: HTML5 Canvas with vanilla JavaScript
- **Platform**: Browser-based (responsive to mobile and desktop)
- **Dependencies**: None (pure vanilla JavaScript, no frameworks)
- **Key Technologies**:
  - HTML5 Canvas for rendering
  - Touch and mouse input handling
  - CSS Grid/Flexbox for layout and HUD
  - CSS animations for visual effects (vignette, flash, heart beat)

## Repository Structure

```
Google_4/
├── index.html          # Main application file (contains all HTML, CSS, and JavaScript)
├── README.md          # Project timestamp log
├── .git/              # Git repository metadata
└── Claude.md          # This file - codebase documentation
```

## File Details

### index.html (58KB)
The single comprehensive file containing the entire application:

**HTML Structure:**
- `<div id="container">` - Main game container
- `<canvas id="gameCanvas">` - Game rendering surface
- `<div id="hud">` - Heads-up display for game stats
  - Health indicator (#hearts)
  - Bomb counter (#bombs)
  - Weapon display (#weaponStat)
  - Score/multiplier (#multStat)
- `<div id="vignette">` - Damage/alert visual effect
- `<div id="flash">` - Hit feedback animation

**CSS Features:**
- Dark theme (color-scheme: dark, background: #05122b)
- Responsive canvas sizing with aspect ratio preservation
- HUD elements with glassmorphism (backdrop-filter: blur)
- Animated visual feedback (heart beat on critical health, vignette pulse)
- Mobile-safe layout (env(safe-area-inset-top))
- Touch-action: none for full touch control

**JavaScript Game Engine:**
The file contains the complete game implementation including:
- Player aircraft control (movement, weapon switching, bombing)
- Enemy spawning and AI behavior
- Weapon systems (multiple weapon types)
- Collision detection
- Score/multiplier tracking
- Boss encounters
- Stage progression
- Input handling (mouse, touch, keyboard)
- Particle effects and animations
- Game state management (menu, playing, game over, victory)

## Key Game Mechanics

1. **Player Control**: Move with mouse/touch, click/tap to shoot, space/double-tap for bombs
2. **Weapons**: Switch between different weapon types with unique fire patterns
3. **Power-ups**: Collect items to upgrade weapons and special abilities
4. **Health System**: Track player health with visual feedback
5. **Bombs**: Limited smart bombs that clear large areas of enemies
6. **Score System**: Multiplier increases with consecutive hits
7. **Stage Bosses**: Unique boss encounters at stage end
8. **Progression**: Multiple stages with increasing difficulty

## Development Notes

### Styling System
- CSS custom properties (:root) for theme consistency
- Responsive sizing using CSS clamp() for scalability
- Glass morphism effects with rgba and backdrop-filter
- Smooth animations using @keyframes

### Responsive Design
- Max width/height constraints on canvas
- Viewport configuration for mobile
- Touch-action: none for optimal touch control
- Safe area support for notched devices

### Performance Considerations
- Single canvas rendering pipeline
- Touch highlight disabled for better UX
- User select disabled during gameplay
- Efficient animation using CSS transforms and opacity

## Build and Deployment

No build process required. Simply serve the `index.html` file via HTTP/HTTPS.

**Deployment:**
1. Commit changes to the designated branch
2. Push to remote repository
3. Server serves the HTML file statically

## Git Workflow

- **Main Branch**: Default production branch
- **Feature Branch**: `claude/inspiring-planck-t8o4to` (development branch)
- All feature development occurs on the feature branch
- Changes are committed with descriptive messages
- Pushed to remote for integration

## Future Enhancement Areas

- Difficulty settings and progression balance
- Additional weapon types and power-ups
- More stages and varied enemy patterns
- Leaderboard/high score persistence
- Sound effects and background music
- Performance optimizations for lower-end devices
- Accessibility improvements (keyboard navigation, ARIA labels)

## Development Guidelines

### Code Organization
- All game code is inline in index.html for simplicity
- Clear separation of concerns through function organization
- CSS organized by component (container, canvas, HUD, effects)

### Modification Tips
- Keep CSS variables at :root for consistent theming
- Maintain touch-friendly event handlers
- Test responsive behavior on multiple viewports
- Verify canvas rendering on different browsers

### Common Tasks
- **Adjust difficulty**: Modify enemy spawn rates and health values
- **Add new weapon**: Update weapon system in JavaScript section
- **Change colors**: Update CSS custom properties or direct color values
- **Add effects**: Create new @keyframes animations and apply them
