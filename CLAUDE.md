# Claude.md — Development Guide for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The game features weapon switching, smart bombs, and stage bosses.

## Architecture

### Single-File Game Engine
The entire game is contained in `index.html` with inline JavaScript. This is intentional for simplicity and minimal deployment overhead.

### Key Components
- **Canvas Rendering**: All graphics are drawn to the game canvas using 2D context
- **Game Loop**: Runs at 60 FPS using `requestAnimationFrame`
- **Entity System**: Player, enemies, projectiles, and effects are managed in arrays
- **Input Handling**: Keyboard and touch/mouse controls supported
- **Audio**: Sound effects via Web Audio API (where available)

## Development Workflow

### Branch Strategy
All development occurs on feature branches prefixed with `claude/`. The main branch is reserved for stable releases.

- Feature branch: `claude/inspiring-planck-4alx4s`
- Always create new commits for changes
- Push with: `git push -u origin <branch-name>`

### Code Style Guidelines

#### HTML/CSS
- Use semantic HTML structure
- Keep CSS within `<style>` tags for single-file simplicity
- Use CSS custom properties (`:root`) for theme colors
- Ensure mobile responsiveness with `clamp()` and `max()`

#### JavaScript
- Use vanilla JS (no frameworks)
- Prefer `const` over `let` for immutable references
- Use descriptive variable names
- Keep functions focused and under 50 lines when possible
- Add comments only when **why** is non-obvious

#### Game Logic
- Separate concerns: rendering, state, input handling, collisions
- Use object literals for entity data
- Keep the game loop tight and performant
- Test gameplay mechanics manually before committing

## Testing

### Manual Testing Checklist
- [ ] Game starts without errors
- [ ] Player can move (keyboard and touch)
- [ ] Weapons cycle and fire correctly
- [ ] Enemies spawn and move
- [ ] Collisions work (projectiles, damage)
- [ ] Smart bombs activate and clear enemies
- [ ] Boss fights trigger at stage milestones
- [ ] Score and UI update correctly
- [ ] Game responds to window resize (responsive)
- [ ] Audio plays (if implemented)

### Browser Compatibility
- Chrome/Edge (latest)
- Firefox (latest)
- Safari (latest, including iOS)
- Mobile browsers (iOS Safari, Chrome Mobile)

## Common Tasks

### Adding a New Feature
1. Work on the feature branch
2. Make atomic commits with clear messages
3. Test manually before pushing
4. Push to the feature branch

### Debugging
- Use browser DevTools console for `console.log()` statements
- Canvas rendering issues: check 2D context calls
- Performance: use Performance tab in DevTools
- Collision issues: visualize hitboxes temporarily with `strokeRect()`

### Performance Optimization
- Limit entity count (enemies, projectiles)
- Batch canvas operations where possible
- Avoid unnecessary DOM manipulation
- Use `requestAnimationFrame` exclusively for animation timing

## Deployment

The game is a static HTML file and can be deployed directly:
- Copy `index.html` to any web server
- No build step or dependencies required
- Set proper MIME types for `.html`

## Git Commit Message Format

Use clear, descriptive commit messages:
```
<type>: <description>

<optional body explaining why>
```

Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `style`

Example:
```
feat: Add smart bomb weapon with screen-clearing effect

Adds a new weapon type that clears all on-screen enemies
and projectiles when activated. Limited uses per stage.
```

## Known Limitations & Constraints

- Single file for simplicity (no build tooling)
- Canvas rendering may have performance limits on very old devices
- Audio implemented via Web Audio API (some browsers may block autoplay)
- No save/load system (session only)

## Resources & References

- MDN Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game loop pattern: https://developer.mozilla.org/en-US/docs/Games/Anatomy
- Raiden gameplay: Classic vertical shmup mechanics

## Claude Code Agent Rules

1. **Never** push to main without explicit user approval
2. **Always** test manually before committing
3. **Keep changes focused**: one feature or fix per commit
4. **Preserve single-file structure** unless major refactoring is requested
5. **Document why**, not what—let code be self-explanatory
6. **Responsive design**: test on multiple viewport sizes
7. **No breaking changes** without discussion with the user
8. **Commit attribution**: Include standard attribution footer in commit messages

---

Last updated: 2026-09-30
