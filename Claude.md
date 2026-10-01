# Claude Code Guidelines for Google_4

This file documents guidelines and conventions for Claude Code and AI assistants working on the Google_4 project.

## Project Overview

**Repository**: ubban/google_4
**Primary Branch**: main
**Development Branch**: claude/inspiring-planck-fiz9ts

This project is a web-based application with HTML/JavaScript components. The main deliverable is accessible via index.html.

## Development Workflow

### Branch Management
- **Development**: Work on the designated feature branch `claude/inspiring-planck-fiz9ts`
- **Commits**: Use clear, descriptive commit messages
- **Attribution**: All commits must include the Claude Code attribution footer:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01Uy1AMt5vMhtpBXp3uciEkb
  ```
- **Push Strategy**: Push with `-u origin <branch-name>` flag
- **Pull Requests**: Only create PRs when explicitly requested by the user

### Git Operations
- Prefer `git fetch` and `git pull` for specific branches
- Retry network failures up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
- Never force-push to shared branches
- Merge base branch changes rather than rebasing on shared work

## Code Standards

### JavaScript/HTML
- Minimize comments; only explain the "why" when non-obvious
- Use clear, descriptive variable and function names
- Avoid premature abstractions and unnecessary refactoring
- Don't add error handling for scenarios that can't happen
- Trust framework guarantees; only validate at system boundaries

### General Principles
- Prefer editing existing files to creating new ones
- Don't introduce features, refactoring, or abstractions beyond what the task requires
- No half-finished implementations
- Three similar lines of code is better than a premature abstraction
- Avoid backwards-compatibility hacks for unused code

## Security Considerations

- Never generate or guess URLs unless confident they're for helping with programming
- Be aware of command injection, XSS, SQL injection, and OWASP Top 10 vulnerabilities
- Fix security issues immediately if discovered
- Validate input only at system boundaries (user input, external APIs)

## Testing and Verification

- For UI changes, test in a browser before reporting completion
- Test both the golden path and edge cases
- Monitor for regressions in other features
- Type checking and test suites verify code correctness, not feature correctness

## Permissions and Confirmation

### Actions Requiring User Confirmation
- Destructive operations (deleting files/branches, git reset --hard)
- Force-pushing code to shared branches
- Pushing code that affects shared state
- Uploading content to third-party services

### Safe Operations (No Confirmation Needed)
- Local file editing and exploration
- Running tests and linters
- Creating commits with clear messages
- Pushing to the designated development branch

## Documentation

- Keep documentation minimal and focused
- Use GitHub-flavored markdown where applicable
- Update Claude.md if development workflow changes significantly

## Scheduled Tasks

This codebase is subject to automated scheduled tasks. When running automated tasks:
- Check git status before making changes
- Create new commits for all changes
- Push to the designated development branch
- Use clear commit messages describing what was updated and why
- Send notifications only when issues that need attention are found

## Contact

For questions about Claude Code capabilities, visit: /help

For issues and feedback: https://github.com/anthropics/claude-code/issues

---

Last Updated: 2026-10-01
