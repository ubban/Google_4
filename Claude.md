# Claude Development Guidelines for Google_4

This file documents guidelines and best practices for working on the Google_4 project with Claude Code.

## Project Overview

Google_4 is a web-based project containing:
- `index.html` - Main HTML file
- `README.md` - Project documentation
- `.git/` - Git repository

## Development Branch

All development work should be done on the designated feature branch:
- **Branch**: `claude/inspiring-planck-9zmghz`

## Files and Directories

### Core Files
- `index.html` - Main application file (safe to edit)
- `README.md` - Project documentation (safe to edit)

### Safe to Modify
- All HTML, CSS, and JavaScript in `index.html`
- Documentation files (*.md)
- Configuration files

### Do Not Modify Without Explicit Permission
- `.git/` - Git internals
- `.gitignore` (if present)
- Git hooks or configurations

## Git Practices

### Committing Changes
1. Always work on the designated branch: `claude/inspiring-planck-9zmghz`
2. Create clear, descriptive commit messages that explain the "why" behind changes
3. Include attribution in commit messages:
   ```
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   Claude-Session: <session-url>
   ```
4. Commit frequently with logical, atomic changes
5. Never amend published commits without explicit permission

### Pushing Changes
1. Use: `git push -u origin claude/inspiring-planck-9zmghz`
2. If network errors occur, retry with exponential backoff (2s, 4s, 8s, 16s)
3. Only push to the designated branch unless explicitly authorized

### Creating Pull Requests
- PRs are created when explicitly requested by the user
- Check for PR templates in `.github/pull_request_template.md` or similar locations
- Include proper attribution in PR descriptions
- Link to relevant issues or documentation

## Code Style and Conventions

### HTML/CSS/JavaScript
- Maintain consistent indentation (use 2 or 4 spaces)
- Use semantic HTML elements
- Write clear, self-documenting code
- Avoid unnecessary comments; only explain non-obvious logic
- Follow existing code patterns in the project

### Naming Conventions
- Use camelCase for JavaScript variables and functions
- Use kebab-case for CSS class names
- Use PascalCase for component/class names

## Testing

- Test changes locally before committing
- Verify that existing functionality is not broken
- Test edge cases and error scenarios
- Update tests if functionality changes

## Deployment

- Follow the repository's deployment procedures
- Test thoroughly before pushing to production
- Document any breaking changes
- Maintain backward compatibility when possible

## Common Tasks

### Running the Application
The project appears to be a static HTML application. Open `index.html` in a browser to run it locally.

### Making Changes
1. Edit the appropriate file
2. Test changes locally
3. Stage changes: `git add <file>`
4. Commit with clear message
5. Push to the designated branch

### Reviewing Changes
Always run a sanity check before pushing:
1. Verify the diff: `git diff`
2. Check git status: `git status`
3. Ensure no unintended files are staged
4. Test the application works as expected

## Documentation

- Keep README.md up to date with significant changes
- Document new features or major modifications
- Update this file if development guidelines change
- Use clear, concise language in all documentation

## Questions and Clarifications

If uncertain about whether a change is appropriate:
1. Check this file first
2. Review the git log for context
3. Look at the file's history and recent changes
4. Ask the user for clarification before proceeding with major changes

## Last Updated

This file was last updated on: 2026-09-29

---

For questions about Claude Code, visit: https://claude.ai/code
