# Claude.md - Codebase Guide

This document provides guidelines for Claude Code when working with the Google_4 project.

## Project Overview

**Google_4** is a web-based project containing static HTML and documentation.

- **Primary files:**
  - `index.html` - Main HTML application file
  - `README.md` - Project documentation
  - `Claude.md` - This file (codebase guidelines)

## Development Workflow

### Branching Strategy
- All development work should be done on feature branches following the pattern: `claude/[branch-name]`
- Current development branch: `claude/inspiring-planck-xqc0ae`
- Never push directly to the main/master branch without explicit permission

### Git Conventions
- Write clear, descriptive commit messages
- Include the Co-Authored-By footer for attribution:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_015TFvnpefToX1ihxyKbjuiT
  ```
- Use `git push -u origin <branch-name>` for pushing changes
- If push fails due to network errors, retry up to 4 times with exponential backoff

### Pull Requests
- Do NOT create pull requests without explicit user permission
- When creating PRs, check for PR templates in `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE/`
- Include the GitHub attribution footer in PR descriptions

## Code Style & Standards

### General Guidelines
- Prefer editing existing files to creating new ones
- Be careful not to introduce security vulnerabilities (OWASP Top 10)
- Avoid backwards-compatibility hacks for unused code
- Keep code comments minimal - only add when the WHY is non-obvious

### HTML/CSS Standards
- Ensure responsive design for mobile and desktop
- Use semantic HTML elements
- Maintain accessibility standards (WCAG)
- Test changes in browser before considering complete

### Documentation
- Keep README.md up to date with project status
- Add comments only for non-obvious logic
- Document any build or setup steps needed

## File Structure

```
Google_4/
├── .git/              # Git repository
├── README.md          # Project documentation
├── index.html         # Main application file
└── Claude.md          # This file
```

## Testing & Validation

- Test UI changes in a browser before reporting completion
- Verify both golden path and edge cases
- Monitor for regressions in existing features
- Run type checking and test suites when available

## Common Tasks

### Adding Features
1. Create a feature branch from the development branch
2. Make changes to the relevant files (primarily index.html)
3. Test changes thoroughly in browser
4. Commit with clear message
5. Push to the feature branch
6. Wait for explicit approval before creating a PR

### Bug Fixes
1. Identify the issue in the codebase
2. Fix the root cause (don't create workarounds)
3. Test the fix thoroughly
4. Commit and push
5. Request review if needed

### Documentation Updates
1. Update README.md or relevant documentation files
2. Keep documentation in sync with code changes
3. Commit and push

## Environment

- **Working Directory:** `/home/user/Google_4`
- **Repository:** ubban/google_4
- **Scratchpad:** `/tmp/claude-0/-home-user-Google-4/[session-id]/scratchpad` for temporary files

## Important Notes

- This is a remote execution environment - files must be committed and pushed to be saved
- The container is ephemeral and reclaimed after inactivity
- Always verify changes before pushing
- No user is watching live during scheduled tasks - surface important findings via notifications

## Contact & Support

- User Email: bobwu@irobotx.top
- For Claude Code questions: Use `/help` or report issues at https://github.com/anthropics/claude-code/issues

---

Last Updated: 2026-10-01
