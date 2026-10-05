# Sky Fighter - Development Guide

Sky Fighter is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript.

## Project Overview

A browser-based bullet-hell shooter featuring:
- Player aircraft with weapon switching mechanics
- Smart bomb capability for clearing the screen
- Multiple enemy types and stage bosses
- Wave-based progression system
- Touch and keyboard input support
- Responsive design for desktop and mobile

## Key Files

- **index.html** - Single-file game implementation containing HTML structure, CSS styling, and all JavaScript game logic

## Development Setup

1. **Running the game locally:**
   - Open `index.html` directly in a web browser
   - No build process or dependencies required
   - Supports all modern browsers with HTML5 Canvas support

2. **Testing:**
   - Manual testing in browser
   - Test on multiple devices (desktop/mobile) due to responsive design
   - Verify touch controls and keyboard input

## Game Mechanics

- **Player Movement:** Arrow keys or mouse/touch
- **Shooting:** Continuous automatic fire
- **Weapon Switching:** Change active weapon
- **Smart Bombs:** Clear all enemies on screen (limited uses)
- **Score & Lives:** Track progress through stages

## Code Organization (within index.html)

The single-file implementation includes:
- **HTML Structure:** Game container and canvas element
- **CSS Styling:** Dark theme with gradient backgrounds, HUD styling
- **Game Engine:**
  - Player ship class and movement
  - Enemy spawning and AI
  - Projectile and collision detection
  - Score/waves/lives tracking
  - Event listeners for input

## Development Guidelines for Claude Code

### When Making Changes:

1. **Test in browser** - Always verify game functionality works as expected
2. **Preserve existing mechanics** - Don't break core gameplay without explicit request
3. **Mobile compatibility** - Keep touch controls functional; test responsive behavior
4. **Performance** - Avoid unoptimized loops; maintain 60fps gameplay
5. **Single-file structure** - Keep all code in index.html unless refactoring is explicitly requested

### Common Tasks:

- **Adding features:** Insert new code into appropriate section (HTML/CSS/JS game classes)
- **Bug fixes:** Locate issue in game logic, fix with minimal changes
- **Balance changes:** Adjust game parameters (difficulty, enemy stats, scoring)
- **Visual updates:** Modify CSS or Canvas drawing code

## Testing Checklist

- [ ] Game starts and loads without errors
- [ ] Player can move and shoot
- [ ] Enemies spawn and move correctly
- [ ] Collision detection works
- [ ] Weapon switching functions
- [ ] Smart bombs clear enemies
- [ ] Game over/win conditions trigger appropriately
- [ ] Works on mobile (touch controls)
- [ ] Works on desktop (keyboard controls)

## Notes

- Game state is client-side only (no backend)
- Randomization used for enemy patterns and spawning
- Canvas rendering is the primary visual output
- No external dependencies (vanilla JavaScript)
