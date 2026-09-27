# Claude.md - Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features a player ship that can switch weapons, drop smart bombs, and battle stage bosses.

- **Repository**: ubban/google_4
- **Main File**: `index.html` (~59KB with embedded CSS and JavaScript)
- **Type**: Browser-based Game (Canvas/WebGL)
- **Platform**: All modern browsers

## Project Structure

```
/Google_4/
├── index.html          # Single-file game application (HTML + CSS + JavaScript)
├── README.md           # Project description
└── .git/               # Git repository
```

## Development Guidelines

### Code Style
- **Language**: Vanilla JavaScript (no frameworks)
- **Styling**: CSS embedded in HTML <style> tag
- **Game Loop**: Canvas-based rendering with requestAnimationFrame
- **Assets**: Inline SVG and canvas drawing (no external image dependencies)

### Game Architecture
The game is organized as a monolithic HTML file with distinct sections:
1. **HTML Structure**: Canvas element and HUD container
2. **CSS Styling**: Game canvas styling, HUD layout, responsive design
3. **JavaScript Logic**: Game engine, entity management, collision detection, UI

### Key Game Components
- **Player Ship**: Controllable entity with weapons and movement
- **Weapons System**: Multiple weapon types that can be switched
- **Enemies**: Various enemy patterns and behaviors
- **Bosses**: Stage-based boss encounters
- **Bombs**: Smart bombs for clearing the screen
- **Collision Detection**: Bullet-to-enemy, enemy-to-player interactions
- **HUD**: Score, lives, weapon info, game state indicators

### Touch & Input Handling
- **Mouse/Touch Support**: Game responds to both mouse movement and touch input
- **Platform Detection**: Handles mobile (portrait mode) and desktop (16:9 aspect)
- **Safe Area Insets**: Respects notches and safe areas on mobile devices

## Testing & Validation

### Manual Testing Checklist
When making game changes, verify:
1. **Canvas Rendering**: Game displays correctly without visual glitches
2. **Input Handling**: Mouse/touch controls work smoothly
3. **Game Physics**: Bullets, enemies, and collision detection function properly
4. **Weapon Switching**: All weapons fire and switch correctly
5. **Bomb Mechanics**: Smart bombs clear enemies and work as intended
6. **Boss Encounters**: Boss patterns and behavior are correct
7. **Mobile Responsiveness**: Game adapts to different screen sizes
8. **Performance**: Game runs smoothly (no lag, dropped frames)
9. **Audio** (if implemented): Sound effects trigger at correct times

### Opening the Game
The game is a standalone HTML file—open `index.html` directly in a browser:
```bash
# Linux/Mac: Open in default browser
open index.html
# or
firefox index.html

# Windows: Double-click index.html or
start index.html
```

## Performance Considerations

- **Single File**: No HTTP requests for assets; everything is embedded
- **Canvas Rendering**: Uses efficient 2D canvas API for rendering
- **Frame Rate**: Targets 60 FPS using requestAnimationFrame
- **Memory**: Game entities are pooled/recycled when possible to reduce garbage collection

## Common Tasks

### Adding a New Feature
1. Identify if it's game logic (JavaScript) or visual (CSS)
2. Add the feature in the appropriate `<script>` or `<style>` section
3. Test the feature manually by opening the game in a browser
4. Verify no regressions in existing gameplay

### Fixing a Bug
1. Reproduce the bug by opening the game and testing the scenario
2. Locate the relevant code section in the JavaScript logic
3. Apply the fix and re-test the specific scenario
4. Verify no side effects on other game systems

### Performance Optimization
- Profile using browser DevTools (Chrome: F12 → Performance)
- Look for inefficient entity updates or rendering calls
- Use object pooling to reduce garbage collection pressure
- Minimize DOM manipulation (game mainly uses Canvas)

## Browser Compatibility

- **Modern Browsers**: Chrome, Firefox, Safari, Edge (all current versions)
- **Mobile Browsers**: iOS Safari, Chrome Mobile, Firefox Mobile
- **Fallback**: The game does not degrade gracefully for older browsers; it requires:
  - HTML5 Canvas
  - CSS Flexible Box Layout
  - ES6 JavaScript features

## Dependencies

**Zero Dependencies**: This project uses no external libraries or frameworks. Everything is vanilla JavaScript.

## Git Workflow

- **Branch**: Develop on `claude/inspiring-planck-c7vlcm`
- **Commits**: Make clear, descriptive commits for each logical change
- **Push**: Push changes to the designated branch with `git push -u origin claude/inspiring-planck-c7vlcm`

## Notes for Claude Code

- The HTML file contains all game logic—changes to the game require editing this single file
- Always validate changes by opening `index.html` in a browser; unit tests are not available for game logic
- When modifying game mechanics, ensure the physics and collision detection remain consistent
- The game uses CSS variables (`:root`) for theming; updating `color-scheme` or colors requires testing on both light and dark backgrounds
- Mobile testing is recommended, especially for touch input validation
