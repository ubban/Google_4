# Sky Fighter - Project Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. It's a single-file web application where players control a fighter aircraft, switch between multiple weapon types, deploy smart bombs, and battle stage bosses.

## Repository Structure

- **index.html** - Main game file containing all HTML, CSS, and JavaScript code
- **README.md** - Project metadata file
- **CLAUDE.md** - This documentation file

## Project Type

- **Technology**: HTML5, CSS3, Vanilla JavaScript (ES6+)
- **Architecture**: Single-file SPA (Single Page Application)
- **Platform**: Web-based (any modern browser with Canvas support)
- **Gameplay**: Arcade-style vertical shooter

## Key Features

- **Player Aircraft Control**: Keyboard/mouse/touch controls for movement
- **Weapon System**: Multiple weapon types that can be switched during gameplay
- **Smart Bombs**: Special weapons that clear the screen
- **Boss Battles**: Stage-based progression with boss encounters
- **HUD Display**: Real-time stats and game information display
- **Responsive Design**: Adapts to different screen sizes and devices
- **Dark Theme**: Dark space-themed visual aesthetic with gradient sky background

## Core Game Components

### Canvas & Rendering
- Uses HTML5 Canvas for all game rendering
- Background gradient for sky effect (dark blue to light blue)
- Touch-friendly interface with crosshair cursor

### Player
- Controllable fighter aircraft
- Position-based health/shield system
- Multiple weapon states
- Bomb/special weapon resources

### Enemies
- Various enemy types
- Projectile attacks
- Boss encounters with unique patterns

### Weapons
- Weapon switching mechanism
- Different fire patterns and damage
- Smart bomb special weapon

### HUD (Heads-Up Display)
- Score display
- Health/shield status
- Current weapon indicator
- Bomb/special weapon counter
- Wave/stage information

## Browser Compatibility

The game requires:
- Modern browser with Canvas support
- CSS Grid/Flexbox support
- Touch and keyboard event handling
- Local environment variable support for safe-area-inset on mobile

## Development Notes

- **Single-file approach**: All code is in index.html for easy deployment
- **Game loop**: Uses requestAnimationFrame for smooth 60fps gameplay
- **Touch support**: Full mobile support with touch-action and tap events
- **Responsive**: Uses viewport meta tags and CSS clamp() for scaling
- **Dark mode**: Explicit dark color-scheme set at root level

## Mobile Optimization

- Viewport settings configured for mobile devices
- Safe area insets for notched displays (env(safe-area-inset-top))
- Touch action optimization (touch-action: none)
- Tap highlight disabled for cleaner UI
- Maximum scale locked to prevent zooming

## Future Development Considerations

- Modularize game logic into separate files if complexity increases
- Add sound effects and background music
- Implement save/high-score functionality
- Add difficulty levels or game modes
- Consider WebGL for enhanced graphics
- Local storage for player preferences

## Attribution

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01BEBfQxceHrqZinhkj9KPWM
