# Claude Code Development Guide

This document provides guidelines for Claude Code agents working with the Google_4 codebase.

## Development Branches

The primary development branch for Claude Code agents is:
- **Branch**: `claude/inspiring-planck-ob5rl2`

All changes should be developed and pushed to this branch unless explicitly instructed otherwise.

## Git Conventions

### Commit Messages

All commits should follow these conventions:

```
[Brief description of change]

[Optional longer explanation if needed]

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01GTVxFjPeZ7jDZ85GywDaSw
```

**Guidelines:**
- First line: 50-72 characters, clear and descriptive
- Use imperative mood: "add feature" not "added feature"
- Include the Co-Authored-By attribution line
- Include the Claude-Session URL for traceability

### Push Operations

- Use: `git push -u origin <branch-name>`
- For network failures: retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
- Never force-push without explicit permission
- Always verify the correct branch before pushing

## Development Workflow

### Before Starting Work
1. Verify you're on the correct branch: `git branch`
2. Fetch latest changes: `git fetch origin <branch-name>`
3. Ensure working tree is clean: `git status`

### Making Changes
1. Make focused, atomic commits for each logical change
2. Avoid unrelated cleanup or refactoring in feature commits
3. Test changes locally before committing
4. Write clear, descriptive commit messages

### After Completing Work
1. Ensure all tests pass
2. Review your own changes: `git log` and `git diff`
3. Push to the designated branch
4. Do NOT create pull requests unless explicitly requested

## Code Quality Standards

- Prefer editing existing files over creating new ones
- Avoid unnecessary comments—only document non-obvious logic
- Don't add error handling for impossible scenarios
- Trust framework guarantees and internal code safety
- For security: prevent injection attacks, XSS, SQL injection, etc.

## Project Structure

- **README.md**: Project documentation (minimal currently)
- **index.html**: Main HTML file (currently 58KB)
- **.git**: Repository management

## Important Notes

- This is a cloud session in an ephemeral container
- All work must be committed and pushed to persist
- The working directory is `/home/user/Google_4`
- Scratchpad for temporary files: `/tmp/claude-0/-home-user-Google-4/f085a713-b270-56aa-b92b-a36e26f6386d/scratchpad`

## Scheduled Tasks

This repository may use scheduled tasks to automate routine updates and maintenance. All changes made by automated tasks should follow the commit conventions listed above.

---

*Last updated: 2026-09-26*
