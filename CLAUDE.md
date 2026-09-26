# Claude.md - Sky Fighter Development Guide

This document provides guidance for Claude Code agents working on the Sky Fighter project.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. The game features:

- Vertical scrolling gameplay mechanics
- Player aircraft control and weapon systems
- Enemy waves and boss encounters
- Dynamic scoring and progression
- Mobile and desktop support with responsive canvas sizing
- Touch and keyboard input handling

## Repository Structure

```
Google_4/
├── README.md          # Project metadata
├── index.html         # Single-page application with game engine
└── .git/              # Git repository
```

## Key Architecture Notes

### Single-File Design
The entire game is contained in `index.html` with embedded CSS and JavaScript. This is intentional for:
- Easy deployment (single file)
- No build tooling required
- Simple hosting and sharing

### Canvas-Based Rendering
- Game uses HTML5 Canvas API for all rendering
- Responsive sizing with `max-width: 100%` and `max-height: 100%`
- Safe area insets for mobile notches via CSS `env(safe-area-inset-*)`

### Input Handling
- Keyboard controls via `keydown`/`keyup` events
- Touch/pointer events with `touch-action: none` for precise control
- Cursor set to `crosshair` for targeting feedback

### Game Loop
- Runs on `requestAnimationFrame` for smooth 60 FPS gameplay
- Handles collision detection, enemy AI, particle effects
- Updates HUD with score, lives, and weapon status

## Development Guidelines for Claude Code Agents

### Before Making Changes

1. **Understand the Current State**
   - Read through the entire `index.html` file to understand game mechanics
   - Note the game loop structure and state management approach
   - Check the existing JavaScript classes/objects for pattern consistency

2. **Test Locally**
   - Open `index.html` in a browser to verify the game runs
   - Test gameplay on desktop (keyboard) and mobile (touch)
   - Check canvas rendering at different screen sizes

### Code Style

- **Vanilla JavaScript**: No frameworks or build tools
- **Variable Naming**: Use camelCase for variables and functions
- **Constants**: Use UPPER_SNAKE_CASE for magic numbers and configuration
- **Comments**: Add comments only for non-obvious logic; game mechanics should be self-explanatory through code
- **Indentation**: Use 2 spaces for consistency with existing code

### Making Changes

1. **Preserve Functionality**
   - Test that changes don't break existing gameplay
   - Ensure mobile responsiveness is maintained
   - Verify touch and keyboard inputs still work

2. **Performance Considerations**
   - Canvas rendering is GPU-accelerated; avoid excessive DOM manipulation
   - Keep game loop tight; move complex calculations outside if possible
   - Profile with browser DevTools if performance degrades

3. **Mobile First**
   - Test on small screens (e.g., 320px width)
   - Ensure touch targets are appropriately sized
   - Verify safe area insets display correctly

### Adding Features

- **Weapons/Enemies**: Extend existing class structures
- **Visual Effects**: Use canvas particles, keep frame rate stable
- **Audio**: Consider file size; may want to use Web Audio API
- **UI Changes**: Maintain dark theme consistency and accessibility

## Git Workflow

### Branching

Development branch: `claude/inspiring-planck-i1cej8`

- All feature work happens on the development branch
- Commits should be atomic and well-described
- Include attribution footer in commit messages (see below)

### Commit Messages

Format: `<type>: <description>`

Examples:
- `feat: add boss-level mechanics`
- `fix: correct weapon fire rate calculation`
- `refactor: optimize collision detection`
- `docs: update CLAUDE.md with new patterns`

Include the standard attribution footer:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01E2RHDdxE64iUwocMfF2a5y
```

### Push and Pull Requests

1. Make changes and commit on the development branch
2. Push with: `git push -u origin claude/inspiring-planck-i1cej8`
3. Create a PR when feature is complete and tested
4. Include gameplay notes in PR description

## Testing Checklist

Before marking work complete:

- [ ] Game starts without errors in browser console
- [ ] Player can move and shoot
- [ ] Keyboard controls work (arrow keys, spacebar, etc.)
- [ ] Touch/mobile controls work (if applicable)
- [ ] Canvas renders correctly at 100% viewport width/height
- [ ] HUD displays correctly (score, lives, weapon status)
- [ ] Enemy waves spawn and behave as expected
- [ ] Collision detection works properly
- [ ] Frame rate remains smooth (check with DevTools Performance tab)
- [ ] No memory leaks (check heap snapshots for repeated gameplay)

## Common Patterns

### Accessing Game State
Game state is typically managed through global variables or a game object. Check the existing structure before adding new state.

### Canvas Rendering
```javascript
ctx.clearRect(0, 0, canvas.width, canvas.height);
// Render game objects
// Update HUD
```

### Event Handling
- Input events are captured at the document/window level
- Coordinate systems must account for canvas offset and scaling
- Mobile events use `clientX`/`clientY` normalized by canvas position

## Known Limitations and Considerations

- Single HTML file limits modularity; consider this when adding large features
- All game data is in memory; no persistence layer exists
- Mobile performance depends on device capabilities
- Touch precision varies across devices

## Questions or Issues?

When working on this project:
1. Read existing code before implementing similar functionality
2. Test thoroughly on both desktop and mobile
3. Commit frequently with clear messages
4. Keep the game fun and playable

---

Last updated: 2026-09-26
