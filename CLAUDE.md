# Claude Code Agent Guidelines for Google_4

## Project Overview

Google_4 is a game development project featuring:
- **Sky Fighter**: A Raiden-style vertical shoot-em-up game
- **Repository**: https://github.com/ubban/Google_4
- **Primary Files**: index.html (main game implementation), README.md

## Development Branch

- **Active Development Branch**: `claude/inspiring-planck-3flzf5`
- Always develop on this branch and push changes here
- Do not push to main/master unless explicitly instructed

## Git Practices

### Commit Messages
- Use clear, descriptive commit messages
- Format: `[Type] Description` (e.g., `[Feature] Add enemy spawning`, `[Fix] Correct collision detection`)
- Include Claude attribution footer in all commits:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_016FMbaJfszfXxRV4PhdvrN8
  ```

### Push Strategy
- Use `git push -u origin claude/inspiring-planck-3flzf5` for first push
- Retry with exponential backoff (2s, 4s, 8s, 16s) if network fails
- Maximum 4 retry attempts

### Branch Merging
- Do not create pull requests without explicit user instruction
- When PR is created, follow template from `.github/pull_request_template.md` if it exists
- End PR descriptions with: 🤖 Generated with [Claude Code](https://claude.com/claude-code)

## Code Style Guidelines

### JavaScript/HTML
- Write clean, well-organized code
- Minimize comments - only add when WHY is non-obvious
- Use meaningful variable and function names
- Keep code DRY but avoid premature abstractions
- No half-finished implementations

### Game Development Specifics
- Maintain game loop efficiency
- Test collision detection thoroughly
- Validate rendering performance
- Ensure responsive controls

## Testing & Validation

### Before Committing
- Run the game in a browser to verify changes work
- Test the golden path and edge cases
- Monitor for regressions in existing features
- Validate rendering and performance

### Running the Game
- Open index.html in a web browser (Chromium available in environment)
- Test all modified features manually
- Screenshot or confirm visual changes work as expected

## File Structure

```
Google_4/
├── index.html          # Main game implementation
├── README.md           # Project readme
├── CLAUDE.md          # This file - Claude Code guidelines
└── .git/              # Git repository
```

## Collaboration Notes

- **User Email**: bobwu@irobotx.top (for identification only)
- **Session Management**: This is an automated scheduled task environment
- **No live user watching**: Async execution - use PushNotification for important updates
- **Attribution**: Always include Claude Code attribution in commits and PRs

## Key Principles

1. **Reversibility First**: Favor reversible, local actions. Confirm risky operations.
2. **Minimize Scope**: Implement only what's requested - no over-engineering.
3. **Test Before Pushing**: Verify changes work in the actual app before committing.
4. **Clear Communication**: Use commit messages and code to communicate intent.
5. **Respect Existing Work**: Investigate before deleting; never discard uncommitted work.

## Common Tasks

### Adding a Feature
1. Create feature on development branch
2. Test thoroughly in browser
3. Commit with clear message
4. Push to `claude/inspiring-planck-3flzf5`

### Fixing a Bug
1. Identify and reproduce the issue
2. Implement minimal fix
3. Test the fix and ensure no regressions
4. Commit and push

### Code Review
- Use `/code-review` skill for comprehensive review
- Address all findings before pushing
- Keep PR discussions focused and brief

## Resources

- Project Repository: https://github.com/ubban/Google_4
- Claude Code Documentation: https://claude.ai/code
- For help with Claude Code: `/help`
- Feedback/Issues: https://github.com/anthropics/claude-code/issues

---

**Last Updated**: 2026-09-26  
**Maintained by**: Claude Code Agent System
