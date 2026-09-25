# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game. Players control an aircraft, switch weapons, drop smart bombs, and defeat stage bosses.

- **Type**: HTML5 Canvas Game
- **Technology**: Single HTML file with embedded CSS and JavaScript
- **Platform**: Web browser (responsive design)
- **Features**: Weapon switching, smart bomb system, boss battles, score tracking, health/bomb HUD

## File Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project metadata
├── Claude.md           # This documentation
└── .git/               # Git repository
```

## Game Architecture

### Core Components

The game is built as a single-file HTML application with three main sections:

1. **HTML Structure**: Minimal DOM with canvas and HUD elements
   - `#container`: Main game container
   - `#gameCanvas`: Canvas element for rendering
   - `#hud`: Game UI (health, score, weapon, multiplier)
   - `#vignette`: Screen damage effect
   - `#flash`: Hit/bomb flash effect

2. **CSS Styling**: Responsive design with:
   - Dark space theme (#05122b, #0b1d3a)
   - Dynamic HUD with glassmorphism effect
   - Canvas gradient background
   - Animations: heartBeat, vignettePulse, flashHit, flashBomb

3. **JavaScript Game Engine**: Canvas-based game loop with:
   - Player aircraft control
   - Enemy spawning and AI
   - Collision detection
   - Weapon system
   - Score/multiplier tracking
   - Visual effects (explosions, vignette, flash)

### Game Mechanics

- **Player Control**: Mouse/touch positioning for aircraft aim
- **Weapons**: Multiple weapon types with switching capability
- **Bombs**: Smart bomb system with limited count
- **Health**: Heart-based health system with visual feedback
- **Score**: Multiplier-based scoring system

## Development Guidelines

### Code Style

- Single monolithic JavaScript structure for simplicity
- Use descriptive variable names for game entities
- Keep canvas rendering efficient (single requestAnimationFrame loop)
- Maintain mobile responsiveness with clamp() and relative units

### Making Changes

1. **Game Logic Modifications**: Edit JavaScript section near end of file
2. **Visual Styling**: Modify CSS within `<style>` tag
3. **HUD/UI Updates**: Adjust HTML structure in body and corresponding JavaScript handlers
4. **Testing**: Open `index.html` in browser; test on multiple screen sizes

### Performance Considerations

- Canvas rendering is the primary performance bottleneck
- Minimize redraws; use efficient collision detection
- Optimize particle effects for mobile devices
- Use requestAnimationFrame for smooth 60 FPS gameplay

### Responsive Design

- Uses `clamp()` for scalable font sizes
- `max-width`/`max-height` on canvas for different screen ratios
- Safe area insets for notched devices
- Touch-action CSS for mobile optimization

## Deployment

- Static HTML file - copy to any web server
- No build process or dependencies required
- Works offline (no external resources)
- MIME types: text/html only

## Browser Compatibility

- Modern browsers with Canvas API support
- Touch events for mobile/tablet
- CSS Grid, Flexbox support required
- requestAnimationFrame for animation

## Future Enhancement Areas

- Multi-stage progression
- Boss battle AI improvements
- Sound effects and music
- Difficulty scaling
- Leaderboard/high score storage
- Mobile-specific optimization

## Testing Checklist

When making changes:
- [ ] Game loads without console errors
- [ ] Player can move and fire weapons
- [ ] Enemies spawn and move correctly
- [ ] Collision detection works (bullets, enemies, bombs)
- [ ] HUD displays accurately (health, score, weapon, bombs)
- [ ] Responsive on desktop, tablet, mobile
- [ ] Touch controls work on mobile devices
- [ ] Visual effects display correctly (flashes, vignette)

---

*Last updated: 2026-09-25*
