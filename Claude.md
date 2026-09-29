# Claude.md - Sky Fighter Project

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built with vanilla JavaScript and HTML5 Canvas. The game features:
- Player aircraft with weapon switching
- Enemy formations and bosses
- Smart bomb system
- Score multiplier mechanics
- Responsive controls for both desktop and mobile

## Project Structure

- `index.html` - Main game file containing HTML, CSS (embedded), and JavaScript
- `README.md` - Basic project information

## Development Guidelines for Claude Code

### Working with the Game Code

1. **Vanilla JavaScript Approach**
   - The game is built entirely with vanilla JavaScript (no frameworks)
   - Keep it lightweight and performant
   - Avoid adding external dependencies unless absolutely necessary

2. **Canvas Rendering**
   - Game uses HTML5 Canvas for rendering
   - All drawing operations are in the embedded JavaScript
   - Performance is critical - optimize render loops and collision detection

3. **Code Organization**
   - Game initialization and setup
   - Game objects (Player, Enemies, Projectiles, Bosses)
   - Game state management
   - Input handling
   - Physics and collision detection
   - Rendering and animation
   - UI/HUD updates

### Git Workflow

- Work on the branch: `claude/inspiring-planck-pqft76`
- Push changes with: `git push -u origin claude/inspiring-planck-pqft76`
- Write clear, descriptive commit messages
- Commit message format:
  ```
  <type>(<scope>): <subject>
  
  <optional body>
  ```
  Examples:
  - `feat(player): add weapon switching animation`
  - `fix(collision): improve bullet-enemy detection`
  - `perf(rendering): optimize draw call batching`

### Testing Changes

1. Open `index.html` directly in a web browser to test the game
2. Check both desktop and mobile responsiveness
3. Test game mechanics: movement, weapons, bombs, enemy encounters
4. Verify HUD updates (health, bombs, score multiplier)

### Code Style Preferences

- Use clear, descriptive variable names
- Keep functions focused and small
- Add comments for complex game logic or subtle invariants
- Avoid premature optimization - readability first
- Use consistent formatting (2-space indentation recommended)

### Common Tasks

**Adding a new game feature:**
1. Identify the game state that needs updating
2. Add new object types or modify existing ones
3. Update the game loop logic
4. Add rendering code
5. Test thoroughly in browser

**Fixing bugs:**
1. Reproduce the issue in the browser
2. Examine the game state and logic
3. Implement minimal fix
4. Verify the fix doesn't break other mechanics

**Performance optimization:**
1. Profile in browser DevTools
2. Focus on hot paths (render loop, collision detection)
3. Measure improvement before and after

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Responsive design for both desktop and mobile
- Touch controls for mobile devices

## Known Conventions

- Game uses `requestAnimationFrame` for the main loop
- Canvas coordinates origin is top-left (0,0)
- Color-scheme: Dark theme (#05122b background, #1e6fd8 sky gradient)
- Font scaling uses CSS clamp for responsive sizing
