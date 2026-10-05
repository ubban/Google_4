# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS, and JavaScript. The game features:
- Player-controlled aircraft with weapon switching
- Smart bomb mechanics
- Progressive difficulty and stage bosses
- Responsive design for desktop and mobile

**Repository**: `ubban/Google_4`
**Development Branch**: `claude/inspiring-planck-h1rl30`

## Project Structure

```
Google_4/
├── README.md          # Project metadata and timestamp
├── index.html         # Main game application (single-file)
├── .git/              # Git repository
└── CLAUDE.md          # This file
```

## Key Files

### index.html
The entire game is contained in a single HTML file (`index.html`). It includes:
- **HTML**: Canvas element for rendering and HUD displays
- **CSS**: Styling for dark theme, responsive layout, and HUD elements
- **JavaScript**: Complete game logic including:
  - Game loop and physics
  - Entity management (player, enemies, projectiles, effects)
  - Weapon systems and smart bombs
  - Collision detection
  - Stage/level progression and boss fights
  - Input handling (keyboard, touch, mouse)
  - Audio system (if implemented)

## Architecture

### Single-File Design
The entire game is in `index.html` to ensure:
- Self-contained deployment (no external dependencies)
- Easy testing in any browser
- Minimal file management complexity

### Game Loop Structure
- **Update**: Input processing, entity updates, collision detection
- **Render**: Canvas drawing and HUD updates
- **Frame Timing**: Uses `requestAnimationFrame` for smooth 60 FPS gameplay

### Key Systems
1. **Entity System**: Player, enemies, projectiles, explosions managed in arrays
2. **Weapon System**: Multiple weapon types with switch mechanics
3. **Collision System**: Rectangular or circular collision detection
4. **Score/Wave System**: Track player progress through stages

## How to Run

### Local Development
```bash
# Open in browser
open index.html

# Or use a local server for better CORS/testing
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Testing
- Test gameplay across desktop browsers (Chrome, Firefox, Safari, Edge)
- Test mobile responsiveness on various device sizes
- Test input methods: keyboard, touch, mouse

## Development Workflow

### Making Changes
1. Edit `index.html` directly
2. Save and reload browser (F5 or refresh)
3. Test gameplay features thoroughly before committing
4. For significant gameplay changes, test on multiple devices

### Code Organization Guidelines
When modifying JavaScript in `index.html`:
- Keep related functionality in the same scope
- Use descriptive variable names for game state
- Add comments for complex collision or physics logic
- Maintain existing code structure to minimize conflicts

### Committing Changes
```bash
# Follow conventional commit format
git add index.html CLAUDE.md
git commit -m "feat: add new weapon type" -m "Description of changes"

# Push to development branch
git push -u origin claude/inspiring-planck-h1rl30
```

## Git Practices

### Branches
- **Main Development**: `claude/inspiring-planck-h1rl30`
- Always commit changes to the designated branch
- Push changes before session ends

### Commit Messages
Follow conventional commit format:
- `feat:` New feature
- `fix:` Bug fix
- `perf:` Performance improvement
- `refactor:` Code restructuring (no feature/bug changes)
- `style:` Code style changes (formatting, whitespace)
- `docs:` Documentation changes

### Pull Requests
Only create PRs when explicitly requested. Include:
- Clear description of changes
- Testing notes
- Breaking changes (if any)

## Important Notes

### Single-File Constraint
This game is intentionally kept as a single HTML file. When adding features:
- Do not split into multiple files unless explicitly approved
- Keep bundle size reasonable
- Consider performance impact of new features

### Browser Compatibility
- Target modern browsers (Chrome, Firefox, Safari Edge)
- Ensure mobile touch input works correctly
- Test before committing gameplay changes

### Performance Considerations
- Entity count management (remove off-screen entities)
- Efficient collision detection
- Smooth frame rate on lower-end devices
- Image/asset optimization if any are added

## Common Tasks

### Adding a New Weapon
1. Locate weapon definitions in JavaScript
2. Define weapon properties (damage, fire rate, projectile type)
3. Add weapon switching logic
4. Test weapon functionality thoroughly

### Adding a New Enemy Type
1. Define enemy class/properties in entity system
2. Implement movement and attack patterns
3. Add spawning logic
4. Balance difficulty

### Adjusting Game Balance
- Weapon damage/fire rates
- Enemy health/speed
- Score multipliers
- Stage progression difficulty curve

## Session Notes

- Last updated: 2026-10-05
- Project is stable and playable
- All changes should be tested in-game before committing
- Focus on gameplay quality and responsiveness

---

For questions or issues, refer to the game logic in `index.html` and test thoroughly in a browser environment.
