# Sky Fighter — CLAUDE.md

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and JavaScript. Players switch weapons, deploy smart bombs, and face off against stage bosses in fast-paced arcade action.

## Project Structure

- **index.html** — Single-file HTML5 game with embedded CSS and JavaScript
  - Game canvas rendering and UI (HUD) for score, health, bombs, weapon type
  - Responsive design with viewport adjustments for mobile (safe area insets, max-scale)
  - Vignette and screen flash effects for visual feedback

- **README.md** — Minimal project metadata

- **.git/** — Version control; main branches:
  - `main` — Production/stable release
  - `claude/inspiring-planck-4kkkjo` — Active development branch for Claude Code

## Key Game Systems

### Rendering Pipeline
- Canvas-based 2D rendering with clear, draw, and update loop
- Player sprite positioned at center-bottom of screen
- Scrolling star field background effect
- Health bars for player and enemies

### Input & Controls
- **Mouse/Touch tracking** — Crosshair cursor follows input
- **Click/Tap to fire** — Weapon systems activate on click
- **Weapon switching** — Cycle through weapon types (bullet spread, missiles, lasers, etc.)
- **Smart bombs** — Limited-use screen-clearing attacks

### Game Entities
- **Player** — Hitbox ~20px, health system, weapon/bomb inventory
- **Enemies** — Spawned in waves, patrol/charge patterns, projectile attacks
- **Bosses** — Stage-end encounters with multi-phase health bars and special attacks
- **Projectiles** — Player shots and enemy bullets with collision detection
- **Explosions** — Particle effects tied to HUD flash and vignette

### Audio/Visual Feedback
- Screen flash on hit or bomb detonation
- Red vignette pulse during critical health
- Heart UI shows player health; empty hearts fade to 0.25 opacity
- Score and multiplier displayed in HUD

## Development Guidelines

### For Claude Code Agent
When making changes:
1. **Test in browser** — Run `index.html` locally to verify game logic, rendering, and mobile responsiveness
2. **Single-file edits** — Game logic, rendering, and styles are all in `index.html`; keep it under ~60KB
3. **Canvas performance** — Avoid unnecessary redraws; batch entity updates before render
4. **Mobile first** — Preserve `touch-action: none`, `user-select: none`, and viewport meta tags
5. **Collision detection** — Check entity hitboxes before weapon/damage application
6. **Game state** — Track player health, score, multiplier, weapon type, and bomb count at top level

### Commit Message Style
Use clear, descriptive commit messages:
- `Add [feature]: Brief description` — New feature
- `Fix [system]: Brief description` — Bug fix
- `Update [area]: Brief description` — Enhancement or refactor

Example: `Fix collision: Boss health bar no longer flickers on hit`

### Branching
- Active work happens on `claude/inspiring-planck-4kkkjo`
- Push to this branch only; do not push to `main` without explicit approval
- All commits include the Claude Code attribution footer

## Technical Notes

- **Game loop** — Likely using `requestAnimationFrame` for smooth 60fps rendering
- **Multiplier system** — Increases on consecutive hits without taking damage
- **Boss patterns** — Multi-phase with escalating difficulty and attack patterns
- **Responsive canvas** — Scales to fit viewport while maintaining aspect ratio

## Quick Start for Development

1. Open `index.html` in a browser to play and test changes
2. Edit game code directly in `<script>` tag at end of HTML
3. Refresh browser to see changes (no build step required)
4. Test on mobile by opening with a mobile device or DevTools device emulation
5. Commit changes with descriptive messages and push to the development branch

---

**Last updated:** 2026-09-25  
**Maintained by:** Claude Code agent  
