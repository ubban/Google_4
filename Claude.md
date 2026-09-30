# Claude Code Guidelines for Google_4

## Project Overview

Google_4 is a web project containing HTML and related resources. The main deliverable is the `index.html` file.

## Development Branch

All development work must be done on: `claude/inspiring-planck-vmrnz6`

## Key Files

- **index.html** - Main web page (58KB)
- **README.md** - Project documentation

## Guidelines for Claude Agents

### Code Quality

1. **Avoid unnecessary abstraction** - Three similar lines is better than a premature abstraction. Don't design for hypothetical future requirements.
2. **Minimal comments** - Only add comments when the WHY is non-obvious (hidden constraints, subtle invariants, workarounds for specific bugs). Don't explain WHAT the code does - well-named identifiers already do that.
3. **No empty error handling** - Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and framework guarantees.
4. **Preferred patterns** - Use existing patterns in the codebase. Keep changes focused and minimal.

### Git Workflow

1. **Commit messages** - Use clear, descriptive messages. Include:
   - Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   - Claude-Session: https://claude.ai/code/session_01DjmVbEpqUcsQzRuadXVWG8

2. **Feature development** - Create commits for each logical change. Don't batch unrelated work.
3. **Push to branch** - Always use `git push -u origin claude/inspiring-planck-vmrnz6`
4. **No force pushes** - Unless explicitly authorized by the user.

### Testing and Verification

- For UI changes, test in a browser before marking as complete
- Verify the golden path and check for regressions
- Type checking and tests verify code correctness, not feature correctness

### File Operations

- Use Edit tool for existing file modifications (sends only diffs)
- Use Write tool for new files or complete rewrites
- Delete unused code completely - avoid backwards-compatibility hacks

### Git Safety

- Run `git status` before destructive operations
- Review changes with `git diff` before committing
- Follow the repository's commit message style conventions
- Never commit files containing secrets (.env, credentials.json, etc.)

## Useful Commands

```bash
# Check status
git status

# View changes
git diff

# Create and push commits
git add <files>
git commit -m "message"
git push -u origin claude/inspiring-planck-vmrnz6

# Fetch latest changes
git fetch origin claude/inspiring-planck-vmrnz6
```

## Common Tasks

### Adding/Modifying Files

1. Read the existing file (if it exists)
2. Make targeted changes using Edit tool
3. Test changes if they affect UI
4. Commit with clear message including session/author info
5. Push to designated branch

### Code Review

When reviewing changes, check for:
- Correctness of logic
- Adherence to existing patterns
- Minimal, focused changes
- No premature abstractions
- Clear commit messages

## Questions or Issues?

- For questions about Claude Code features, use the claude-code-guide agent
- Report issues at https://github.com/anthropics/claude-code/issues
- Check project documentation in README.md

---

Last updated: 2026-09-30
