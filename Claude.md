# Claude.md - Codebase Guidelines

This file provides guidance for Claude Code and other AI assistants working on this codebase.

## Project Overview

- **Repository**: ubban/google_4
- **Branch**: `claude/inspiring-planck-je0pna`
- **Type**: Web-based project

## Codebase Structure

```
.
├── README.md          # Project documentation
├── index.html         # Main HTML file
└── Claude.md          # This file
```

## Development Guidelines

### Working on Features

1. Always work on the designated feature branch: `claude/inspiring-planck-je0pna`
2. Create clear, descriptive commit messages
3. Push changes to the branch using `git push -u origin <branch-name>`

### Code Standards

- Follow existing code style and conventions
- Keep changes minimal and focused on the task
- Add comments only when the WHY is non-obvious
- Write comprehensive commit messages explaining the reasoning

### Git Practices

- Create new commits rather than amending existing ones
- Use descriptive commit messages (2-3 sentences maximum)
- Reference issue numbers when applicable
- Include proper attribution in commit messages

## Testing & Quality

- Test changes locally before pushing
- Ensure existing functionality is not broken
- Review your own diffs before committing

## Pull Request Process

- Do not create pull requests unless explicitly requested
- When creating PRs, check for existing PR templates in:
  - `.github/pull_request_template.md`
  - `.github/PULL_REQUEST_TEMPLATE.md`
  - Root `PULL_REQUEST_TEMPLATE.md`
  - `docs/PULL_REQUEST_TEMPLATE.md`

## Repository Access

- GitHub access is scoped to: `ubban/google_4`
- Do not access repositories outside this scope

## AI Assistant Instructions

### For Claude Code & AI Assistants

- **Preferred tools**: Use Read, Edit, Write, Glob, Grep for file operations
- **Avoid**: Command injection, XSS, SQL injection, and other security vulnerabilities
- **Backwards compatibility**: Avoid hacks; delete unused code completely
- **Comments**: Default to no comments; only add when WHY is non-obvious
- **Error handling**: Only validate at system boundaries (user input, external APIs)

### Refactoring Rules

- Don't introduce abstractions beyond what's needed
- Three similar lines is better than premature abstraction
- No half-finished implementations
- Focus on the immediate task, not hypothetical future requirements

## Contact & Support

For questions about Claude Code:
- `/help` - Get help with using Claude Code
- Report issues at: https://github.com/anthropics/claude-code/issues

## Session Information

- **Model**: Claude Haiku 4.5
- **Attribution**: Include proper attribution in commits and PRs as configured in system reminders
- **Email**: bobwu@irobotx.top (for authorship identification)

## Last Updated

2026-09-28
