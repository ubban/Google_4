# Claude.md - Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 and Canvas. It runs directly in the browser with no build process required.

## Key Technical Details

- **Language**: JavaScript (vanilla, no frameworks)
- **Rendering**: HTML5 Canvas API
- **Architecture**: Single-page application (`index.html` contains all code)
- **Game Loop**: RequestAnimationFrame-based
- **Input**: Keyboard + mouse/touch controls

## Development Guidelines

### Code Style

- Use modern JavaScript (ES6+) where appropriate
- Keep the code within a single file for simplicity
- Use clear variable and function names that reflect game concepts
- Avoid external dependencies; vanilla JavaScript only

### Game Development Best Practices

- **Collision detection**: Use bounding box or distance-based algorithms
- **Rendering**: Clear canvas each frame and redraw all objects
- **Performance**: Maintain 60 FPS target; profile with browser DevTools
- **Mobile support**: Test on mobile devices; ensure touch controls work smoothly
- **Responsive design**: Game should adapt to different screen sizes and safe-areas (notches)

### Testing

- Open `index.html` directly in a browser (no server needed)
- Test on multiple devices: desktop, tablet, mobile
- Verify responsive behavior across screen sizes
- Check touch controls on mobile devices
- Use browser console for debugging (console.log, etc.)

### Git Conventions

- Keep commits focused and descriptive
- Include feature name and brief description in commit messages
- Avoid large refactors without clear justification

### Common Development Tasks

- **Adding new features**: Add game logic and rendering code to the main game loop
- **Adding enemies**: Define enemy patterns and update the spawn/update logic
- **UI updates**: Modify HUD or on-canvas text rendering
- **Performance optimization**: Profile with DevTools; look for rendering bottlenecks
- **Bug fixes**: Test thoroughly across browsers and devices before committing

## File Structure

```
/
├── index.html       (entire game - HTML, CSS, and JavaScript)
├── README.md
└── Claude.md        (this file)
```

## Deployment

The game is ready to deploy as-is:
1. No build process needed
2. No dependencies to install
3. Simply host `index.html` on any web server
4. Works on modern browsers that support HTML5 Canvas

## Notes for Claude Code

- When making changes, test the game in a browser to verify the visual/gameplay impact
- Canvas-based games require frame-by-frame testing; don't rely solely on code review
- Consider mobile device behavior when implementing new features
- Performance matters for smooth gameplay—avoid unnecessary redraws
