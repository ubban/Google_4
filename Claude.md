# Claude.md — Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla JavaScript and HTML5 Canvas. Players control a spaceship, switch weapons, drop smart bombs, and defeat stage bosses.

- **Tech Stack**: HTML5, CSS3, JavaScript (ES6+)
- **Entry Point**: `index.html` (single-page game)
- **Repository**: `ubban/google_4`
- **Development Branch**: `claude/inspiring-planck-51ph7b`

## Project Structure

```
/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project metadata
└── .git/               # Git repository
```

The entire game is contained in a single `index.html` file. The structure within the file is:

1. **HTML**: Game container with canvas element and HUD
2. **CSS**: Styling for the canvas, HUD stats, animations (keyframes for effects)
3. **JavaScript**: Game logic, entities, collision detection, etc.

## Key Game Systems

### Player/Ship
- Health system (5 hearts by default)
- Weapon switching (multiple weapon types with different fire patterns)
- Smart bomb mechanic (limited charges)
- Position locked to horizontal center, moves vertically

### Enemies
- Multiple enemy types with different behaviors
- Boss encounters at stage end
- Projectile attacks and collision detection

### Gameplay Mechanics
- Stage progression system
- Score and multiplier system
- Visual feedback (flashes, vignette effects for damage)
- Sound/visual effects for impacts, bombs, etc.

### UI/HUD
- Health hearts display
- Bomb counter
- Current weapon display
- Score and multiplier display
- Real-time HUD updates during gameplay

## Development Guidelines

### Code Style
- Use ES6+ features (const/let, arrow functions, template literals)
- Keep related code organized in logical sections
- Use meaningful variable and function names
- Add comments only for non-obvious logic

### Working with the Canvas
- Canvas dimensions adapt to viewport (max-width/max-height)
- Coordinate system: (0,0) at top-left, X increases rightward, Y increases downward
- All drawing happens in a single render loop
- Update game state first, then render

### Entity Management
- Store entities (player, enemies, projectiles) in arrays or Sets
- Implement collision detection for projectiles, player damage, boss health
- Clean up inactive entities (off-screen, destroyed)

### Adding Features
When adding new features:
1. Define the feature in a clear comment block
2. Update the relevant entity arrays/state
3. Add collision/update logic in the main loop
4. Update HUD if needed
5. Test across different screen sizes (mobile/desktop)

### Mobile Considerations
- Game uses `touch-action: none` and supports touch input
- HUD uses `clamp()` and viewport units for responsive sizing
- Cursor is `crosshair` for aiming feedback
- Safe area insets respected for notched devices (`env(safe-area-inset-top)`)

## Git Workflow

### Branch
All work happens on: **`claude/inspiring-planck-51ph7b`**

### Commits
- Write clear, descriptive commit messages
- Reference what changed (e.g., "Add new enemy type: laser drone")
- Include Co-Author attribution in commit messages

### Push
Use: `git push -u origin claude/inspiring-planck-51ph7b`

### Pull Requests
- Create PRs when work is ready for review
- Use the PR template if one exists in `.github/pull_request_template.md`
- Include a summary of changes and test results
- Link to any related issues

## Running and Testing

### Local Development
1. Open `index.html` in a modern web browser
2. Game should load immediately with the canvas visible
3. Use keyboard/mouse to play and test features

### Testing Checklist
- [ ] Game loads without console errors
- [ ] Canvas renders correctly (check inspector for size)
- [ ] Keyboard/mouse input works
- [ ] Touch input works on mobile
- [ ] HUD displays correctly and updates in real-time
- [ ] Player can move and shoot
- [ ] Weapons switch properly
- [ ] Bombs function and display correctly
- [ ] Enemies spawn and behave as expected
- [ ] Collision detection works
- [ ] Game responds to win/lose conditions
- [ ] Visual effects (flashes, vignette) play correctly

### Debugging Tips
- Use browser DevTools (F12) to inspect the canvas and check for errors
- Add `console.log()` statements to trace game state
- Use the Performance tab to monitor frame rate
- Check the Network tab if assets aren't loading

## Important Conventions

### Canvas Rendering
- Clear canvas at start of each frame: `ctx.clearRect(...)`
- Draw background first, then entities, then effects
- Use `requestAnimationFrame()` for smooth 60 FPS rendering

### Game State
- Keep all game state in easily accessible variables
- Update state before rendering each frame
- Avoid tightly coupling rendering and state updates

### Mobile & Responsive
- Test changes on both desktop and mobile (use browser dev tools)
- Canvas should be centered and scale appropriately
- HUD should remain readable at all viewport sizes

### Performance
- Reuse canvas context rather than creating new ones
- Avoid DOM manipulation during game loop (update once per frame or less)
- Use `const` and `let` appropriately to help with memory management
- Profile if adding resource-heavy features

## File Format Notes

The `index.html` is a self-contained file with:
- **Line 1**: Standard HTML5 doctype
- **Lines 2-8**: Metadata and viewport configuration
- **Lines 9+**: Single `<style>` block with all CSS
- **Body**: Game container with canvas and HUD
- **Script**: All JavaScript in a single `<script>` tag at the end

When editing, be careful to maintain the structure and validity of the HTML/CSS/JS.

## Resources

- [MDN Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [MDN Collision Detection](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) (if adding sound)

## Tips for Claude Code Agents

1. **Read the full file first**: Since everything is in one file, start by reading `index.html` completely to understand the full picture.

2. **Test in browser immediately**: Always test changes in a browser after editing. Use DevTools to check for errors.

3. **Keep changes focused**: Make one logical change per commit. If adding a weapon, don't refactor the entire collision system in the same commit.

4. **Preserve structure**: The single-file format means everything is interdependent. Make sure your changes don't break the CSS or JavaScript scoping.

5. **Comment complex sections**: If adding complex game logic (new enemy AI, new weapon type, boss pattern), leave brief comments explaining the intent.

6. **Check cross-browser**: Test in Chrome, Firefox, and Safari if possible. Canvas behavior is generally consistent, but mobile browsers have quirks.

---

Last Updated: 2026-09-28
