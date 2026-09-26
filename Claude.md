# Claude.md - Sky Fighter Project Guidelines

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page HTML5 application. The game features:
- A player-controlled aircraft that switches weapons and drops smart bombs
- Stage bosses with distinct attack patterns
- Mobile-friendly responsive design with touch support
- Dark theme with dynamic gradient backgrounds

The codebase is entirely self-contained in `index.html` - a monolithic single-file game with embedded CSS, JavaScript, and game assets.

## Codebase Structure

- **index.html**: The only source file, containing:
  - HTML structure with game canvas and HUD elements
  - Embedded CSS for styling and layout (responsive, touch-optimized)
  - Embedded JavaScript for all game logic, physics, and AI

## Development Guidelines

### Code Organization
- Keep all game code within the single `index.html` file to maintain simplicity
- Organize logic into distinct sections: canvas setup, game state, entity systems, input handling, rendering, and update loops
- Use semantic HTML and CSS Grid/Flexbox for layout

### Game Development Principles
- Maintain 60 FPS render loop using `requestAnimationFrame`
- Use canvas 2D context for all rendering
- Implement collision detection efficiently (AABB for most entities, pixel-perfect where needed)
- Audio/sound effects should be non-blocking and optional (graceful fallback if not available)

### Responsive Design
- Game must scale to fit mobile and desktop screens
- Use CSS viewport-fit for notch-aware mobile browsers
- Touch controls should coexist with keyboard controls
- Maintain aspect ratio with safe-area consideration

### Performance Considerations
- Minimize DOM manipulation - most rendering is canvas-based
- Pre-calculate sprite positions and animations where possible
- Batch canvas operations to reduce draw calls
- Use requestAnimationFrame, not setTimeout, for game loop

### Browser Compatibility
- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Graceful degradation for missing features (localStorage, touch events)
- Test on mobile devices with various screen sizes and safe areas

## Testing and Validation

### Manual Testing Checklist
- [ ] Game loads without console errors
- [ ] Game canvas renders correctly at various viewport sizes
- [ ] Player can control aircraft with keyboard (arrow keys, space, etc.)
- [ ] Enemies spawn and move according to patterns
- [ ] Collision detection works (bullets hit enemies, enemy bullets hit player)
- [ ] HUD displays correctly (health, score, wave, etc.)
- [ ] Boss stages appear after regular enemy waves
- [ ] Weapon switching mechanic works smoothly
- [ ] Smart bomb mechanic works and clears screen appropriately
- [ ] Game over and victory conditions display correctly
- [ ] Touch controls work on mobile devices
- [ ] Frame rate remains stable (60 FPS target)
- [ ] No memory leaks during extended play sessions

### Browser DevTools
- Check Console for errors and warnings
- Monitor Performance tab for frame rate stability
- Verify Network tab shows only the single HTML file loaded

## Deployment / Publication

### Release Process
1. Validate all manual test checklist items pass
2. Run performance profiling to ensure 60 FPS stability
3. Commit changes with descriptive message
4. Push to the designated branch
5. Create pull request for code review if required
6. Upon approval, merge to main branch

### Hosting
- The game is a static HTML file - can be served by any web server
- GitHub Pages can serve directly from the repository
- No build step required
- Ensure `Content-Type: text/html; charset=utf-8` is set by server

## Git Workflow

### Branch Convention
- Feature/fix branches: `feature/description` or `fix/description`
- Main development branch: Designated feature branch (e.g., `claude/inspiring-planck-h1i14q`)
- Default branch: `main`

### Commit Messages
- Use clear, descriptive messages
- Start with verb: "Add", "Fix", "Improve", "Refactor"
- Keep first line under 72 characters
- Include Co-Authored-By footer for Claude-generated commits:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: [session-url]
  ```

### Pull Requests
- Describe what changed and why
- Link to any related issues
- Include testing evidence (screenshots, performance metrics)
- End with PR footer:
  ```
  🤖 Generated with [Claude Code](https://claude.com/claude-code)
  https://claude.ai/code/session_[session-id]
  ```

## Code Style Guidelines

### HTML/CSS
- Use semantic HTML5 elements where appropriate
- Use CSS custom properties (variables) for colors, dimensions
- Responsive design with `clamp()` for fluid typography
- Mobile-first approach with progressive enhancement

### JavaScript
- Use consistent indentation (2 spaces)
- Descriptive variable and function names (camelCase for variables/functions, CONSTANT_CASE for constants)
- Keep functions focused and single-purpose
- Use const/let appropriately (avoid var)
- Comment complex algorithms and non-obvious logic only

### Canvas Drawing
- Clear canvas at start of each frame
- Organize drawing calls logically (background, entities, UI)
- Use canvas state save/restore for transformations
- Optimize draw order to minimize overdraw

## Special Considerations

### Mobile-First Design
- Game must work on portrait and landscape orientations
- Touch events should feel responsive (minimal input lag)
- Safe area (notches, rounded corners) must be respected
- Zoom must be disabled for better gameplay feel

### Performance on Low-End Devices
- Minimize particle effects or provide quality settings
- Optimize asset reuse (sprite sheets, cached calculations)
- Profile on older devices if performance is critical

### Accessibility
- Ensure color contrast meets WCAG AA standards
- Provide high-contrast mode if possible
- Consider alternatives to sound-only feedback
- Keyboard controls should work fully

### Known Limitations
- Single HTML file keeps complexity contained but limits organization
- No build process means no minification/bundling
- All assets must be inline or embedded
- Consider file size impact of additions

## Useful Commands

```bash
# View current branch
git branch

# View recent commits
git log --oneline -20

# Check what files would be committed
git status

# View changes to be committed
git diff --cached

# Push to the feature branch
git push -u origin <branch-name>

# Fetch latest from remote
git fetch origin

# Create and switch to a new branch
git checkout -b <branch-name>
```

## Review Checklist for Claude Agents

Before committing changes:
1. ✓ Code is syntactically valid (no console errors)
2. ✓ Changes maintain single-file structure
3. ✓ Performance impact assessed (still 60 FPS)
4. ✓ Mobile responsiveness maintained
5. ✓ Manual testing completed per checklist above
6. ✓ Commit message is clear and follows convention
7. ✓ No unintended whitespace or formatting changes
8. ✓ All related issues/PRs referenced appropriately

## References

- [MDN Canvas API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)
- Repository: https://github.com/ubban/Google_4
