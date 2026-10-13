# Sky Fighter - Developer Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application. The game features dynamic gameplay mechanics including weapon switching, smart bombs, stage bosses, and responsive controls for both desktop and mobile platforms.

- **Type**: Web-based game
- **Language**: JavaScript (ES2020+)
- **Framework**: Vanilla JavaScript with Canvas API
- **Platform**: Browser-based (responsive design for mobile/desktop)
- **Main File**: `index.html` (58KB single-file app)

## Architecture

### Game Engine Structure

The game uses a single HTML file (`index.html`) containing:
- **HTML Structure**: Canvas container and HUD elements
- **CSS Styling**: Responsive layout with dark theme, safe-area insets for mobile
- **JavaScript Logic**: Complete game engine embedded in `<script>` tags

### Key Game Systems

1. **Rendering System**
   - Canvas-based rendering (2D context)
   - Layered drawing: background → enemies → player → projectiles → UI
   - Adaptive sizing with `max-width` and `max-height` constraints

2. **Input Handling**
   - Keyboard controls (arrow keys, space bar)
   - Touch controls for mobile (pointer events)
   - Custom `touch-action: none` to prevent browser gestures

3. **Game Loop**
   - RequestAnimationFrame (RAF) for smooth 60fps gameplay
   - Update phase: entity movement, collision detection
   - Render phase: draw all visible elements

4. **Entity System**
   - Player ship with health and state management
   - Enemy ships with AI behavior and weapon systems
   - Projectiles (bullets, bombs) with velocity and lifetime
   - Boss entities with multi-phase patterns

5. **Weapon System**
   - Multiple weapon types with different firing patterns
   - Player can switch weapons during gameplay
   - Different cooldown and damage characteristics per weapon

6. **Collision Detection**
   - Rectangle-based collision for efficiency
   - Separate collision layers: player-bullet, enemy-bullet, entity-entity
   - Hit feedback with damage application and visual effects

7. **UI/HUD System**
   - Score, health, lives, wave indicators
   - Weapon display with ammo/cooldown
   - Game over and wave completion screens
   - Safe-area aware positioning for notched devices

## Development Guidelines

### Code Organization
- All code is in a single file for simplicity and deployment
- JavaScript is organized into logical sections with clear boundaries
- No build process required - file can be opened directly in browser

### Responsive Design
- Uses CSS `clamp()` for fluid typography and spacing
- Implements `env(safe-area-inset-*)` for mobile notch safety
- Canvas scales intelligently with `max-width` and `max-height`
- Touch targets are appropriately sized for mobile interaction

### Mobile Considerations
- `viewport-fit=cover` for full screen on notched devices
- `user-scalable=no` to prevent zoom interfering with gameplay
- `touch-action: none` prevents default browser touch behaviors
- Pointer events for both mouse and touch input

### Performance
- Use requestAnimationFrame for smooth animation
- Minimize DOM reflows - canvas-based rendering only
- Reuse objects where possible to reduce GC pressure
- Consider reducing particle effects on low-end devices

### Styling Best Practices
- Use CSS custom properties (`:root`) for theme colors
- Maintain dark theme for reduced eye strain during gameplay
- Use `box-shadow` for depth without actual 3D transforms
- Apply `text-shadow` for readable text on complex backgrounds

## File Structure

```
Google_4/
├── README.md           # Basic project info with timestamp
├── index.html          # Complete game implementation
└── CLAUDE.md          # This file - developer documentation
```

## Common Development Tasks

### Adding a New Enemy Type
1. Define enemy properties (size, speed, health, weapon)
2. Create AI behavior function for movement pattern
3. Add collision and damage handling
4. Test in appropriate game level/wave

### Modifying Weapon Behavior
1. Locate weapon definition in the weapons system
2. Adjust damage, cooldown, projectile speed, or spread pattern
3. Test weapon switching and firing rate
4. Verify balance against enemy difficulty

### Adjusting Game Difficulty
1. Modify enemy spawn rate in wave definition
2. Adjust enemy health or damage output
3. Tweak boss patterns and attack frequency
4. Balance with player weapon power and health

### Adding Visual Effects
1. Use canvas `fillRect`, `arc`, or image drawing
2. Consider performance impact on mobile devices
3. Test on various canvas sizes
4. Ensure effects don't obscure gameplay

## Testing

### Manual Testing
- Test on multiple screen sizes (mobile, tablet, desktop)
- Verify touch controls work on actual devices
- Check safe-area handling on notched phones
- Test all weapon types and switching
- Verify collision detection works correctly
- Test game over and level completion states

### Browser Compatibility
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)
- Canvas 2D API widely supported
- RequestAnimationFrame available in all modern browsers
- ES2020+ features should work in contemporary browsers

## Deployment

### Building
- No build process needed
- Single `index.html` file can be deployed directly
- Static hosting suitable (GitHub Pages, Netlify, etc.)

### Performance Optimization
- Consider minifying HTML/CSS/JavaScript if file size becomes critical
- Optimize any embedded images/assets
- Use gzip compression for serving

### Live Server
```bash
# Python 3
python -m http.server 8000

# Node.js
npx http-server

# Then open: http://localhost:8000
```

## Useful Git Commands

```bash
# Check current branch
git branch

# View recent changes
git log --oneline -n 10

# See file changes
git diff index.html

# Stage and commit changes
git add CLAUDE.md index.html
git commit -m "Update game features and documentation"

# Push to remote
git push -u origin claude/inspiring-planck-lqsqyz
```

## References

### Canvas API
- [MDN Canvas Tutorial](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Canvas 2D Context](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)

### Web Game Development
- [RequestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- [Pointer Events](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events)
- [Safe Area Insets](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafeAreaLayoutGuide/)

## Notes for Claude Code Agents

- This is a single-file game implementation - all changes go to `index.html`
- Test changes in a browser before pushing
- Keep the CSS responsive with `clamp()` and safe-area support
- Maintain performance - avoid expensive operations in the game loop
- Preserve the dark theme color scheme (#05122b, #0d2f5e, etc.)
- Mobile compatibility is important - test touch input thoroughly
- Update this CLAUDE.md when adding new major systems

---

Last updated: 2026-10-13
