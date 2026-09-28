# Sky Fighter - Claude.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas, and JavaScript. The game features:
- Vertical scrolling gameplay
- Player aircraft with weapon switching
- Enemy formations and bosses
- Smart bomb power-ups
- Score tracking and lives system

This is a single-file game project (index.html) with embedded HTML, CSS, and JavaScript.

## Project Structure

```
Google_4/
├── index.html       # Complete game implementation (HTML + CSS + JavaScript)
├── Claude.md        # This file - guidance for Claude Code sessions
└── README.md        # Project description
```

## Running the Game

1. **Browser Play**: Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local Server** (optional): Serve via HTTP for better security context
   ```bash
   python -m http.server 8000
   # Then visit http://localhost:8000
   ```

## Controls

- **Mouse/Touch**: Move your aircraft with mouse cursor or touch position
- **Click/Tap**: Shoot (automatic on desktop, requires manual click on mobile for performance)
- **Spacebar**: Drop smart bomb (when available)
- **P**: Pause/Resume game

## Code Organization

The `index.html` file is organized as follows:

### Structure
1. **HTML Head** - Metadata, viewport, theme color, favicon
2. **CSS Styles** - Game canvas styling, HUD layout, responsive design
3. **HTML Body** - Canvas element and HUD display
4. **JavaScript** - All game logic embedded in one `<script>` block

### Key Game Modules (within the script)
- **Canvas & Input**: Canvas setup, mouse/touch event handlers
- **Game State**: Player ship, enemies, projectiles, power-ups, score, lives
- **Game Loop**: Animation frame loop, collision detection, rendering
- **Player**: Aircraft position, movement, shooting, weapon states
- **Enemies**: Enemy types, movement patterns, spawning logic
- **Collision Detection**: Bullet-enemy, enemy-player interactions
- **Rendering**: Draw game objects, HUD, backgrounds
- **Effects**: Visual feedback, explosions, scoring

## Development Guidelines

### When Modifying This Project

1. **Edit Strategy**: Since the entire game is in one file, make surgical edits to specific sections:
   - Locate the exact JavaScript object or function you're modifying
   - Preserve indentation and formatting
   - Keep related logic together (e.g., all player code, all enemy code)

2. **Testing**: Always test in a browser after changes
   - Open index.html in your default browser
   - Play through several waves to verify:
     - Enemy spawning works correctly
     - Collisions are accurate
     - Score updates properly
     - Boss behavior matches expectations

3. **No Build Step**: This project has no build process or dependencies
   - Pure HTML5 with no npm, webpack, or compilation needed
   - All assets are inline (SVG, embedded data)

4. **Performance Considerations**:
   - Canvas rendering is the performance bottleneck
   - Limit concurrent enemies and projectiles
   - Test on lower-end devices/older browsers
   - Use requestAnimationFrame (already implemented) for smooth 60fps

5. **Responsive Design**:
   - Game scales with viewport using CSS transformations
   - Touch controls should work on mobile
   - Test on various screen sizes

## Code Review Guidelines

When reviewing changes to this project, check for:

1. **Correctness**:
   - Collision detection logic is accurate
   - Game state remains consistent
   - Score/lives updates are correct
   - Enemy spawn patterns work as intended

2. **Performance**:
   - No infinite loops or unnecessary calculations per frame
   - Asset creation happens once, not every frame
   - Efficient collision detection (avoid O(n²) where possible)

3. **Gameplay Balance**:
   - Enemy difficulty scales appropriately with waves
   - Boss patterns are challenging but fair
   - Weapon variety provides meaningful choices
   - Power-ups appear at reasonable frequency

4. **Code Quality**:
   - Comments explain complex game logic (collision, AI patterns)
   - Variable names clearly indicate purpose (player, enemy, projectile)
   - Related code is kept together in logical sections
   - No dead/unused code

5. **Compatibility**:
   - Works without external dependencies
   - Compatible with major browsers
   - Mobile touch support works smoothly

## Commit Message Guidelines

Use clear, descriptive commit messages following this pattern:

```
Type: Brief description

Optional longer explanation of the change and why.

- Point 1 about the implementation
- Point 2 about the impact
```

**Types**:
- `feat:` New feature (new weapon, enemy type, game mode)
- `fix:` Bug fix (collision, spawning, scoring)
- `refactor:` Code reorganization without behavior change
- `perf:` Performance improvement
- `style:` Code formatting or organization
- `docs:` Documentation or comments

**Examples**:
- `feat: Add triple-shot weapon option`
- `fix: Correct enemy collision boundary detection`
- `perf: Optimize projectile rendering with culling`

## Common Tasks

### Adding a New Enemy Type
1. Locate the enemy definition object in the game code
2. Add a new type with position, size, health, movement pattern
3. Add drawing code in the render section
4. Test spawning and collision

### Adjusting Game Difficulty
1. Find the wave spawning logic
2. Modify enemy count or spawn rate per wave
3. Adjust boss health or attack patterns
4. Test difficulty curve

### Adding a New Weapon
1. Add weapon definition to the player object
2. Implement firing logic in the shooting section
3. Add weapon switching logic
4. Test all weapon types work correctly

### Fixing a Visual Bug
1. Locate the rendering code for the affected element
2. Check coordinate calculations and dimensions
3. Verify CSS styles aren't interfering
4. Test on multiple browsers/resolutions

## Testing Checklist

Before committing changes:
- [ ] Game starts without errors
- [ ] Player can move and shoot
- [ ] Enemies spawn and move correctly
- [ ] Collisions register properly
- [ ] Score updates on enemy defeat
- [ ] All weapons function
- [ ] Smart bombs work as expected
- [ ] Boss appears and behaves correctly
- [ ] Game ends when lives reach zero
- [ ] No console errors

## Resources & References

- **Game Design**: Vertical shooter with progressive difficulty
- **Browser APIs**: Canvas 2D Context, MouseEvent, TouchEvent, KeyboardEvent
- **Performance**: RequestAnimationFrame for 60fps rendering
- **Accessibility**: Keyboard and touch controls supported

## Notes for Future Sessions

- This is a learning/demo project, not a production game
- Single-file architecture keeps it simple but limits scalability for major features
- Consider splitting into modules if game scope significantly expands
- All visual assets are procedurally generated or SVG-based (no external image files)
