# CLAUDE.md: Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented in HTML5 Canvas with JavaScript. The game features:
- Vertical scrolling gameplay
- Player aircraft with multiple weapon types
- Enemy formations with stage progression
- Collectible power-ups and bombs
- Boss encounters
- Responsive design for mobile and desktop

## Repository Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript combined)
├── README.md           # Project README
└── CLAUDE.md           # This file
```

## Development Guidelines for Claude Code Agents

### Branch Policy
- **Main branch**: `main` - Contains stable, released versions
- **Feature branches**: Use the assigned feature branch (e.g., `claude/inspiring-planck-fxc5lb`) for development
- All changes must be committed and pushed to the assigned feature branch
- Never push directly to `main` without explicit permission

### Commit Conventions
- Write clear, descriptive commit messages in imperative mood
- Include meaningful context about the change
- Example: `Add pause menu functionality to gameplay` or `Fix collision detection for power-ups`

### Code Style
- The main game code is in `index.html` as a single integrated file
- Maintain consistent indentation (2-space indentation used throughout)
- Keep related functionality together within the JavaScript section
- Add comments for complex game mechanics and algorithms

### Game Architecture

The game code in `index.html` follows this structure:

1. **HTML**: Canvas element and HUD containers
2. **CSS**: 
   - Game canvas styling
   - HUD layout and animations
   - Visual effects (vignette, flash effects)
   - Responsive design for different screen sizes
3. **JavaScript**:
   - Game state management (player, enemies, projectiles, etc.)
   - Collision detection system
   - Stage and boss progression
   - Input handling (keyboard, mouse, touch)
   - Rendering pipeline

### Key Game Systems

- **Player System**: Position, health, weapons, bombs
- **Enemy System**: Formations, bullet patterns, health
- **Collision System**: Hit detection between player/enemies/projectiles
- **Stage System**: Progressive enemy waves and boss encounters
- **HUD System**: Score, health, bombs, weapon display

### Testing
Before committing changes:
1. Test gameplay on both desktop and mobile devices
2. Verify collision detection accuracy
3. Check responsive design across different screen sizes
4. Ensure visual effects render smoothly
5. Test all weapon types and power-ups
6. Verify boss encounters work correctly

### Making Changes
1. Keep changes focused and incremental
2. Test thoroughly before committing
3. Write clear commit messages
4. Push to the assigned feature branch
5. For significant features, test gameplay flow end-to-end

### Performance Considerations
- The game runs on a fixed canvas size with smooth 60 FPS target
- Keep draw calls efficient
- Cache DOM references where possible
- Minimize memory allocations in game loop

### Responsive Design Notes
- Canvas is centered and maintains aspect ratio
- HUD uses viewport units for scaling
- Touch controls are optimized for mobile play
- Safe area insets are respected for notched devices

## Important Notes

- The entire game is self-contained in `index.html` for easy deployment
- Do not split into multiple files without project discussion
- Game state is managed in-memory during play
- No persistent save system currently implemented
- No external dependencies - pure HTML5/CSS3/JavaScript

## Contact & Attribution
When working on this project as a Claude Code agent, ensure all commits include proper attribution in the commit message footer.
