# Claude Code Guidelines for Google_4

## Project Overview
Google_4 is a web-based project with the following structure:
- `index.html` - Main HTML file containing the application
- `README.md` - Basic project documentation
- `.git/` - Git version control

## Working with Claude Code

### Branch Information
- Default development branch: `claude/inspiring-planck-i0gkbm`
- All changes should be committed to the designated branch specified in the development session

### Git Practices
- Create descriptive commit messages that explain the "why" not just the "what"
- Use clear commit messages with the format: "[type] description"
  - Examples: "[feat] add new feature", "[fix] resolve bug", "[docs] update documentation"
- Always push changes to the designated branch with `git push -u origin <branch-name>`

### Project File Conventions
- HTML files use standard markup conventions
- Keep documentation updated in README.md and this file (CLAUDE.md)
- Test changes in a browser before marking features as complete

### Code Quality Standards
- Ensure HTML is semantic and accessible
- Follow modern web standards
- No unnecessary comments - only add when the WHY is non-obvious
- Keep commits focused and reversible

### Testing & Validation
- Test changes in a real browser environment when making UI changes
- Verify both the happy path and edge cases
- Watch for regressions in existing features

## Attribution
When making commits or PRs, include:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session-url>
```

## Last Updated
2026-09-26 (Automated by Claude Code agent)
