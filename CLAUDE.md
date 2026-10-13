# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and JavaScript. Players control an aircraft, switch between different weapons, drop smart bombs, and defeat stage bosses.

**Repository**: ubban/Google_4  
**Current Branch**: claude/inspiring-planck-8cvafg

## Project Structure

```
Google_4/
├── index.html          # Single-page game application (HTML + CSS + JS)
├── README.md           # Project title and timestamp
└── .git/              # Git repository
```

## Key Features

- **Player Aircraft**: Vertical movement with weapon switching capability
- **Multiple Weapons**: Different weapon types with distinct behavior
- **Smart Bombs**: Power-ups that clear enemies
- **Stage Bosses**: Boss fights with unique mechanics
- **HUD Display**: Real-time stats including health, ammo, weapon type, multiplier
- **Visual Effects**: 
  - Gradient sky background (blue to light blue)
  - Vignette effect on critical health
  - White flash on hit/bomb impact
  - Smooth animations and particle effects
- **Responsive Design**: Works on desktop and mobile devices (touch controls)

## Codebase Architecture

The entire game is contained in a single `index.html` file with embedded CSS and JavaScript.

### HTML Structure
- `#container`: Main game wrapper
- `#gameCanvas`: 2D rendering canvas (game world)
- `#hud`: Heads-up display with game stats
  - Heart display (health indicator)
  - Bomb counter
  - Weapon type indicator
  - Multiplier display
- `#vignette`: Red damage effect overlay
- `#flash`: White screen flash effect

### Key JavaScript Game Objects & Mechanics

**Player**
- Position and velocity tracking
- Health system (hearts)
- Weapon switching
- Smart bombs (limited count)
- Collision detection with enemies/bullets

**Enemies**
- Basic enemy patterns
- Boss entities with unique behavior
- Bullet spawning and movement
- Wave/stage progression

**Weapons**
- Multiple weapon types (indicated by color in HUD: lime green)
- Ammo or energy system
- Fire rate mechanics

**Game Loop**
- Input handling (keyboard/mouse/touch)
- Update game state
- Collision detection
- Render canvas
- HUD updates

### Styling Notes
- Dark theme (background: #05122b)
- Color scheme:
  - Yellow (#ffd54f): Score/stats values
  - Lime green (#7CFC00): Weapon indicator
  - Orange (#ff8a65): Multiplier
  - Blue gradient: Sky background
- Responsive sizing using CSS clamp() for mobile support

## Development Guidelines

### Before Making Changes
1. Test changes locally in a browser
2. Verify visual effects still work correctly
3. Test touch controls on mobile if modifying input
4. Check HUD display on various screen sizes

### Code Style
- Single `index.html` file structure — keep related code sections together
- Use descriptive variable names for game objects (player, enemies, bullets)
- Comment non-obvious game mechanics

### Common Tasks

**Adding Features**
- Modify CSS in `<style>` block for visual changes
- Add game mechanics in the main JavaScript section
- Update HUD indicators for new game state

**Testing**
- Open `index.html` directly in a browser
- Test on desktop and mobile devices
- Verify all weapons and bomb mechanics work
- Check boss fight progression

**Git Workflow**
- Commit changes to the feature branch: `claude/inspiring-planck-8cvafg`
- Use descriptive commit messages
- Include player-visible changes in commits

## Recent Changes

- **Latest**: "Add Sky Fighter: Raiden-style vertical shoot-em-up game" (c4dc8f6)
- **Initial**: Repository setup (4efdad1)

## Notes for Claude Code Sessions

- The game is self-contained in `index.html` — no build step required
- Use `file://` protocol or a local server to test (CORS may be needed for some features)
- Canvas rendering is performance-sensitive — profile any animation changes
- Touch event handling is configured with `touch-action: none` for optimal mobile gameplay
