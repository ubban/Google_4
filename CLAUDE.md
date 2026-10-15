# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The game features classic shmup gameplay with weapon switching, smart bombs, and stage bosses.

- **Type**: Web-based browser game
- **Tech Stack**: Vanilla JavaScript, HTML5 Canvas, CSS3
- **Primary File**: `index.html` (single-page app with embedded game code)
- **Status**: Active development

## Repository Structure

```
Google_4/
├── README.md           # Project timestamp and title
├── index.html          # Complete game implementation
└── CLAUDE.md           # This file
```

## Key Files

### index.html (Primary Game Implementation)
The entire game is contained in a single `index.html` file with:
- **HTML Structure**: Canvas element, HUD container, and modal dialogs
- **CSS**: Styling for UI, HUD, modals, and responsive design (mobile-friendly with viewport and safe-area support)
- **JavaScript**: Complete game engine including:
  - Game loop and state management
  - Player ship controls and mechanics
  - Weapon system with multiple weapon types
  - Enemy AI and spawning system
  - Collision detection and particle effects
  - HUD display and game UI
  - Touch and mouse input handling
  - Stage/level system with bosses
  - Sound effects (using Web Audio API)

## Game Architecture

### Core Systems

1. **Input System**
   - Mouse control (position-based aiming)
   - Touch/mobile support
   - Keyboard fallback

2. **Rendering**
   - Canvas-based 2D graphics
   - Gradient backgrounds
   - Sprite rendering (all graphics generated programmatically)
   - Particle effects for explosions and impacts

3. **Gameplay Systems**
   - Player ship with health and movement
   - Weapon switching (primary and secondary weapons)
   - Smart bomb mechanic
   - Enemy spawning waves
   - Stage/boss system
   - Score and combo tracking

4. **Audio**
   - Web Audio API for sound effects
   - Procedurally generated sounds

## Development Workflow

### Running the Game
Simply open `index.html` in a web browser. No build process or dependencies required.

### Testing
- Test on desktop (Chrome, Firefox, Safari)
- Test on mobile (iOS Safari, Chrome Mobile) with touch controls
- Test responsiveness across different viewport sizes
- Test boss mechanics and weapon switching

### Making Changes
1. Edit the JavaScript game code in `index.html`
2. Test locally in the browser
3. Verify across different devices/browsers
4. Commit changes with clear descriptions
5. Push to the feature branch

## Code Style & Conventions

- **Formatting**: Standard JavaScript conventions
- **Comments**: Add comments for complex game logic and game tuning parameters (difficulty, speed, damage values, etc.)
- **Performance**: Keep rendering efficient; avoid unnecessary redraws
- **Mobile Optimization**: Test and optimize for touch controls and smaller screens

## Common Game Parameters to Adjust

Look for these configurable values in the JavaScript for tuning game difficulty/feel:

- Enemy spawn rates and waves
- Player ship speed and health
- Weapon damage and fire rates
- Boss health and attack patterns
- Particle counts for effects
- Background scroll speed

## Git Workflow

- **Branch**: `claude/inspiring-planck-t0k0yo` (all development on this branch)
- **Commits**: Clear, descriptive commit messages focusing on what changed and why
- **Attribution**: Include co-author line in commits:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: [session-url]
  ```

## Known Limitations

- Single file monolithic structure (all code in index.html)
- No persistent data (high scores reset on page reload)
- No sound on first interaction (browser Web Audio API requires user gesture)

## Future Enhancement Ideas

- Local storage for high score persistence
- Multiple difficulty levels
- Additional weapon types
- Custom stage editor
- Leaderboard system
- Mobile app wrapper (Cordova/Electron)

## Resources

- [Canvas API Docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Audio API Docs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
