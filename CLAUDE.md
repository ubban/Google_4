# Claude Code Guide for Google_4

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The game features:
- Player-controlled aircraft with multiple weapons
- Wave-based enemy spawning
- Boss encounters
- Smart bombs and power-ups
- Score tracking and difficulty progression

## Project Structure

```
Google_4/
├── index.html      # Complete game implementation (single file)
├── README.md       # Basic project info
└── CLAUDE.md       # This file
```

## Key Technical Details

### Technology Stack
- **Platform**: HTML5 Canvas
- **Language**: JavaScript (vanilla, no frameworks)
- **Rendering**: 2D Canvas API
- **Input**: Keyboard and mouse/touch controls

### Game Architecture
The entire game is self-contained in `index.html` with:
- Inline CSS styling
- Embedded JavaScript game logic
- Single canvas element for rendering
- Event-based input handling

## Important Conventions

### Code Style
- Use vanilla JavaScript (no dependencies)
- Maintain compatibility with all modern browsers
- Use CSS custom properties (`:root` variables) for theming
- Responsive design with `clamp()` for scaling

### Game Mechanics
- Canvas dimensions: Dynamic, centered in viewport
- Coordinate system: Top-left origin (0,0)
- Physics: Simple position-based movement
- Collision detection: AABB (axis-aligned bounding box)
- Sound: Implemented but can be toggled

## Common Tasks

### Running the Game
Simply open `index.html` in a web browser. No build process or server required.

### Testing
- Open in multiple browsers (Chrome, Firefox, Safari, Edge)
- Test on mobile/touch devices
- Verify fullscreen mode and responsiveness
- Test with/without sound enabled

### Making Changes
1. Edit `index.html` directly
2. Changes are live on file refresh
3. Test in browser before committing
4. Commit with clear messages describing gameplay/UI changes

## Known Patterns

### Game Loop
- Uses `requestAnimationFrame` for consistent 60 FPS rendering
- Separate update and render phases
- Event listeners for keyboard/mouse/touch input

### Asset Handling
- All graphics drawn procedurally with Canvas API
- No external image files
- Themeable colors via CSS variables

### UI Elements
- HUD (heads-up display) for stats
- Canvas-based game rendering
- Responsive to window resize and orientation changes

## Development Guidelines

### When Adding Features
- Maintain single-file architecture (no external imports)
- Preserve existing game mechanics
- Keep performance in mind (canvas rendering is GPU-accelerated)
- Test across devices and screen sizes

### When Fixing Bugs
- Identify the issue in the game loop logic
- Verify the fix doesn't break other mechanics
- Test in browser before committing
- Include issue details in commit message

### Responsive Design
- Use CSS `clamp()` for font sizes and gaps
- Test at mobile, tablet, and desktop viewports
- Ensure canvas scales properly on all devices
- Respect safe area insets for notched devices

## Attribution

Claude Code agents working on this project should include proper attribution in commits:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01J7jBhUCY6dg3ujieJawfFV
```

## Scheduled Tasks

This project uses Claude Code scheduled tasks for maintenance. When automated updates are needed, they are triggered on this branch: `claude/inspiring-planck-pnymaw`

All changes should be committed to this branch and pushed to origin.
