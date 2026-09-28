# Claude.md - Development Guide for Google_4

## Project Overview

**Google_4** is a Sky Fighter: Raiden-style vertical shoot-em-up game built with HTML5 and JavaScript. This is a static HTML project with game logic implemented entirely in the browser.

## Project Structure

- `index.html` - Main game implementation with HTML, CSS, and JavaScript
- `README.md` - Basic project information
- `.git/` - Git repository metadata

## Development Guidelines

### Code Style

- Use vanilla JavaScript (no external dependencies required)
- Keep all code modular and maintainable within a single HTML file
- Use clear variable and function names
- Add comments for non-obvious game logic

### Testing

- Test game features manually in a browser before committing
- Verify:
  - Game starts correctly
  - Player controls respond as expected
  - Enemies spawn and behave properly
  - Collision detection works
  - Score tracking functions
  - Game over and win conditions

## Claude Code Automation

### Branches

All development work should be done on feature branches:
- **Primary branch**: `claude/inspiring-planck-t066hw`
- Always create new commits rather than amending existing ones
- Use clear, descriptive commit messages

### Git Workflow

```bash
# Start work on the feature branch
git checkout claude/inspiring-planck-t066hw

# Make changes and commit
git add <files>
git commit -m "Clear description of changes"

# Push to remote
git push -u origin claude/inspiring-planck-t066hw
```

### Commit Message Format

Include a clear summary of changes:
```
Add feature X or Fix issue Y

- Specific change 1
- Specific change 2
```

Attribution is automatically added by Claude Code.

### Pull Requests

- PRs are created automatically when needed for code review
- If a PR template exists, its structure is preserved
- Include details about testing and any behavioral changes

## Common Tasks

### Adding Game Features

1. Modify game logic in index.html
2. Test changes in browser (open index.html or run a local server)
3. Commit with descriptive message
4. Push to feature branch

### Bug Fixes

1. Identify the issue and affected code
2. Fix and test in browser
3. Commit with reference to the issue
4. Push changes

### Performance Improvements

- Use browser DevTools to identify bottlenecks
- Test frame rate and memory usage after changes
- Document any significant optimizations in commit message

## Notes for Claude Code

- This project requires manual browser testing; automated tests aren't available
- The game is self-contained in a single HTML file
- Changes are immediately visible when opening the file in a browser
- Focus on code quality and game functionality
- Keep features in scope and avoid over-engineering
