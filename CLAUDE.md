# Claude.md - Sky Fighter Project Guidelines

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game implemented in HTML/JavaScript. The entire game is self-contained in `index.html` with embedded CSS and JavaScript.

## Project Structure
- `index.html` - Main game file (HTML, CSS, and JavaScript all embedded)
- `README.md` - Project documentation
- `.git/` - Version control

## Running the Game
1. Open `index.html` in a modern web browser
2. The game runs directly with no build process or dependencies required
3. Full-screen play is supported on both desktop and mobile devices

## Development Guidelines

### Code Organization
- The game uses a single HTML file for simplicity
- CSS is embedded in a `<style>` block
- JavaScript logic is embedded in a `<script>` block
- The canvas element is used for rendering game graphics

### Browser Compatibility
- Requires ES6+ JavaScript support
- Uses CSS Grid, Flexbox, and modern web APIs
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)
- Supports touch input for mobile devices

### Game Features
- Vertical scrolling shoot-'em-up gameplay
- Weapon switching mechanics
- Smart bomb ability
- Stage bosses and enemies
- Score tracking and game state management

## Making Changes

### Testing Changes
1. Modify the game logic in the `<script>` section of `index.html`
2. Reload the browser to test changes
3. Verify gameplay mechanics work correctly (controls, collisions, scoring)
4. Test on both desktop and mobile if possible

### Commit Guidelines
- Provide clear, descriptive commit messages
- Test changes in-game before committing
- Use the branch: `claude/inspiring-planck-4hwew7`
- Push all changes to the designated branch

### Common Tasks
- **Bug fixes**: Modify the relevant game logic in the `<script>` section
- **Feature additions**: Add new game mechanics or UI elements
- **Performance improvements**: Optimize render loops and collision detection
- **Visual tweaks**: Modify CSS styles or canvas rendering code

## Notes for Claude Code Agents
- The entire codebase is in a single file, so changes are straightforward
- Always test gameplay after making changes
- The game uses a main game loop running at display refresh rate
- Collision detection and enemy/bullet management are core systems
- Mobile support is important—test touch controls if modifying input handling
