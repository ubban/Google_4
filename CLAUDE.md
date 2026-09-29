# Claude Development Guidelines for Google_4

This document provides guidelines for Claude Code agents working on the Google_4 project.

## Project Overview

Google_4 is a web-based project containing:
- Static HTML/web content (index.html)
- Project documentation (README.md)
- Git repository for version control

## Development Branch

All development work on this project should be done on the branch:
```
claude/inspiring-planck-sf0m0d
```

Always create new changes on this branch and push to it.

## Git Workflow

### Commits
- Write clear, descriptive commit messages that explain the "why" behind changes
- Keep commits focused and atomic (one logical change per commit)
- Include the co-author attribution line in commit messages:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01Fctm6y6n12xqc7g9ZLXM7B
  ```

### Pushing Changes
- Always push to the designated development branch using:
  ```
  git push -u origin claude/inspiring-planck-sf0m0d
  ```
- If network errors occur, retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

### Pull Requests
- Create pull requests when explicitly requested
- For PR descriptions, include:
  - Summary of changes
  - Rationale and context
  - Testing performed
  - Attribution footer: `🤖 Generated with [Claude Code](https://claude.com/claude-code)`

## Code Style & Standards

### General Principles
- Write minimal, focused code - avoid over-engineering
- No unnecessary comments or docstrings
- Comments should explain "why", not "what" the code does
- Delete unused code completely rather than leaving commented-out code or TODO comments
- Avoid backwards-compatibility hacks

### HTML/Web Content
- Keep HTML clean and semantic
- Use standard markup practices
- Test changes in the browser before marking complete
- Verify the golden path and edge cases work correctly

## Testing & Validation

- Run any available tests before committing
- Test UI changes in a browser to verify they work as intended
- Check for regressions in existing features when making changes
- Use type checking and linting tools if available in the project

## Security

- Never commit secrets, credentials, or sensitive information
- Be cautious of security vulnerabilities (XSS, SQL injection, etc.)
- Validate user input at system boundaries
- Use safe, secure coding practices by default

## Documentation

- Keep README.md updated with project information
- Document any significant architectural decisions
- Maintain clarity for future developers
- Use markdown formatting for consistency

## File Organization

```
Google_4/
├── README.md           # Project documentation
├── CLAUDE.md           # This file - Claude development guidelines
├── index.html          # Main web content
└── .git/               # Git repository
```

## Common Tasks

### Updating Project Documentation
- Modify README.md with accurate project information
- Keep timestamps and version information current

### Adding Web Content
- Add files to the root or appropriate subdirectories
- Test in browser before committing
- Update this document if structure changes

### Maintenance
- Run `git status` before making changes to check state
- Keep working tree clean
- Archive or delete temporary files before committing

## Automated Scheduled Tasks

This project may have scheduled Claude Code agents running automated maintenance tasks:
- Updates to CLAUDE.md documentation
- Code quality checks and improvements
- Dependency monitoring and updates
- Regular codebase audits

## Contact & Issues

For questions or issues:
- Check this CLAUDE.md file for guidelines
- Review git history for context on past decisions
- Use `/help` for Claude Code CLI assistance
- Report issues at https://github.com/anthropics/claude-code/issues

---
*Last Updated: 2026-09-29*
