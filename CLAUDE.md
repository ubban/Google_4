# Claude Code Guidelines for Google_4

## Project Overview

**Google_4** is a game development project featuring **Sky Fighter**, a Raiden-style vertical shoot-em-up game built with HTML5. This is the primary deliverable of the project.

## Development Workflow

### Branch Strategy

- **Primary Development Branch**: `claude/inspiring-planck-mrdwq6`
  - All Claude Code agent work should be developed on this branch
  - Push changes with: `git push -u origin claude/inspiring-planck-mrdwq6`
  - Ensure clear, descriptive commit messages
  
- **Main Branch**: `main`
  - Production-ready code
  - Merges come from feature/development branches via pull requests

### Commit Message Convention

All commits should follow this format:
```
<type>: <description>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01726xkRWip7hVuXeCXfq8ba
```

Where `<type>` is one of:
- `feat`: New feature or game mechanic
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `docs`: Documentation updates
- `perf`: Performance improvements
- `style`: Code style/formatting

## Project Structure

- `index.html`: Main game file containing the Sky Fighter game implementation
- `README.md`: Project information
- `CLAUDE.md`: This file - guidelines for Claude Code agents

## Game Development Guidelines

### Code Quality
- Keep game code well-organized and commented where logic is non-obvious
- Maintain separation between game logic, rendering, and input handling
- Test gameplay mechanics before committing changes

### Testing Gameplay
Before committing game changes:
1. Open `index.html` in a browser
2. Test the primary game mechanics (movement, shooting, enemies, collisions)
3. Verify no regressions in existing features
4. Test on different screen sizes if applicable

### Git Operations

- **Pull Strategy**: `git pull origin <branch-name>` for specific branches
- **Push Strategy**: `git push -u origin <branch-name>` for new branches or updates
- **Retry Logic**: On network failures, retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
- **Avoid**: Destructive operations without explicit user approval (force push, hard reset, history rewriting)

## Important Notes

- Pull requests are only created when explicitly requested by users
- When creating PRs, include a clear summary of changes
- All development commits require attribution footer (see Commit Message Convention above)
- This is an automated workflow - follow the guidelines for consistency

---

_Last updated: 2026-09-28_
