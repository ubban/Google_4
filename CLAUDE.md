# CLAUDE.md - Google_4 Project Guidelines

## Project Overview

Google_4 is a web-based project with an HTML interface. This document provides guidance for Claude Code agents and developers working on this codebase.

## Repository Structure

- `index.html` - Main HTML file (58KB) - contains the primary application interface and logic
- `README.md` - Project documentation
- `.git/` - Git repository metadata

## Development Guidelines

### Code Style & Conventions

- Follow HTML5 standards
- Keep code clean and maintainable
- Use semantic HTML when appropriate
- Minimize external dependencies when possible

### Git Workflow

1. Always work on the designated feature branch: `claude/inspiring-planck-4avzdp`
2. Create clear, descriptive commit messages
3. Push changes using `git push -u origin <branch-name>`
4. Include attribution lines in commits for Claude-assisted work

### Testing

Before committing changes:
- Validate HTML syntax
- Test changes in a browser (Chromium is available)
- Ensure no existing functionality is broken

### File Modifications

When modifying `index.html`:
- Preserve existing functionality unless specifically removing it
- Update related documentation if needed
- Test thoroughly in browser before pushing

## Commit Message Format

For Claude-assisted work, use the following format:
```
<description of changes>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session-url>
```

## Pull Requests

- Only create PRs when explicitly requested
- Use clear PR titles (under 70 characters)
- Reference related issues or feature requests in the description
- Include testing details in the PR body

## Special Instructions

None currently configured. Update this section if new constraints or patterns emerge.

## Contact & Support

For questions about working with this codebase, refer to the README.md or consult the project maintainer.

---

**Last Updated**: 2026-09-29  
**Updated By**: Claude Code Agent
