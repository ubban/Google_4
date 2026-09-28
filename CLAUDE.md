# Claude Code Guidelines for Google_4

## Project Overview

Google_4 is a web-based game repository containing interactive game implementations including Sky Fighter (a Raiden-style vertical shoot-em-up game). The project uses HTML, CSS, and JavaScript for game development.

## Development Workflow

### Branch Strategy
- **Development Branch**: `claude/inspiring-planck-iovul6` - Use this branch for feature development and bug fixes
- **Main Branch**: `main` - Stable release branch, do not push directly
- When starting work, always ensure you're on the designated development branch

### Commit Message Conventions
All commits should include:
- Clear, descriptive message explaining the change
- Co-author attribution: `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>`
- Session reference for tracking

Format:
```
<type>: <description>

<optional detailed explanation>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01LKacxHug6fG1i5HDr8EdFX
```

Commit types:
- `feat:` - New feature or game
- `fix:` - Bug fix
- `refactor:` - Code refactoring without changing functionality
- `style:` - Code style changes (formatting, missing semicolons, etc.)
- `docs:` - Documentation updates
- `test:` - Adding or updating tests

## File Structure

```
Google_4/
├── README.md              # Project overview
├── CLAUDE.md              # This file - Claude Code guidelines
├── index.html             # Main entry point (games and UI)
└── .git/                  # Git repository metadata
```

## Code Quality Guidelines

### HTML/CSS/JavaScript
- Write clean, semantic HTML
- Use CSS for styling with a clear structure
- Comment non-obvious JavaScript logic
- Keep game logic modular and testable
- Test games manually in a browser before committing

### Pull Requests
- PRs are created from the development branch
- Always include a summary of changes
- Provide testing instructions
- Reference any related issues

## Testing

- Test games in multiple browser environments when possible
- Verify responsive design on different screen sizes
- Test game mechanics and controls thoroughly
- Performance should be acceptable on standard computers

## Deployment

- Merge from development branch to `main` only when features are stable
- Tag releases with semantic versioning when ready for deployment
- Update README.md with new game additions

## Claude Code Specific Rules

1. **Always develop on the designated branch** - `claude/inspiring-planck-iovul6`
2. **Push changes regularly** with clear commit messages
3. **Test functionality** before committing game code
4. **Ask for confirmation** before force-pushing or destructive git operations
5. **Keep commits atomic** - one logical change per commit
6. **Document significant changes** in commit messages

## Getting Started

When starting a new Claude Code session:
1. Ensure you're on branch `claude/inspiring-planck-iovul6`
2. Review the latest changes with `git log --oneline -5`
3. Test any existing games to understand current functionality
4. Make focused changes for the task at hand
5. Commit with descriptive messages
6. Push to the development branch

## Contact & Attribution

All Claude Code sessions should attribute work properly with:
- Session tracking URLs
- Model identification when relevant
- Clear authorship for future reference
