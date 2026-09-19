# Sky Fighter - Codebase Documentation

## Project Overview

Sky Fighter is a browser-based, Raiden-style vertical shoot-'em-up game. Players control an aircraft that can switch weapons, drop smart bombs, and face off against stage bosses. The game is built entirely with vanilla JavaScript, CSS, and HTML—no external frameworks or libraries.

**Repository**: ubban/Google_4  
**Branch**: claude/inspiring-planck-563t3q  
**Language**: JavaScript (ES6+)  
**Platform**: Web (Canvas-based)

## Codebase Structure

### Main Files

- **index.html** - Single-file application containing:
  - HTML structure with canvas element for rendering
  - Embedded CSS styling (dark theme, responsive layout)
  - Embedded JavaScript game engine and logic

- **README.md** - Minimal project metadata

## Key Game Components

### HTML Structure
- `#container` - Main viewport container
- `#gameCanvas` - Canvas element for game rendering
- `#hud` - Heads-up display showing:
  - Health (hearts)
  - Smart bombs remaining
  - Current weapon type
  - Combo/multiplier counter
- `#vignette` - Visual effect for critical health (red vignette pulsing)
- `#flash` - Flash overlay for hit/bomb effects

### Visual Design
- **Color Scheme**: Dark space-themed palette
  - Primary: #05122b (dark blue background)
  - Accent: #1976d2 (bright blue icon)
  - Gradient sky: #0d2f5e to #7fc1ff
- **UI**: Semi-transparent HUD elements with backdrop blur
- **Responsive**: Uses CSS clamp() for responsive text sizing
- **Mobile Support**: Touch-action handling, viewport-fit, safe-area-inset

### Game Mechanics (Inferred from HTML)

#### Health System
- Hearts display shows player health
- Critical state triggers heartbeat animation
- Lost hearts fade out visually

#### Weapons
- Weapon type indicator displayed in HUD
- Weapon stat in lime green (#7CFC00)

#### Smart Bombs
- Limited quantity system
- Visual bomb count in HUD
- Triggered by player action

#### Scoring/Multiplier
- Combo/multiplier counter in orange (#ff8a65)

## Development Guidelines

### Code Style
- Vanilla JavaScript (no transpiler needed)
- Single-file structure keeps all code together
- Inline CSS within `<style>` tags for self-contained game

### Responsive Design Considerations
- Mobile-first approach using CSS clamp()
- Touch actions disabled for game canvas
- Safe-area-inset for notched devices
- Max-width constraints on HUD elements

### Browser Requirements
- Modern browsers with:
  - Canvas API support
  - ES6 JavaScript support
  - CSS Flexbox and Grid
  - CSS Backdrop-filter (graceful degradation available)
  - Touch Events API

### Performance Notes
- Single canvas render context
- Use requestAnimationFrame for game loop
- Prevent default touch behaviors with touch-action: none

## Known Features

### Player Character
- Controllable aircraft
- Health system with hearts
- Crosshair cursor during gameplay

### Weapons
- Switch-able weapon system
- Weapon stats displayed in HUD

### Items
- Smart bombs with limited quantity
- Visual bomb counter

### Enemies
- Boss encounters at stage end
- Enemy projectile system

### Visual Effects
- Vignette effect for critical health
- Screen flash on hit/bomb impact
- Gradient background for parallax effect

## Future Enhancement Considerations

- Sound effects system (game ready for audio)
- Different difficulty levels
- Persistent leaderboard (client-side storage)
- Screen shake effects
- Particle systems for explosions
- Mobile performance optimization

## Session Notes

- Game is fully functional within index.html
- All logic must remain in embedded JavaScript
- CSS animations are pre-defined (heartBeat, vignettePulse, flashHit, flashBomb)
- HUD is read-only (pointer-events: none)

---

*Last updated: 2026-09-19*  
*Documented by Claude Code agent*
