# Claude.md — Google_4 / Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application. The game features:
- Player-controlled aircraft with weapon switching and smart bombs
- Progressive wave-based enemy spawning with stage bosses
- Score tracking and game state management
- Touch and keyboard input support for responsive gameplay

The entire game is contained in `index.html` as an embedded HTML/CSS/JavaScript application.

## Project Structure

```
Google_4/
├── index.html          # Complete game (HTML + CSS + JavaScript)
├── README.md           # Basic project metadata
└── Claude.md           # This file — development guidelines
```

## Key Technical Details

### Game Architecture
- **Canvas-based rendering**: Uses HTML5 Canvas for all graphics
- **Game loop**: Frame-based animation (requestAnimationFrame)
- **State management**: Game state tracks player, enemies, projectiles, and waves
- **Input handling**: Both touch (mobile) and keyboard (desktop)
- **Scoring system**: Points awarded for defeating enemies and completing waves

### Core Game Systems
1. **Player**: Single aircraft with health, position, weapon selection, and bomb count
2. **Weapons**: Multiple weapon types with different firing patterns
3. **Enemies**: Various enemy types with different behaviors and patterns
4. **Projectiles**: Bullets and bombs with collision detection
5. **Waves**: Progressive difficulty with stage bosses
6. **UI**: HUD for score, lives, weapon, health, and bomb count

### Styling & Responsive Design
- Dark blue gradient background (space theme)
- Responsive canvas sizing (max-width/height with 100% coverage)
- Safe area insets for notched devices
- Touch-action disabled to prevent browser gestures
- Crosshair cursor for gameplay clarity

## Development Guidelines

### Code Style
- Keep embedded HTML/CSS/JavaScript organized with clear section comments
- Use descriptive variable names (player, enemies, projectiles, etc.)
- Maintain single-file structure—avoid external dependencies
- Use vanilla JavaScript (no frameworks)

### Game Mechanics
- Player starts with full health at screen center
- Weapon switching via number keys (1-9)
- Smart bombs drop via spacebar with limited count
- Enemy waves increase in difficulty progressively
- Stage bosses have unique behaviors and higher health
- Game ends when player health reaches zero

### Canvas Drawing & Rendering
- All graphics rendered via canvas 2D context
- Maintain aspect ratio with window resize handling
- Use efficient drawing (batch operations where possible)
- Clear canvas each frame before redrawing

### Input Handling
- Keyboard: Arrow keys for movement, number keys for weapons, spacebar for bombs
- Touch: Single-touch targeting with player following center screen
- Both systems should remain responsive and lag-free

### Performance Considerations
- Minimize redraws; clear and redraw entire scene each frame
- Limit particle effects and visual clutter
- Monitor requestAnimationFrame performance
- Test on both desktop and mobile devices

## Testing

### Manual Testing Checklist
- **Game start**: Player appears at center, HUD displays correctly
- **Movement**: Smooth player movement in all directions
- **Weapons**: Switch weapons (1-9) and verify firing patterns
- **Collisions**: Bullets hit enemies, enemies collide with player
- **Scoring**: Score updates correctly when enemies are defeated
- **Bombs**: Smart bombs spawn and clear nearby enemies
- **Waves**: Difficulty increases, bosses appear as expected
- **Game over**: Game ends when player health reaches zero
- **Responsive**: Game scales properly on mobile and desktop
- **Input**: Both keyboard and touch input work seamlessly

### Browser Compatibility
- Test on modern browsers (Chrome, Firefox, Safari, Edge)
- Verify mobile responsiveness (iOS Safari, Chrome Mobile)
- Confirm touch event handling on devices with notches

## Important Conventions

### Commits
- Make clear, descriptive commit messages explaining changes
- Reference specific systems modified (e.g., "Fix enemy wave spawning logic")
- Include context in the message body for significant changes

### Branch Strategy
- Develop on `claude/inspiring-planck-4bntot`
- Push completed work with clear commit history
- Keep commits logically grouped and reviewable

### File Integrity
- Do not split the game into multiple files
- Keep `index.html` as the single source of truth
- Maintain the embedded structure

## When Modifying the Game

1. **Understand the current game loop**: How the canvas renders, how input is handled, how collisions work
2. **Test changes locally**: Open `index.html` in a browser and verify gameplay
3. **Commit incrementally**: Each logical change should be a separate commit
4. **Maintain balance**: Don't make changes that significantly alter intended difficulty
5. **Update this file**: If adding new systems or major changes, document them here

## Quick Reference

### How to Run
- Open `index.html` directly in a web browser
- No build process or server required
- Game is fully playable immediately

### Key Game States
- **PLAYING**: Active gameplay, handle input, update entities
- **WAVE_COMPLETE**: Wait before advancing to next wave
- **GAME_OVER**: Display final score, wait for restart

### Important Game Constants
- Look in the code for: player health, enemy spawn rates, wave progression, damage values
- Adjust these values to balance difficulty and gameplay feel
