# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS, and JavaScript. Players control a fighter aircraft, switch between weapons, deploy smart bombs, and defeat stage bosses in an intense arcade-style experience.

### Key Features
- Vertical scrolling shoot-'em-up gameplay
- Weapon switching system
- Smart bomb mechanics
- Stage-based boss battles
- Responsive design for desktop and mobile devices
- Retro arcade aesthetics with modern visuals

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML, CSS, and JavaScript combined)
├── README.md           # Project readme
└── Claude.md          # This file — codebase documentation
```

## Architecture

The project is a **single-file web application** with all code (HTML markup, CSS styling, and JavaScript logic) contained within `index.html`. This monolithic approach is intentional for simplicity and ease of deployment.

### Main Components

#### 1. Canvas & Rendering
- Game renders to an HTML5 canvas element with ID `gameCanvas`
- Responsive sizing with max-width/max-height constraints
- Gradient background (blue sky effect)
- Dark theme with color-scheme: dark

#### 2. Game State Management
- Game loop runs at ~60 FPS using `requestAnimationFrame`
- Player position, health, weapons, and ammunition tracked
- Enemy/bullet/particle positions updated each frame
- Collision detection between player, enemies, bullets, and bosses

#### 3. Input Handling
- Desktop: Mouse movement for aiming, click to shoot
- Mobile: Touch controls with crosshair cursor
- Keyboard controls for alternative input methods

#### 4. Game Systems
- **Player Ship**: Controlled entity with multiple weapon types
- **Weapons**: Different projectile patterns and fire rates
- **Smart Bombs**: Area-of-effect damage with cool-down mechanic
- **Enemies**: Spawned in waves, patrol and attack patterns
- **Bosses**: Stage bosses with unique attack patterns and health bars
- **Particles**: Visual effects for explosions and impacts
- **HUD**: Displays health, score, weapon info, and stage progress

#### 5. Visual Design
- Dark blue color scheme (#05122b, #0b1d3a)
- Gold/cyan accent colors for UI elements
- Text shadow for readability on game canvas
- SVG favicon embedded in HTML head

## Development Guidelines

### Code Style
- Vanilla JavaScript (ES6+ features acceptable)
- Single-file structure — keep all code in `index.html`
- Use semantic HTML for markup
- Maintain consistent indentation (2-4 spaces)

### Adding Features
When adding new features:
1. Keep CSS styles within the `<style>` tag in the document head
2. Keep JavaScript within a `<script>` tag before `</body>`
3. Update the game state system to track new entities/properties
4. Test on both desktop (mouse) and mobile (touch) input
5. Ensure visual elements fit within the responsive design

### Common Tasks

#### Adding a New Enemy Type
1. Define enemy sprite/visual in CSS or canvas drawing code
2. Add enemy class/object pattern in JavaScript game state
3. Add to enemy spawning logic (usually in a wave system)
4. Implement collision/damage logic

#### Adding a New Weapon
1. Define weapon properties (fire rate, projectile speed, pattern)
2. Add to weapon selection system
3. Implement shooting logic (bullet spawning and movement)
4. Update HUD to display new weapon option

#### Creating a New Boss
1. Define boss sprite and attack patterns
2. Add stage/level that triggers boss encounter
3. Implement boss-specific collision and damage logic
4. Create victory/stage clear condition

### Performance Considerations
- Canvas-based rendering is efficient for this game type
- Use object pooling for frequently created/destroyed entities (bullets, particles)
- Minimize DOM manipulation — keep it all in canvas
- Test frame rate on target devices (mobile especially)

## Debugging Tips

### Console Logging
Open browser DevTools (F12) to check console for errors or debug output you add.

### Canvas Debugging
- Add visual debugging by drawing rectangles/circles for hitboxes
- Use `console.log()` to track entity positions and states
- Monitor frame rate and update times

### Input Testing
- Desktop: Move mouse across canvas, click to shoot
- Mobile: Touch and drag to aim, tap to shoot
- Keyboard: Test alternative key bindings if implemented

## File Formats & Technologies

- **Language**: HTML5, CSS3, JavaScript (ES6+)
- **Graphics**: Canvas 2D API
- **Deployment**: Static HTML file — no build process required
- **Browser Support**: Modern browsers with Canvas and ES6 support

## Deployment

The game requires no build process or server-side code. Simply:
1. Upload `index.html` to web server or static hosting
2. Open in modern web browser
3. Game runs entirely in browser

## Future Enhancement Ideas

- Level progression system with increasing difficulty
- Power-up items (health recovery, weapon upgrades)
- Sound effects and background music
- Save/load game progress
- Leaderboard system
- Difficulty settings (easy/normal/hard)
- Tutorial/help screen
- Multiple character skins or ships
- Online multiplayer (advanced)

## Notes for Claude Code

- This is a self-contained frontend project with no external dependencies
- Changes should maintain gameplay balance and visual coherence
- Test changes in actual browser to verify game functionality
- Keep the single-file structure unless there's compelling reason to split
- Consider mobile performance when adding visual effects

## Last Updated

2026-10-13 (Automated by Claude Code)
