# Claude Code Guidelines for Google_4

## Project Overview

Google_4 is a web-based project containing core functionality in index.html. This repository is configured for automated Claude Code development with scheduled tasks and pull request automation.

## Development Environment

- **Primary Language**: HTML/JavaScript
- **Working Directory**: `/home/user/Google_4`
- **Repository**: ubban/google_4 on GitHub
- **Development Branch**: `claude/inspiring-planck-s7cgdv`

## Getting Started with Claude Code

### Branch Strategy

- All development work must be done on the designated feature branch: `claude/inspiring-planck-s7cgdv`
- Never push directly to main or master branches
- Push changes with: `git push -u origin <branch-name>`
- If push fails due to network errors, retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

### File Structure

```
Google_4/
├── .git/              # Git repository metadata
├── .claude/           # Claude Code configuration (create as needed)
├── README.md          # Project readme
├── index.html         # Main application file
└── Claude.md          # This file - Claude Code guidance
```

## Working with Claude Code

### Before Starting Development

1. Check the current branch: `git status`
2. Fetch latest changes: `git fetch origin`
3. Create or switch to feature branch if needed

### During Development

- Use the file editing tools (Edit, Write, Read) for code changes
- Run bash commands for testing and validation
- Make incremental, atomic commits with clear messages
- Test changes locally before pushing

### Commit Guidelines

All commits must include proper attribution. Append the following to commit messages:

```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01Tuv3oHJK7iqDMScPeqGrQQ
```

Example commit:
```
git commit -m "Add feature X to improve functionality

Detailed explanation of the change.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01Tuv3oHJK7iqDMScPeqGrQQ"
```

### Push Workflow

After committing changes:

```bash
# Verify changes are ready
git status

# Push to the designated branch
git push -u origin claude/inspiring-planck-s7cgdv
```

## Testing and Validation

- For HTML/JavaScript changes, open `index.html` in a browser to verify functionality
- Use browser developer tools to check for console errors
- Validate markup with HTML validators when applicable
- Test responsive design on multiple screen sizes

## Pull Request Guidelines

When creating pull requests:

1. Check for existing PR templates in `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE/`
2. Use clear, descriptive titles (under 70 characters)
3. Include the attribution footer in PR descriptions:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_01Tuv3oHJK7iqDMScPeqGrQQ
```

4. Do NOT create PRs without explicit user permission
5. Provide detailed description of changes and test methodology

## Code Quality Standards

- Keep code clean and maintainable
- Avoid unnecessary abstractions or premature optimization
- Write minimal comments - only explain the "why" when non-obvious
- Use meaningful variable and function names
- Remove unused code rather than leaving placeholder comments

## Automated Tasks and Scheduling

This repository may run automated scheduled tasks via Claude Code:

- Scheduled tasks are configured through the system and run autonomously
- No live user interaction is required for scheduled runs
- Results are captured in the session transcript
- Notifications are sent when important events occur

## Common Development Tasks

### Update Code
1. Read the relevant file
2. Edit the file with necessary changes
3. Test locally
4. Commit with proper attribution
5. Push to feature branch

### Debug Issues
1. Check git status and recent commits
2. Review browser console for errors
3. Use grep or search tools to find related code
4. Make targeted fixes
5. Push changes

### Review Changes Before Pushing
```bash
git diff                    # See unstaged changes
git diff --staged          # See staged changes
git log --oneline -5       # See recent commits
```

## Important Restrictions

- ❌ Do NOT push to main/master without explicit permission
- ❌ Do NOT force push to shared branches
- ❌ Do NOT skip pre-commit hooks with --no-verify
- ❌ Do NOT commit secrets or credentials
- ❌ Do NOT delete branches without confirmation

## Contacting Support

For questions about Claude Code functionality:
- Documentation: https://claude.ai/code
- Issue reporting: https://github.com/anthropics/claude-code/issues
- Help command: `/help` in Claude Code

## Session Information

- **Current Session ID**: 01Tuv3oHJK7iqDMScPeqGrQQ
- **Model**: Claude Haiku 4.5 (claude-haiku-4-5-20251001)
- **User Email**: bobwu@irobotx.top (for attribution only)
- **Last Updated**: 2026-09-30

---

This Claude.md file provides guidance for Claude Code automation and development workflows on the Google_4 repository. Update this file when significant changes to the development process occur.
