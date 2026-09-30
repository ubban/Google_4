# Claude Development Guidelines for Google_4

## Project Overview
Google_4 is a web-based shoot-em-up game project (Raiden-style vertical shmup). It uses HTML5 Canvas and vanilla JavaScript for game development.

## Repository Structure
- `index.html` - Main game file containing HTML, CSS, and JavaScript
- `README.md` - Project metadata
- `.git/` - Git repository

## Development Practices

### Code Style
- Use vanilla JavaScript (no frameworks required)
- Follow consistent naming conventions: camelCase for variables/functions, PascalCase for classes/constructors
- Keep functions focused and single-responsibility
- Aim for readable, self-documenting code with minimal comments

### Git Conventions
- Branch: `claude/inspiring-planck-4xws4q` (designated development branch)
- Commit messages: Clear, descriptive, present tense
  - Format: `<action>: <description>`
  - Examples: "Add player collision detection", "Fix bullet spawn timing", "Refactor enemy AI"
- Always push to the designated branch only

### Testing & Validation
- Test game features in a browser before declaring complete
- Verify golden path gameplay and edge cases
- Check for regressions in existing features
- Use browser console for debugging

### Game Development Standards
- Use HTML5 Canvas for rendering
- Organize code into logical game systems (player, enemies, bullets, collision, rendering)
- Maintain game loop structure with update() and draw() cycles
- Test performance with multiple entities
- Ensure responsive controls and consistent frame rates

### File Changes
- Prefer editing existing files over creating new ones
- Minimize comments; code should be self-explanatory
- No trailing comments or removed code annotations
- Clean up temporary debug code before committing

### When in Doubt
- Favor simplicity and directness
- Don't over-engineer for hypothetical future features
- Focus on the immediate task requirements
- Test feature-complete in browser before marking done

## Commit Attribution
All commits include:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_016KewvP7gq6fqmBbR2cJHZJ
```

## Contact
For questions about Claude Code or this setup, see: https://github.com/anthropics/claude-code/issues
