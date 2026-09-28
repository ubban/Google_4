# Claude.md - Codebase Guidelines

This document provides guidance for Claude Code and other AI assistants working on the Google_4 project.

## Project Overview

**Google_4** is a project repository managed via Claude Code. This is the primary development space for the project.

**Repository**: ubban/google_4  
**Primary Development Branch**: `claude/inspiring-planck-zzsex7`

## Development Guidelines

### Branch Policy

- All development work should be committed to the designated branch: `claude/inspiring-planck-zzsex7`
- Do NOT push to the main branch without explicit authorization
- Always use `git push -u origin <branch-name>` when pushing
- If network failures occur during push, retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

### Git Workflow

1. Make changes to files in the working directory
2. Use descriptive commit messages following standard conventions
3. Include attribution in commit messages:
   ```
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   Claude-Session: <session-url>
   ```
4. Push changes to the designated branch

### Merge and Rebase Strategy

- Prefer merging base branches when needed (use `git merge` over rebase)
- Keep commit history clear and descriptive
- When a PR is merged, treat follow-up work as a fresh change on the same branch

## Code Quality Standards

- Write clear, concise code with meaningful variable and function names
- Add comments only when the WHY is non-obvious
- Avoid premature abstractions; three similar lines is acceptable
- Trust internal code and framework guarantees
- Only validate at system boundaries (user input, external APIs)

## Common Tasks

### Creating/Updating Documentation

- Documentation files (*.md) should be created/updated as needed
- Keep documentation synchronized with code changes
- Use clear section headings and descriptive content

### Testing

- Run project-specific test commands before committing
- Ensure all tests pass before pushing
- Fix test failures immediately rather than ignoring them

## Repository Structure

- Root level contains main project files and documentation
- README.md serves as the primary project documentation
- Claude.md (this file) contains AI assistant guidelines

## Important Notes

- The repository is actively managed through Claude Code
- GitHub integration is available via MCP tools when needed
- Always verify changes before pushing to ensure code quality

---

Last updated: 2026-09-28  
Maintained by: Claude Code Agent
