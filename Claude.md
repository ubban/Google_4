# Claude Code Guidelines — Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/Canvas/JavaScript application. The game features:
- Player-controlled ship with weapon switching
- Enemy spawning and patterns
- Boss battles with multiple stages
- Smart bombs, scoring system, and health management
- Mobile and desktop support with touch controls

**Repository**: https://github.com/ubban/Google_4  
**Language**: HTML5, CSS, JavaScript (vanilla, no dependencies)  
**Entry point**: `index.html`

## Architecture

The entire game is implemented in a single `index.html` file (~15KB) with:
- **HTML**: Semantic structure and game canvas
- **CSS**: Styling, animations, responsive layout with viewport handling
- **JavaScript**: Game engine, entity management, input handling, rendering loop

### Key Sections (in order of appearance in HTML)

1. **HEAD**: Meta tags, viewport setup, styling
2. **BODY**: Game container and canvas element
3. **HTML Elements**: HUD stats, flash/vignette effects
4. **CSS Styles**: Game UI, animations, responsive rules
5. **JavaScript**: Engine and game logic

## Development Guidelines

### Code Style

- **No external dependencies**: Keep the project simple; use only vanilla HTML/CSS/JavaScript
- **Single file structure**: All changes go in `index.html` unless refactoring is specifically requested
- **Comments**: Add only when logic is non-obvious (e.g., complex math, game mechanics quirks)
- **Naming**: Use clear, descriptive variable/function names; prefer `playerShip` over `p`

### Making Changes

1. **Locate the section**: Use Ctrl+F to find the relevant section (e.g., search "class Player" or "function update")
2. **Test locally**: Open `index.html` in a browser and verify changes work before committing
3. **Mobile first**: Always test touch controls and responsive layout (viewport: 320px–2560px)
4. **Performance**: Keep frame rate stable (60fps target); avoid expensive DOM operations in the game loop

### Common Tasks

#### Adding a New Enemy Type
1. Find the enemy spawning logic (search: "function spawnEnemy")
2. Define the new enemy class with health, damage, sprite
3. Add to the `enemyTypes` or spawning conditions
4. Test with multiple instances on screen

#### Modifying Game Balance
- Player health: Search "const PLAYER_HEALTH"
- Weapon damage: Search "damage:" in weapon definitions
- Enemy stats: Search "health:" in enemy class definitions
- Boss health: Search "boss.health" or "BOSS_HEALTH"

#### Fixing or Adding Game Features
- Input handling: Search "addEventListener" (keyboard, mouse, touch)
- Rendering: Search "ctx.draw" or "canvas context"
- Physics/collision: Search "distance" or "collision"
- UI updates: Search "#hud" or "getElementById"

## Workflow & Branching

### Branch Strategy

- **Main branch**: `main` — stable, production-ready
- **Development branch**: `claude/inspiring-planck-0ztqp4` — active development
  - This is the designated branch for all Claude Code work
  - Push commits and PRs to this branch unless otherwise instructed

### Committing Changes

Use clear, concise commit messages that describe the change:

```
Add boss health bars to HUD
Improve enemy bullet pattern variety
Fix mobile touch target accuracy
Balance weapon switching speed
```

### Testing Before Commit

1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Test the golden path: start game → survive stage → reach boss → defeat boss
3. Test edge cases: 
   - Rapid weapon switching
   - Spawning many enemies at once
   - Mobile viewport (test at 375px width)
4. Check console for errors (F12 Developer Tools)

## Running & Testing

### Local Development

```bash
# Open in browser (macOS)
open index.html

# Or drag/drop index.html into any browser window
# Game runs immediately, no server needed
```

### Mobile Testing

- Use device browser or Chrome DevTools device emulation (F12 → Toggle device toolbar)
- Test touch controls: tap to move, drag to aim (if implemented)
- Verify canvas scales properly at different aspect ratios

### Debugging

- Open Browser DevTools (F12)
- Check Console tab for JS errors
- Use `console.log()` for debugging values in the game loop
- Inspect canvas rendering with `console.log(ctx)` if needed

## File Structure

```
Google_4/
├── .git/                 # Git metadata
├── .gitignore           # Git ignore rules (if exists)
├── Claude.md            # This file
├── README.md            # Project description
└── index.html           # The entire game (all code)
```

## Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Game runs slowly | Check browser hardware acceleration; profile JS with DevTools |
| Touch controls unresponsive | Verify touch event listeners; check `touch-action: none` in CSS |
| Canvas doesn't fill screen | Check viewport meta tag and CSS `#gameCanvas` sizing rules |
| Boss not spawning | Search "BOSS_HEALTH" and check spawn conditions |
| Weapon not switching | Check weapon index logic and key bindings |

## Deployment

The game is a static HTML file — deploy by:

1. Push to `claude/inspiring-planck-0ztqp4` branch
2. Create PR to `main` when ready
3. Merge to `main`
4. GitHub Pages (if enabled) will serve `index.html` automatically

No build step, no dependencies, no server required.

## References

- **MDN Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Game Dev Patterns**: Sprite management, collision detection, game loops
- **Raiden Mechanics**: Vertical scrolling, weapon switching, boss patterns

## Questions for Claude

Before making major changes, ask:

1. **Scope**: Is this a bug fix, feature, or refactor?
2. **Impact**: Will it affect mobile experience, frame rate, or save data?
3. **Testing**: How should we validate the change works?
4. **Backward compatibility**: Do we need to support older features?

---

**Last Updated**: 2026-09-26  
**Maintained by**: Claude Code Agent
