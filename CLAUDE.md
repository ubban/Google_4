# Sky Fighter — Claude Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5/JavaScript and Canvas. Players control a fighter jet to destroy enemies, switch weapons, drop smart bombs, and defeat stage bosses.

- **Technology**: Vanilla HTML5, CSS3, JavaScript
- **Deployment**: Single `index.html` file with embedded styles and game logic
- **Gameplay**: Top-down vertical shooter with progressive difficulty and boss fights

## Architecture

### Single-File Structure

The project is intentionally kept as a single `index.html` file for portability and ease of deployment. The file contains:

- **HTML Structure**: Minimal DOM (game canvas, HUD overlay)
- **CSS Styling**: Dark space-themed UI with responsive viewport scaling
- **Game Logic**: Complete game engine in embedded `<script>` tag

### Key Game Systems

- **Player**: Fighter jet controlled via mouse/touch or keyboard
- **Enemies**: Various enemy types with different behaviors
- **Weapons**: Primary and secondary weapons with firing modes
- **Smart Bombs**: Screen-clearing power-up mechanic
- **Boss Encounters**: Stage-ending boss battles
- **Scoring & Progression**: Level/stage progression with increasing difficulty

## Development Guidelines

### Making Changes

1. **Edit index.html**: All game code is in one file. Use a text editor or IDE with HTML/JavaScript support.
2. **Test Locally**: Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge). The game runs entirely client-side.
3. **No Build Step**: The project requires no compilation, bundling, or npm dependencies.

### Code Style

- Use clear, descriptive variable and function names.
- Keep game logic organized into logical sections (player, enemies, rendering, etc.).
- Avoid breaking the single-file structure unless there's a compelling reason.
- Comment complex game logic or non-obvious mechanics.

### Testing & Validation

- **Manual Testing**: Open the HTML file in a browser and play through levels.
- **Cross-Browser**: Test on different browsers to ensure Canvas rendering and input handling work correctly.
- **Mobile**: Test touch controls on mobile devices; the game is responsive and touch-enabled.
- **Performance**: Monitor frame rate during heavy enemy/particle activity; aim for smooth 60 FPS gameplay.

### Gameplay Features to Preserve

When making changes, ensure these core mechanics remain functional:

- **Player Movement**: Mouse/touch control and keyboard directional input
- **Weapon Switching**: Ability to change between weapon types
- **Smart Bombs**: Explosion effect that clears the screen
- **Boss Encounters**: Scripted boss battles at stage transitions
- **Score Tracking**: Persistent score and stage progression
- **Audio Feedback**: Visual effects and UI responsiveness (if sounds are added in future)

## Common Tasks

### Adding Enemy Types

Extend the enemy spawning logic to create new enemy patterns, behaviors, and sprites. Update enemy collision and damage logic accordingly.

### Modifying Difficulty

Adjust enemy spawn rates, health pools, bullet patterns, and stage progression in the game loop. Increase difficulty gradually with each stage.

### Tweaking Weapon Balance

Modify weapon damage, fire rate, reload time, and visual effects to balance gameplay. Test impact on enemy clear rates and difficulty.

### Optimizing Performance

Profile rendering and update loops. Consider:
- Culling off-screen entities
- Pooling bullets and particles
- Reducing particle counts on lower-end devices
- Using requestAnimationFrame efficiently

## Troubleshooting

- **Game Not Starting**: Check browser console for JavaScript errors. Ensure the HTML file is not corrupted.
- **Controls Unresponsive**: Verify event listeners for mouse/touch/keyboard input are attached to the game canvas.
- **Low Frame Rate**: Profile with browser DevTools. Check for expensive DOM operations or inefficient render loops.
- **Touch Not Working**: Confirm touch event handlers are enabled and `touch-action` CSS is correct.

## File Structure

```
Google_4/
├── README.md          # Project description
├── CLAUDE.md          # This file (Claude development guide)
└── index.html         # Complete game (HTML + CSS + JavaScript)
```

## Git Workflow

- **Branch**: Work on `claude/inspiring-planck-fcuocn`
- **Commits**: Use clear, descriptive commit messages that explain the why and what of changes
- **Push**: Push changes with `git push -u origin claude/inspiring-planck-fcuocn`

---

*This guide is maintained to help Claude and other AI assistants understand the project structure, development practices, and gameplay mechanics. Update it when adding major features or changing architecture.*
