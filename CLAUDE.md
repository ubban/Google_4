# Claude Code Configuration

## Project Overview
This is the Google_4 repository - a web project with an HTML-based interface.

## Development Guidelines

### Branch Strategy
- **Main development branch**: `claude/inspiring-planck-ghu3r2`
- Always push work to the designated branch
- Create PRs against the main branch when ready to merge

### Git Practices
- Commit messages should be clear and descriptive
- Include relevant context about why changes were made
- Use `git push -u origin <branch-name>` for initial pushes
- Include attribution footer in all commits and PRs

### Code Style
- Maintain consistency with existing code patterns
- Keep changes minimal and focused on the task at hand
- Avoid unnecessary refactoring unless explicitly requested
- Default to no comments unless the WHY is non-obvious

## Claude Agent Instructions

### How to Work on This Project
1. **Read first** - Always read existing files before making changes
2. **Verify changes** - Test features in a browser when possible
3. **Commit frequently** - Mark tasks complete as soon as they're done
4. **Push regularly** - Keep the remote branch up to date

### Task Execution
- For bug fixes: Fix the bug without unnecessary cleanup
- For features: Implement only what was requested
- For refactoring: Improve code without changing behavior
- For documentation: Keep it accurate and concise

### Testing
- Run local tests before pushing
- Test UI changes in a browser if applicable
- Verify that existing features still work

## File Structure
- `index.html` - Main HTML file
- `README.md` - Project readme
- `CLAUDE.md` - This configuration file

## Permissions
- GitHub access to: `ubban/Google_4`
- Network access configured through proxy agent

## Useful Commands
```bash
# View current branch
git branch -v

# Push changes to designated branch
git push -u origin claude/inspiring-planck-ghu3r2

# View recent commits
git log --oneline -10

# Check uncommitted changes
git status
```

## PR Template
When creating PRs, include:
- Summary of changes (bullet points)
- Test plan section with testing steps
- Any relevant context or decisions made

## Questions or Issues?
- Use `/help` to get help with Claude Code
- Report issues at https://github.com/anthropics/claude-code/issues
