# Claude Code Guidelines for Google_4

## Project Overview

This is a game development project: **Sky Fighter** - a Raiden-style vertical shoot-em-up game built with HTML5, CSS, and JavaScript.

## Development Branch

All development work should be done on the designated branch:
- **Branch:** `claude/inspiring-planck-kwufkd`
- Always commit and push changes to this branch unless explicitly instructed otherwise

## Commit Message Convention

Commit messages should be clear and descriptive. Follow this format:

```
Brief description of changes (50 characters or less)

[Optional longer explanation if needed]
```

**Attribution:** All commits will automatically include:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_016rMJLyWTpQijJJH61hngzA
```

## Code Standards

### HTML/CSS/JavaScript
- Use vanilla JavaScript (no external dependencies unless necessary)
- Keep code modular and well-organized
- Use meaningful variable and function names
- Add comments only for non-obvious logic (avoid stating what the code does)

### Game Development
- Test gameplay changes in the actual browser before committing
- Verify that the game runs without console errors
- Maintain backward compatibility with existing features
- Document any new game mechanics or features

## Testing & Verification

Before pushing changes:
1. **Browser Testing:** Open `index.html` in a browser and verify all changes work correctly
2. **Visual Inspection:** Check that the game renders properly and no visual bugs exist
3. **Functionality:** Test any new or modified features (controls, game mechanics, UI)
4. **Console:** Ensure there are no JavaScript errors in the browser console

## Git Workflow

### For New Features/Fixes
1. Work on the designated branch (`claude/inspiring-planck-kwufkd`)
2. Test thoroughly in the browser
3. Commit with a clear message
4. Push to origin with `git push -u origin claude/inspiring-planck-kwufkd`

### Handling Issues
- If network failures occur during push, retry up to 4 times with exponential backoff
- For merge conflicts on the base branch, merge it in and resolve conflicts carefully
- Never force-push unless explicitly authorized

## File Organization

```
Google_4/
├── README.md          # Project overview
├── CLAUDE.md          # This file
├── index.html         # Main game file (HTML + CSS + JS)
└── .git/              # Git repository
```

## Important Rules

1. **No destructive actions without confirmation** - Before deleting files or doing force operations, ask for confirmation
2. **Prefer editing over creating** - Modify existing files when possible
3. **Keep changes focused** - One feature/fix per commit
4. **Test before pushing** - Always verify changes work in the browser
5. **Use Claude Code tools** - Prefer dedicated tools (Read, Edit, Write) over Bash for file operations

## Helpful Commands

```bash
# Check status
git status

# View recent commits
git log --oneline -5

# See changes
git diff

# Test locally
# Open index.html in your browser to test the game
```

## Notes for Claude Code Agents

- This is a web-based game project, so visual/interactive testing is essential
- Always run the game in a browser to verify changes work
- Pay attention to performance and responsiveness
- Keep the game fun and playable!
- Reference line numbers when making changes: `file_path:line_number`

---

*Last updated: 2026-09-26*
