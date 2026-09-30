# Claude Code Guidelines for Google_4

This file documents guidelines and conventions for Claude Code when working on the Google_4 codebase.

## Project Overview

This is a web-based project containing HTML and static content. The main entry point is `index.html`.

## Development Branch

All development should be done on the designated feature branch: `claude/inspiring-planck-3s07aj`

## Git Workflow

### Commits
- Write clear, descriptive commit messages
- Include relevant context about the changes
- End commit messages with the attribution footer provided by Claude Code

### Push Process
- Always push to the designated feature branch
- Use `git push -u origin <branch-name>` format
- For network issues, retry with exponential backoff (2s, 4s, 8s, 16s)

### Pull Requests
- Create PRs only when explicitly requested
- Check for PR templates in `.github/` directory
- Follow repository-specific conventions

## Code Style and Quality

### General Principles
- Write clear, concise code
- Prefer simplicity over complex abstractions
- Add comments only when the "WHY" is non-obvious
- Use meaningful variable and function names

### HTML/CSS/JavaScript
- Follow semantic HTML5 practices
- Keep CSS organized and maintainable
- Use vanilla JavaScript unless external libraries are justified

## Testing

- Test changes manually when working on UI/frontend
- Run any available test suites before committing
- Verify golden path and edge cases for new features

## Repository Structure

```
/
├── README.md       - Project documentation
├── index.html      - Main entry point
└── CLAUDE.md       - This file (Claude Code guidelines)
```

## Important Notes

- Keep the working tree clean before starting new work
- Always verify git status before major operations
- Report issues through the standard GitHub workflow
- Use the designated branch for all feature work

## Contact and Feedback

For questions about Claude Code or this workflow, visit:
- Claude Code Help: /help
- GitHub Issues: https://github.com/anthropics/claude-code/issues
