# Claude Code Configuration for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The game features weapon switching, smart bombs, and stage bosses.

- **Tech Stack**: HTML5, JavaScript (vanilla), Canvas API
- **Type**: Web-based arcade game
- **Repository**: ubban/google_4

## Development Branch

All Claude Code work on this project is developed on the feature branch:
```
claude/inspiring-planck-cprqvq
```

- **Never** push directly to `main` without explicit user approval
- Always commit to the feature branch designated above
- Create PRs from the feature branch when requesting review/merge

## Commit Message Guidelines

Commit messages should follow these conventions:

- **Format**: `<type>: <short description>`
- **Types**:
  - `feat`: New feature or game mechanic
  - `fix`: Bug fix
  - `refactor`: Code refactoring without behavior change
  - `perf`: Performance improvements
  - `style`: Code formatting, naming (no functional change)
  - `docs`: Documentation updates
  - `chore`: Dependency updates, tooling changes

- **Example**:
  ```
  feat: add weapon switching mechanic with keyboard controls
  ```

- **Attribution**: All commits must end with:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: <session-url>
  ```

## Code Review & Quality

- Code should be reviewed for **correctness** and **functionality**
- Prioritize testing features in the actual game (not just unit tests)
- For game mechanics: verify they work as intended in-game
- For visual/UI changes: test rendering and animations
- Keep changes focused and minimal—no over-engineering

## Testing

Since this is a single-file HTML game:
- Test gameplay features manually in a browser
- Verify no console errors
- Test on both desktop and mobile where applicable
- Ensure canvas scaling and responsive behavior work correctly

## Game Development Notes

- The game uses Canvas 2D context for rendering
- Input handling: keyboard (desktop) and touch (mobile)
- HUD displays game stats (score, health, level)
- Stage progression with boss battles
- Weapon and bomb systems

## Pull Request Guidelines

When creating a pull request:
1. Include a clear description of changes
2. Reference any related issues or features
3. Note any testing performed
4. Mention if the change affects gameplay, visuals, or performance
5. PR description should end with the Claude Code attribution

## Branch Strategy

- `main`: Stable, released version
- `claude/inspiring-planck-cprqvq`: Active development branch for Claude agents
- Other branches: Must be created with explicit user request

## Code Style

- Use descriptive variable and function names
- Comment only for non-obvious logic (WHY, not WHAT)
- Keep functions focused and reasonably sized
- No unnecessary abstractions for hypothetical future use
- Prefer simplicity over premature optimization
