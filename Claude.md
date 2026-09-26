# Claude.md - Google_4 Project Guidelines

## Project Overview
Google_4 is a web-based project with HTML and documentation.

## Development Branch
- **Primary branch**: `claude/inspiring-planck-9n9e9e`
- **Default branch**: `main` (or `master`)
- Always commit changes to the designated branch
- Push changes with: `git push -u origin <branch-name>`

## Git Workflow

### Commits
- Use clear, descriptive commit messages
- Include attribution footer in all commits:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01CMbqMNQ2QN1uUh229E1dah
  ```
- One logical change per commit
- Reference issue numbers when applicable

### Push Strategy
- Always use: `git push -u origin <branch-name>`
- On network failures, retry with exponential backoff (2s, 4s, 8s, 16s)
- Never push to `main`/`master` without explicit user permission
- Never force-push unless explicitly authorized

## Code Standards

### HTML/CSS/JavaScript
- Write clean, maintainable code
- Avoid unnecessary comments - add only when WHY is non-obvious
- No premature abstractions or over-engineering
- Follow existing code patterns in the repository

### File Organization
- Keep files in their appropriate locations
- Use meaningful file names
- Document complex logic only when needed

## Testing & Quality
- Run tests before pushing changes
- Verify changes work in browser/intended environment
- Check for regressions in existing functionality
- Type checking and linting should pass

## Common Tasks

### Making Changes
1. Fetch latest code: `git fetch origin <branch-name>`
2. Create/update local branch: `git checkout -B <branch-name> origin/<branch-name>`
3. Make changes to files
4. Test changes locally
5. Commit with descriptive message
6. Push to remote branch

### Handling Merge Conflicts
1. Run: `git status` to see conflicted files
2. Resolve conflicts in editor
3. Regenerate any generated files
4. Test thoroughly
5. Commit and push

## Important Notes
- Do not delete or drastically refactor code without explicit user request
- Do not add features beyond what's requested
- Do not modify CI/CD pipelines without authorization
- Security is a priority - avoid introducing vulnerabilities
- Ask for clarification on ambiguous requests

## Contact
For questions about this project structure or guidelines, refer to user: bobwu@irobotx.top
