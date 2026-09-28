# Claude.md - Project Guidelines

## Project Overview
This is the Google_4 project, a static HTML-based application. The project consists primarily of an `index.html` file that serves as the main entry point.

## Development Practices

### Code Style
- Write clean, semantic HTML5
- Use meaningful class and ID names that describe purpose, not appearance
- Prefer semantic HTML elements (`<section>`, `<article>`, `<nav>`, etc.) over generic `<div>` tags
- Maintain consistent indentation (2 spaces for HTML)

### JavaScript Guidelines
- Write modular, reusable code
- Use modern ES6+ syntax where supported
- Minimize inline scripts; prefer external script files when practical
- Add comments only for non-obvious logic
- Avoid global variables; use modules or IIFE patterns

### CSS Guidelines
- Keep CSS organized and maintainable
- Use class-based styling over inline styles
- Follow a consistent color scheme and typography
- Ensure responsive design with appropriate media queries
- Prefer CSS variables for colors, spacing, and other reusable values

### Testing & Validation
- Test HTML markup validity with HTML validators
- Verify responsive behavior across common breakpoints
- Test cross-browser compatibility for modern browsers
- Check for accessibility compliance (WCAG 2.1 AA standard)

## Git Workflow

### Branching Strategy
- Development work happens on feature branches
- Branch naming: `feature/description` or `fix/description`
- The designated branch for this session is `claude/inspiring-planck-4kpmlj`

### Commits
- Write clear, descriptive commit messages
- Use present tense ("Add feature" not "Added feature")
- Include why the change was made, not just what changed
- Commits must include proper attribution footer

### Pull Requests
- Include clear title and description
- Reference related issues where applicable
- Ensure all tests pass before requesting review
- Follow the PR template if one exists

## File Organization
```
Google_4/
├── index.html      # Main application file
├── README.md       # Project documentation
└── Claude.md       # This file - Claude development guidelines
```

## Project-Specific Notes
- This is a static HTML project; no build process is required
- Changes should be committed directly to the working branch
- Test changes by opening `index.html` in a web browser
- Ensure all changes are backward compatible

## Attribution
When creating commits or PRs, include:
- Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
- Claude-Session reference where applicable

---
*Last updated: 2026-09-28*
