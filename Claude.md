# Claude.md — Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game runs entirely in a single `index.html` file (~1669 lines) without external dependencies.

**Repository**: ubban/Google_4
**Current Branch**: claude/inspiring-planck-w6yvb7
**Last Updated**: 2026-02-23

## Project Structure

```
.
├── index.html          # Single-file game implementation (~1669 lines)
├── README.md           # Project metadata
└── Claude.md           # This file
```

## Technologies

- **HTML5 Canvas** — All rendering
- **Vanilla JavaScript** — No frameworks or build tools
- **Responsive Design** — Mobile-friendly with viewport meta tags
- **Touch Support** — Touch-action optimized for mobile gameplay
- **Dark Theme** — CSS color-scheme: dark

## Game Features

- **Weapon Switching** — Multiple weapon types available during gameplay
- **Smart Bombs** — Screen-clearing weapons with limited uses
- **Boss Battles** — Stage-based progression with boss encounters
- **Mobile/Desktop Support** — Crosshair cursor on desktop, touch controls on mobile
- **Performance Optimized** — Fixed canvas rendering with efficient game loops

## Code Organization (in index.html)

The file is organized into distinct sections:
1. **HTML Structure** — Single container + canvas + HUD
2. **Styles (CSS)** — Game styling, responsive layout, animations
3. **JavaScript (Game Logic)** — Game engine, controls, rendering

### Key Game Logic Areas

- **Game State Management** — Player health, score, weapon state, bomb count
- **Entity Management** — Player, enemies, projectiles, boss
- **Collision Detection** — Bullet-enemy, bomb-enemy, enemy-player
- **Rendering Pipeline** — Background, enemies, projectiles, UI
- **Input Handling** — Keyboard (desktop), mouse/touch (mobile)

## Development Guidelines

### When Making Changes

1. **Test in Browser** — Always run the game to verify changes work
2. **Mobile Testing** — Test on mobile viewport (max-width: 100vw)
3. **Performance** — Avoid blocking operations in the game loop
4. **Collision Logic** — Game physics are critical; test thoroughly
5. **Edge Cases** — Handle screen boundaries, missing entities, rapid input

### Code Style

- **Variable Naming** — Use descriptive names (e.g., `playerHealth`, `bulletSpeed`)
- **Comments** — Add comments for non-obvious game logic (collision formulas, state transitions)
- **Functions** — Keep game loop functions pure when possible
- **Constants** — Define magic numbers as named constants at the top of the script

### Game Loop Architecture

The game uses a standard requestAnimationFrame loop:
```
update() → render() → requestAnimationFrame(gameLoop)
```

Changes to timing, physics, or rendering must maintain this structure.

## Testing & Running

1. **Open in Browser** — Simply open `index.html` in a modern browser
2. **Mobile** — Test on actual mobile device or browser devtools device emulation
3. **Debug** — Use browser DevTools console for any errors
4. **Performance** — Monitor FPS in-game; target 60 FPS on desktop, 30+ FPS on mobile

## Important Notes for Claude Agents

- **Single File Codebase** — All changes must stay in `index.html`; do not split into separate files
- **No Build Process** — No compilation, bundling, or transpilation
- **Backward Compatibility** — Ensure new features don't break existing gameplay
- **Mobile Priority** — Maintain touch and responsive design as equal to desktop
- **Browser Compatibility** — Target modern browsers (Chrome, Firefox, Safari, Edge)

## Accessibility & UX

- **Crosshair Cursor** — Provides clear aiming feedback
- **Safe Area Insets** — Respects notches on mobile devices (env(safe-area-inset-*))
- **Touch Optimization** — No hover states; all interactions keyboard/touch compatible
- **Color Contrast** — Text shadows ensure readability on gradient backgrounds

## Common Tasks

### Adding a New Weapon Type
1. Define weapon properties (damage, fire rate, projectile sprite)
2. Modify weapon switching logic in input handler
3. Add rendering code for new projectile
4. Update HUD to display weapon type

### Balancing Gameplay
- Adjust `playerSpeed`, `enemySpeed`, damage values, spawn rates
- Test in-browser for feel and difficulty progression
- Consider mobile device performance impact

### Fixing Bugs
- Reproduce in browser first
- Add console.log statements if needed
- Isolate the problematic system (collision, rendering, input)
- Test edge cases after fix

## Git Workflow

- **Commits** — One logical change per commit
- **Branch** — All work on `claude/inspiring-planck-w6yvb7`
- **Messages** — Describe the "why" and "what" of changes
- **Testing** — Run the game before committing

---

**Last Updated**: 2026-09-28
**Maintained By**: Claude Code Agent
