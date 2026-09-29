# Claude Code Configuration for Google_4

## Project Overview

Google_4 is a web-based project with a simple structure containing an HTML interface and supporting assets.

## Development Branch

All development work should be done on the branch: `claude/inspiring-planck-2ukbv2`

## Git Workflow

### Commits
- Write clear, descriptive commit messages that explain the "why" not just the "what"
- Include the required attribution footer in all commits:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_018cqb5Kf6V6JSDvYMAjExdY
  ```

### Pushing
- Always push to the designated development branch
- Use `git push -u origin claude/inspiring-planck-2ukbv2` when pushing
- Do NOT push to the default branch without explicit authorization
- Retry failed pushes with exponential backoff (2s, 4s, 8s, 16s)

### Pull Requests
- Only create PRs when explicitly requested by the user
- Use PR templates if they exist in `.github/PULL_REQUEST_TEMPLATE.md`
- Include the attribution footer:
  ```
  🤖 Generated with [Claude Code](https://claude.com/claude-code)
  
  https://claude.ai/code/session_018cqb5Kf6V6JSDvYMAjExdY
  ```

## Code Style Guidelines

- No comments unless the WHY is non-obvious (hidden constraint, subtle invariant, workaround)
- Don't write docstrings or multi-line comment blocks
- Use well-named identifiers that are self-documenting
- Avoid backwards-compatibility hacks and unnecessary abstractions
- Delete unused code completely instead of marking it with comments

## File Changes

### Preferred Practices
- Use Edit tool for modifying existing files (sends only the diff)
- Use Write tool for creating new files or complete rewrites
- Prefer editing existing files over creating new ones
- Review changes before committing using `git diff` and `git status`

### Before Committing
- Run `git status` to see all changes
- Run `git diff` to review modifications
- Verify no sensitive files (`.env`, credentials) are included
- Check that changes align with the task requirements

## Testing and Validation

- Test UI changes in a browser before marking complete
- Run the project's test suite if available
- Verify type checking passes
- Test both golden path and edge cases for new features
- Monitor for regressions in existing features

## Project Structure

```
Google_4/
├── README.md           # Project documentation
├── index.html          # Main HTML interface
├── .git/               # Git repository metadata
└── Claude.md           # This file - Claude Code configuration
```

## Important Reminders

- This is a remote execution environment - anything worth keeping must be committed and pushed
- Don't create empty commits or use force pushes without explicit authorization
- Always confirm risky actions (deletions, force operations, pushing to shared branches)
- Use the scratchpad directory (`/tmp/claude-*`) for temporary files, not `/tmp`
- Follow the designated branch workflow - never push to different branches without permission

## Troubleshooting

If you encounter git network errors:
1. Check the proxy status: `curl -sS "$HTTPS_PROXY/__agentproxy/status"`
2. Refer to `/root/.ccr/README.md` for proxy configuration issues
3. Retry the operation with exponential backoff

## Contact & Support

For issues with Claude Code, visit: https://github.com/anthropics/claude-code/issues

---
_Updated: 2026-09-29_
