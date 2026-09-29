# Claude.md — Sky Fighter Development Guidelines

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5/Canvas and JavaScript. The game features:
- Switch weapons system
- Smart bomb mechanics
- Stage bosses
- Progressive difficulty

## Project Structure
- `index.html` - Single-file game implementation with HTML, CSS, and JavaScript inline
- `README.md` - Project metadata
- No external dependencies (vanilla JavaScript)

## Development Guidelines

### Code Style
- Use vanilla JavaScript (no frameworks or build tools)
- Keep all code inline in `index.html` for simplicity
- Use semantic HTML5 structure
- CSS should support dark mode and responsive design
- Maintain crosshair cursor for desktop gameplay

### Game Mechanics to Preserve
- Canvas-based rendering for performance
- Touch and mouse input support
- Safe area insets for notch/cutout devices
- Dynamic difficulty scaling
- Boss encounter mechanics
- Weapon switching system
- Smart bomb functionality

### Testing Requirements
- Manual testing in desktop browser (Chrome/Firefox/Safari)
- Mobile testing on iOS Safari and Android Chrome
- Verify responsive scaling on various screen sizes
- Test touch controls and mouse controls independently
- Validate performance on lower-end devices

### Browser Compatibility
- Modern browsers supporting:
  - Canvas 2D Context
  - CSS Grid/Flexbox
  - ES6 JavaScript (arrow functions, const/let, template literals)
- Touch Action CSS for input handling
- Viewport meta tags for mobile optimization

### Commits
- Use clear, descriptive commit messages
- Focus commits on specific features or fixes
- Include co-author attribution when working with other agents

### Pull Requests
- Create descriptive PR titles (e.g., "Add boss encounter mechanics" vs. "Update game")
- Include testing notes in PR body
- Link related issues if applicable
- Verify visual/gameplay changes in browser before pushing

## Common Tasks

### Adding Game Features
1. Identify the appropriate location in the JavaScript code
2. Implement the feature with clear function naming
3. Test thoroughly across platforms
4. Document any new input bindings or mechanics

### Performance Optimization
- Profile with DevTools Performance tab
- Optimize Canvas rendering operations
- Minimize DOM reflows
- Use requestAnimationFrame for smooth animation

### Mobile Optimization
- Test with DevTools device emulation
- Verify safe area handling
- Ensure touch targets are adequate size
- Check battery/performance impact

## Development Branch
Work on feature branches and create PRs for review before merging to main.

## Notes for Claude Code
- This is a self-contained HTML game file
- Changes are immediately visible without build steps
- Browser DevTools are essential for debugging
- Performance matters for gameplay experience
