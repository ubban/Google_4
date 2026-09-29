# Claude.md - Development Guide for Google_4

This document provides guidance for Claude Code agents working on the Google_4 codebase.

## Project Overview

Google_4 is a web project containing HTML and markdown documentation. The project maintains code quality through consistent development practices and clear git workflows.

## Development Workflow

### Branches

- **Primary development branch**: `claude/inspiring-planck-sxv5qv`
- **Main branch**: `main` (protected)

All development work should be committed to the designated branch unless explicitly instructed otherwise.

### Git Practices

1. **Commit messages** should be clear and descriptive, following this format:
   ```
   [Type] Brief description of changes
   
   Optional longer explanation if needed.
   
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   Claude-Session: <session-url>
   ```

2. **Push strategy**:
   - Use `git push -u origin <branch-name>` for initial push
   - For network failures, retry up to 4 times with exponential backoff
   - Never force-push without explicit permission
   - Always ensure the working tree is clean before pushing

3. **Pull requests**:
   - Create pull requests only when explicitly requested
   - Include clear description of changes
   - Ensure CI/tests pass before pushing for review

## Code Quality Standards

### File Organization

- HTML files should be well-structured with semantic markup
- Markdown files should be properly formatted with clear headings
- Keep related files together in logical groupings

### Commit Practices

- Make atomic commits that logically group related changes
- Don't combine unrelated changes in a single commit
- Review `git status` before committing to avoid unintended files
- Avoid committing sensitive files (`.env`, `credentials`, tokens, etc.)

### Testing and Validation

- Run tests locally before pushing
- Verify linting passes
- Test HTML files in a browser to ensure rendering is correct
- Check for any broken links or missing resources

## Special Considerations

### Security

- Never commit sensitive data (API keys, credentials, tokens)
- Be cautious with file permissions and access controls
- Validate all user inputs in any interactive features

### Documentation

- Keep this Claude.md file updated as development practices evolve
- Document any non-obvious architectural decisions
- Maintain clear README.md with project setup instructions

## For Claude Code Agents

When working on this codebase:

1. **Always develop on the designated branch**: `claude/inspiring-planck-sxv5qv`
2. **Respect existing patterns**: Follow conventions already in place
3. **Keep changes focused**: Address only what's requested, avoid scope creep
4. **Verify before pushing**: Test changes locally and review diffs carefully
5. **Communicate clearly**: Use descriptive commit messages and PR descriptions
6. **Ask for confirmation**: Before taking risky actions (force pushes, deletes, etc.)

## Current Environment

- Repository: `ubban/google_4`
- Development Branch: `claude/inspiring-planck-sxv5qv`
- Platform: Linux
- Default branch: `main`

## Last Updated

2026-09-29

---

*This guide was created and should be maintained by Claude Code agents working on this project.*
