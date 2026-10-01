# Claude Code Guidelines for Google_4

## Project Overview

**Google_4** is a web-based project containing HTML and related assets. The main entry point is `index.html`.

## Development Workflow

### Branching Strategy

- **Primary Branch**: Work on designated feature branches as specified in your session
- **Branch Naming**: Follow the pattern `claude/[descriptive-name]`
- **Default Branch**: `main` or `master` (check with `git branch -r` if unsure)

### Git Workflow

1. **Always develop on your assigned branch** (currently: `claude/inspiring-planck-m2rj3z`)
2. **Before starting work**: Fetch and ensure you're on the correct branch
   ```bash
   git fetch origin
   git checkout claude/inspiring-planck-m2rj3z
   ```
3. **Commit frequently** with clear, descriptive messages
4. **Push regularly** with: `git push -u origin claude/inspiring-planck-m2rj3z`
5. **Handle merge conflicts** carefully - always merge the base branch rather than rebase others' work

### Commit Message Format

Use clear, descriptive commit messages:
```
[Type] Brief description

Optional detailed explanation of the change and why it was made.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

Commit types:
- `[feat]` - New feature
- `[fix]` - Bug fix
- `[refactor]` - Code refactoring
- `[docs]` - Documentation updates
- `[test]` - Test additions or updates
- `[style]` - Code style changes
- `[perf]` - Performance improvements

## File Structure

```
/home/user/Google_4/
├── README.md          # Project documentation
├── Claude.md          # These guidelines (you are here)
├── index.html         # Main HTML entry point
└── .git/              # Git repository
```

## Working with HTML

- **Main File**: `index.html` contains the core web content
- **No Build Step**: This project appears to be a static HTML site
- **Testing**: Open `index.html` directly in a browser to verify changes

## Common Tasks

### Viewing Changes

1. **Check git status**:
   ```bash
   git status
   ```

2. **Review changes before committing**:
   ```bash
   git diff
   ```

3. **View commit history**:
   ```bash
   git log --oneline -10
   ```

### Making Changes

1. Edit files directly in the working directory
2. Use `git add` to stage changes
3. Commit with descriptive messages
4. Push to your branch with `git push -u origin [branch-name]`

## Code Style

- Keep HTML well-formatted and readable
- Use semantic HTML5 elements where appropriate
- Maintain consistent indentation (2 or 4 spaces - check existing files)
- Add comments for complex sections
- Avoid inline styles; use external CSS if needed

## Testing & Verification

- **Visual Testing**: Open `index.html` in a browser to verify rendering
- **Link Checking**: Verify all internal links work correctly
- **Cross-browser**: If possible, test in multiple browsers (Chrome, Firefox, Safari, Edge)
- **Responsive Design**: Test on mobile, tablet, and desktop viewports if applicable

## Important Notes

- **No Destructive Actions Without Confirmation**: Do not force-push, delete branches, or use `git reset --hard` without explicit user approval
- **Preserve Working Branches**: Merge conflicts should be resolved carefully - never discard another developer's work
- **Attribution**: All commits must include proper attribution lines as shown in commit message format
- **Session Context**: Session ID for attribution: `https://claude.ai/code/session_01JSDxdd5m9awCxr5TXcwh9f`

## Git Safety Practices

When encountering issues:

1. **Network Errors on Push**: Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)
2. **Merge Conflicts**: Always investigate before modifying - use `git merge` not `git rebase` for others' work
3. **Unexpected State**: Investigate before deleting - files may represent in-progress work
4. **Stashing**: Use `git stash -u` to save work before switching branches

## Reaching Out for Help

If you need guidance on Claude Code features, workflow, or best practices:
- Use `/help` for Claude Code documentation
- Report issues at: https://github.com/anthropics/claude-code/issues
- Check available skills with `/list-skills` or type `/` to see available commands

## Quick Reference Commands

```bash
# Check current branch and status
git branch -v && git status

# Pull latest changes from your branch
git pull origin claude/inspiring-planck-m2rj3z

# Create and switch to your branch (if needed)
git checkout -b claude/inspiring-planck-m2rj3z

# Stage all changes
git add .

# Commit with message
git commit -m "[type] Description here"

# Push to your branch
git push -u origin claude/inspiring-planck-m2rj3z

# View recent commits
git log --oneline -10

# Fetch all remote changes
git fetch origin
```

---

**Last Updated**: 2026-10-01
**Project**: Google_4
**Branch**: claude/inspiring-planck-m2rj3z
