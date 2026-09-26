# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The entire game is self-contained in a single `index.html` file with inline CSS and JavaScript. The game features a player-controlled aircraft, enemies, weapons, and stage bosses.

### Key Features
- Player ship with health system
- Multiple weapons with ammo
- Smart bombs that clear the screen
- Enemy waves with visual variety
- Stage-based boss encounters
- Score and multiplier system
- Responsive design for desktop and mobile

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + inline JavaScript)
├── README.md           # Project header
└── Claude.md          # This file
```

## Development Workflow

### Running the Game

1. Open `index.html` directly in a browser (file:// protocol works fine)
2. Use a local web server for best results: `python -m http.server 8000` or `npx http-server`
3. Test on mobile by accessing via your machine's IP address

### Game Controls

**Desktop:**
- Arrow keys or WASD to move
- Space or Click to shoot
- Shift to switch weapons
- B for bomb

**Mobile/Touch:**
- Tap/drag to move
- Tap to shoot
- Double-tap for bomb

## Making Changes with Claude

### Adding Game Features

When requesting changes to the game:
1. Be specific about what you want (mechanics, visuals, gameplay)
2. Specify where in the code to add it
3. Consider performance — everything runs on the main thread
4. Test in both desktop and mobile viewports

### Common Modifications

- **Enemy behavior:** Look for enemy spawn logic and AI patterns
- **Weapon mechanics:** Search for weapon firing and collision detection
- **Visual effects:** Check animations, particle effects, and shader-like CSS
- **UI/HUD updates:** Look for #hud element and stat display logic

## Code Guidelines

- **Keep the single-file structure** — maintain inline CSS and JavaScript for portability
- **Performance-focused:** Avoid creating objects unnecessarily in the game loop
- **Comments for complex logic:** Document enemy AI, boss patterns, and collision systems
- **Test on actual canvas size:** Game responds to viewport, test at various resolutions

## Git Workflow

This project uses a feature branch: `claude/inspiring-planck-64ols8`

All development happens on this branch. Commits should have clear messages describing the changes made.

```bash
git status                    # Check current changes
git add index.html           # Stage changes
git commit -m "Description"  # Commit with message
git push -u origin claude/inspiring-planck-64ols8  # Push to branch
```

## Tips for Claude Code Sessions

1. **Reference line numbers** when discussing code sections
2. **Test changes visually** — play the game to verify mechanics work correctly
3. **Check console for errors** — open DevTools to catch any JavaScript issues
4. **Mobile testing matters** — the game supports touch controls
5. **Save early and often** — commit meaningful changes to preserve work

## Performance Considerations

- Canvas rendering runs at 60 FPS (requestAnimationFrame)
- Avoid allocating memory in game loop (pre-allocate object pools if needed)
- Keep particle effects reasonable for mobile performance
- Profile with browser DevTools if experiencing slowdown

## Future Enhancement Ideas

- More enemy types and patterns
- Additional weapon types
- Leaderboard system
- Level customization
- Sound effects (with permission)
- Difficulty settings
- Visual themes/skins
