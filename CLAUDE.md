# Claude Development Guidelines for Google_4

## Project Overview

**Google_4** is a Raiden-style vertical shoot-em-up game (Sky Fighter). It's a browser-based game built with HTML5/Canvas and JavaScript, featuring classic arcade-style gameplay with vertical scrolling action.

## Key Files and Structure

- `index.html` - Main entry point containing the full game implementation
- `README.md` - Project metadata and timestamp
- `.git/` - Git version control

## Project Type

- **Frontend**: Browser-based game
- **Technology**: HTML5, Canvas API, Vanilla JavaScript
- **No Build Process**: Direct browser execution

## Running the Project

To run the game:
1. Open `index.html` directly in a web browser, or
2. Use a local web server: `python3 -m http.server 8000` (or similar)
3. Navigate to `http://localhost:8000/index.html`

## Development Guidelines

### Code Style
- Use vanilla JavaScript (no frameworks)
- Keep game logic contained within the canvas context
- Maintain consistent indentation and naming conventions
- Add comments only when behavior is non-obvious

### Testing
- Test gameplay manually in the browser
- Verify controls are responsive
- Check collision detection and game mechanics
- Test across different browser sizes and refresh rates

### Git Workflow
- Development branch: `claude/inspiring-planck-wxowo3`
- Commit messages should describe the feature or fix clearly
- Use present tense: "Add feature" not "Added feature"

## Common Tasks

### Adding Game Features
- Modify the game logic in the `<script>` section of index.html
- Test immediately in browser for visual/gameplay feedback
- Key areas: enemy patterns, player weapons, collision detection

### Bug Fixes
- Identify the issue location in index.html script section
- Fix and test in browser
- Commit with descriptive message explaining the fix

### Performance Optimizations
- Use browser DevTools to profile performance
- Focus on render loop efficiency
- Optimize collision detection for many objects

## Known Patterns

- Game runs in a single HTML file for simplicity
- Canvas-based rendering with requestAnimationFrame
- Sprite/object management through arrays
- Keyboard input handling via event listeners

## Claude-Specific Notes

- This is a complete, self-contained project
- No external dependencies or build steps required
- Test gameplay after any changes - this is a visual medium
- When reviewing code, pay attention to game logic correctness and performance
- The project demonstrates classic arcade game patterns

## Attribution

Commits authored by Claude should include appropriate co-author attribution in git commits.
