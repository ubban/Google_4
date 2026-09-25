# Google_4 Codebase Guide

## Project Overview

Google_4 is a web-based project with a static HTML frontend (`index.html`) and minimal configuration. This is a simple project that requires documentation for consistent collaboration.

## Repository Structure

```
Google_4/
├── README.md          # Basic project readme
├── index.html         # Main HTML file (static web page)
└── CLAUDE.md          # This file - codebase documentation
```

## Tech Stack

- **Frontend**: HTML5 (static)
- **Version Control**: Git

## Getting Started

### Prerequisites
- A modern web browser (Chrome, Firefox, Safari, Edge)
- Git for version control

### Running the Project

1. Open `index.html` directly in a web browser, or
2. Serve the project with a local HTTP server:
   ```bash
   # Using Python 3
   python -m http.server 8000
   
   # Using Node.js (if http-server is installed)
   npx http-server
   ```
3. Navigate to `http://localhost:8000` (or appropriate port)

## Development Workflow

### Branch Strategy
- Development branch: `claude/inspiring-planck-2ibisb`
- All changes should be committed and pushed to this branch
- Use descriptive commit messages following the format:
  - Feature: "Add [feature description]"
  - Fix: "Fix [issue description]"
  - Docs: "Update [documentation area]"

### File Structure
- Keep HTML in `index.html`
- If expanding the project, consider organizing into:
  - `css/` directory for stylesheets
  - `js/` directory for JavaScript files
  - `assets/` directory for images and resources

## Development Guidelines

### HTML Development
- Use semantic HTML5 elements
- Maintain clean, readable code structure
- Test in multiple browsers before committing

### Git Practices
- Write clear, descriptive commit messages
- Keep commits focused and atomic
- Include `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>` in commit messages
- Push to `claude/inspiring-planck-2ibisb` branch

### Code Style
- Indent HTML with 2 spaces
- Use meaningful class and ID names
- Keep inline styles minimal (prefer external CSS if needed)

## Project Goals & Considerations

- Maintain simplicity and avoid unnecessary complexity
- Document any new features or significant changes
- Test changes in a browser before committing
- Keep the project accessible and performant

## Common Tasks

### Adding New Content
1. Edit `index.html`
2. Test in browser
3. Commit changes with descriptive message
4. Push to development branch

### Future Expansion
If the project expands:
- Consider adding CSS preprocessing (Sass/LESS)
- Implement bundling (Webpack/Vite) if JavaScript grows
- Add automated testing if applicable
- Consider a build process if needed

## Version History

- **Latest Update**: 2026-09-25
- Created CLAUDE.md for documentation purposes
- Repository started: 2026-02-23

## Questions or Issues

When working on this project:
- Refer to this CLAUDE.md for context
- Keep documentation updated as the project evolves
- Use descriptive branch names and commit messages
