# Claude Code Guidelines for Google_4 (Sky Fighter)

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application. The game features:
- Vertical scrolling shoot-'em-up gameplay
- Weapon switching mechanics
- Smart bomb mechanics
- Stage bosses
- Mobile-optimized controls with touch support
- Dark-themed UI with sci-fi aesthetic

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (single file)
├── README.md           # Project metadata
├── .git/               # Git repository
└── CLAUDE.md           # This file - Claude Code guidelines
```

## Key Files

### index.html (Primary Game File)
- **Purpose**: Contains the complete Sky Fighter game implementation
- **Format**: Single HTML file with embedded CSS and JavaScript
- **Size**: ~59KB
- **Technology Stack**: 
  - HTML5 Canvas API for rendering
  - Vanilla JavaScript (no external dependencies)
  - CSS Grid and Flexbox for UI layout
  - Touch and keyboard event handling

## Game Architecture

The game is implemented entirely within index.html with the following main sections:

1. **HTML Structure**: Container div with canvas for game rendering and HUD elements
2. **CSS Styling**: 
   - Dark theme (#05122b background, blue gradients for sky)
   - Responsive design with viewport meta tags for mobile
   - Cross-browser compatibility (webkit prefixes for mobile)
3. **JavaScript Game Engine**:
   - Canvas-based rendering loop
   - Entity management (player, enemies, projectiles, bosses)
   - Physics and collision detection
   - Game state management (menu, gameplay, game over)
   - Audio management (if implemented)

## Development Guidelines

### Code Organization
- **Single File**: All game logic is in index.html to ensure portability and ease of sharing
- **Embedded Resources**: CSS and JavaScript are inline; no external dependencies
- **Responsive Design**: Game adapts to different screen sizes and device orientations

### Mobile Support
- Touch event handling for mobile devices
- Safe area insets (notch support) for mobile displays
- Viewport optimization with maximum-scale, user-scalable=no
- Crosshair cursor for pointer precision

### Performance Considerations
- Canvas rendering optimization
- Efficient entity management and pooling
- Touch action optimization (touch-action: none)
- No external library dependencies

### Browser Compatibility
- Modern browsers with HTML5 Canvas support
- Mobile browsers (iOS Safari, Chrome Android)
- Graceful degradation for older browsers

## Working with This Project

### Testing
- Open `index.html` directly in a web browser
- Test on desktop (keyboard controls)
- Test on mobile devices (touch controls)
- Verify UI responsiveness across screen sizes

### Making Changes
1. Edit the `<script>` section in index.html for game logic changes
2. Edit the `<style>` section for visual/layout changes
3. Update meta tags or HTML structure for UI changes
4. Keep the file as a single consolidated HTML file

### Common Tasks

#### Adding Features
- New weapon types: Add to weapon management logic in JS
- New enemy types: Extend entity system
- Boss battles: Add boss-specific behavior and sprites
- Sound effects: Wire up audio events in game engine

#### Bug Fixes
- Collision detection issues: Check entity overlap logic
- Physics problems: Verify movement calculations
- UI rendering: Check CSS and canvas coordinate system
- Controls: Verify event listeners and input handling

### Code Style
- Use vanilla JavaScript (no external libraries)
- Keep game loop efficient (60 FPS target)
- Document complex game mechanics with comments
- Test frequently on target devices (mobile and desktop)

## Version Control

- **Branch**: Work on feature/fix branches as needed
- **Main Branch**: Contains stable, playable versions
- **Commits**: Include clear messages describing changes
- **Testing**: Always test on mobile and desktop before committing

## Future Enhancement Areas

- Sound/music implementation
- Additional weapon types
- More boss encounters
- Difficulty progression
- Score/leaderboard system
- Pause/resume functionality
- Accessibility improvements

---

*Last Updated: 2026-09-27*
*Maintained by Claude Code agents*
