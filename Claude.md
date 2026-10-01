# Claude.md - Google_4 Project Guidelines

## Project Overview

Google_4 is a web project containing HTML/CSS/JavaScript content for interactive functionality.

**Repository**: ubban/google_4  
**Primary Branch**: main  
**Development Branch**: claude/inspiring-planck-phh8b2

## Development Guidelines

### Code Style & Standards

- Use consistent indentation (2 spaces for HTML/CSS/JavaScript)
- Follow semantic HTML5 practices
- Write clean, readable code with minimal comments (only explain the "why" when non-obvious)
- Use modern JavaScript (ES6+) where applicable
- Keep CSS organized and avoid inline styles

### File Organization

- All project files should remain in the root directory
- The `index.html` is the main project file
- `README.md` contains basic project information
- No additional directories required unless necessary for scalability

## Git Workflow

### Commit Guidelines

All commits must include the Claude Code attribution footer in commit messages:

```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01X5KPfcYPaMnHf9c2CTgKZd
```

### Commit Message Format

- Use clear, descriptive messages that summarize the change
- Focus on the "why" rather than the "what"
- First line should be under 70 characters
- Use imperative mood: "Add feature" not "Added feature"

### Branch Management

- Always work on the designated development branch
- Push changes with: `git push -u origin <branch-name>`
- Never force-push without explicit permission
- Clean up temporary branches after merging

## Development Practices

### Before Pushing Code

1. Verify all changes are intentional and complete
2. Test changes locally in a browser if they affect HTML/CSS/JavaScript
3. Ensure no merge conflicts exist
4. Review the diff to catch any unintended changes

### Testing

- Test UI changes in the browser before considering work complete
- Verify both light and dark mode rendering if applicable
- Test on multiple screen sizes for responsive changes
- Validate all interactive elements work as expected

### Feature Implementation

- Implement only what is requested (no premature optimization or abstraction)
- Keep changes minimal and focused
- Do not add unnecessary error handling for impossible scenarios
- Trust framework guarantees and internal code

## File-Specific Notes

### index.html

- Main project file containing the interactive interface
- All significant changes should be tested in a browser
- Maintain consistency with existing code style
- Update README.md if major functionality changes

## Code Review Checklist

When making changes, ensure:

- [ ] Code follows the style guidelines above
- [ ] Changes are tested (browser testing for UI changes)
- [ ] No breaking changes to existing functionality
- [ ] Git commit message follows the format specified
- [ ] Changes are pushed to the correct branch

## Additional Guidelines

### When to Ask for User Input

- Complex architectural decisions
- Trade-offs that require user preference
- Ambiguous requirements
- Risk assessment for destructive operations

### Security Considerations

- Avoid command injection vulnerabilities
- Validate input at system boundaries (user input, external APIs)
- Do not add unnecessary validation for internal code paths
- Follow OWASP guidelines for web security

## Helpful Commands

```bash
# Check status
git status

# View current branch
git branch

# Switch to development branch
git checkout claude/inspiring-planck-phh8b2

# Push changes
git push -u origin claude/inspiring-planck-phh8b2

# View recent commits
git log --oneline -10
```

## Contact & Support

For questions about Claude Code features or setup:
- Visit: https://github.com/anthropics/claude-code/issues
- Use: /help command in Claude Code

---

**Last Updated**: 2026-10-01  
**Author**: Claude Code Agent  
**Session**: https://claude.ai/code/session_01X5KPfcYPaMnHf9c2CTgKZd
