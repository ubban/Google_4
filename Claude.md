# Claude.md — Sky Fighter Project Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:
- Player-controlled aircraft with weapon switching
- Smart bombs for area damage
- Enemy waves and stage bosses
- Score multiplier system
- Health/lives system with critical state warnings
- Responsive design for desktop and mobile

## Codebase Structure

### Main Files
- **index.html**: Single-page application containing HTML structure, CSS styling, and embedded JavaScript game code
- **README.md**: Project metadata and timestamp
- **.git/**: Version control directory

## Key Game Systems

### Player Mechanics
- **Health System**: Lives displayed as heart icons (❤️), critical state triggers visual warning effects
- **Weapons**: Multiple weapon types with visual indicators; player can switch weapons
- **Smart Bombs**: Limited-use area damage attacks (💣), displayed in HUD
- **Movement**: Mouse/touch-based aiming and movement control
- **Crosshair Cursor**: Visual feedback for targeting

### Visual Effects
- **Vignette Effect**: Red damage indicator that pulses when player takes damage
- **Flash Effects**: White screen flashes for hit feedback (player damage) and bomb explosions
- **HUD Display**: Game statistics (health, bombs, current weapon, score multiplier) with semi-transparent styled pills
- **Canvas Gradient Background**: Blue sky gradient effect

### Game State
- **Multiplier System**: Score multiplier that changes color (orange #ff8a65) in HUD
- **Stage Progression**: Waves of enemies with boss encounters
- **Collision Detection**: Player, enemies, projectiles, and bombs

## Development Guidelines

### Code Style
- Vanilla JavaScript (no frameworks)
- Canvas API for rendering
- Event-driven architecture for input handling (mouse, touch, keyboard)
- CSS Grid/Flexbox for HUD layout with backdrop filters and transforms

### When Modifying Code
1. **Game Logic Changes**: Ensure collision detection, scoring, and state management remain consistent
2. **Visual Changes**: Test responsive design (mobile, tablet, desktop via viewport meta tag)
3. **Canvas Rendering**: Remember canvas rendering order (backgrounds drawn first, then game objects)
4. **Touch Support**: Game supports both mouse and touch input; test both interaction modes
5. **Performance**: Canvas operations should remain performant on lower-end devices

### Color Scheme
- **Background**: Dark blue (#05122b, #0b1d3a, #0d2f5e)
- **Accents**: 
  - Score: Gold (#ffd54f)
  - Weapon: Lime green (#7CFC00)
  - Multiplier: Orange (#ff8a65)
  - Vignette/Damage: Red
  - Border**: Semi-transparent white (rgba(255,255,255,0.25))

## Testing Checklist

When making changes, verify:
- [ ] Game initializes without console errors
- [ ] Player can move and aim with mouse/touch
- [ ] Weapons work and can be switched
- [ ] Enemies spawn and move correctly
- [ ] Collision detection registers hits properly
- [ ] HUD displays accurate stats
- [ ] Health system works (damage, critical state, game over)
- [ ] Bombs function and explode correctly
- [ ] Visual effects (flashes, vignette) display correctly
- [ ] Game is responsive on mobile viewport (360px-480px width)
- [ ] No memory leaks from continuous canvas rendering

## Git Workflow

- **Branch**: `claude/inspiring-planck-rgwoop` (development branch)
- **Commits**: Include descriptive messages explaining the "why" of changes
- **Attribution**: Include co-author and session information in commits (see example in git history)

## Notes for Claude Code Agents

- The entire game is contained in a single index.html file for easy deployment
- Canvas rendering runs in a continuous loop; be careful with state mutations
- All game state should be localized to avoid side effects
- Mobile viewport includes safe-area-inset considerations (notches, home bars)
- Test changes by opening index.html in a browser before considering work complete
