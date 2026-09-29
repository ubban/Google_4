# Claude.md - Codebase Guidelines

## Project Overview
This repository contains the Google_4 project. The codebase uses Claude Code for automated development and maintenance tasks.

## Development Practices

### Git Workflow
- **Primary branch**: `main` (or default branch for releases)
- **Feature branches**: Use descriptive names (e.g., `feature/description`, `fix/issue-name`)
- **Commit messages**: Include clear, descriptive messages following conventional commit format
  - Format: `<type>(<scope>): <subject>`
  - Types: feat, fix, docs, style, refactor, test, chore
  - Example: `feat(parser): add support for new format`

### Code Style
- Maintain consistency with existing code patterns
- Use meaningful variable and function names
- Keep functions focused and single-purpose
- Add comments only when the WHY is non-obvious

### Automated Tasks
- Scheduled tasks run on their designated branches as specified
- Changes should be committed and pushed to the appropriate branch
- Status updates and notifications are sent for significant findings

## Automation Rules
- Commits made by Claude Code should include proper attribution
- Only push to branches explicitly designated in the task
- Never force-push unless explicitly authorized
- Always test changes before pushing

## Attribution
When Claude Code commits changes:
```
Co-Authored-By: Claude <noreply@anthropic.com>
```

## File Structure
- `index.html` - Main project file
- `README.md` - Project documentation
- `Claude.md` - This file; guidelines for Claude Code interaction

## Repository Access
This session operates under the following scope:
- Repository: `ubban/google_4`
- Development branch: `claude/inspiring-planck-7ggqen`

## Notes for Future Updates
- Keep this file updated as coding standards evolve
- Document any new project-specific conventions
- Maintain this as a single source of truth for Claude Code operations
