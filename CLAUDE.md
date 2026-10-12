# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application (index.html). The game features progressive difficulty, multiple enemy types, boss encounters, weapon upgrades, and dynamic visual/audio feedback.

## Architecture

The entire game is contained in `index.html` (~1669 lines), organized into three sections:

### HTML Structure (top)
- Canvas element for rendering
- Overlay UI for menus, pause, and game-over screens
- HUD elements: score display, weapon info, hearts/bombs, boss health bar
- Control buttons: mute, reduced-motion toggle, pause
- Touch controls for mobile (joystick + fire/bomb buttons)
- Accessibility elements (screen reader announcements)

### CSS (middle)
- Dark space-themed styling with gradient backgrounds
- Responsive design using clamp() for mobile/desktop scaling
- Animations: heart beat, combo pop, stage banners, boss bar transitions
- Glass-morphism effect (backdrop-filter) for UI cards
- Safe area insets for notched devices

### JavaScript (bottom, in IIFE)

**Core Game Systems:**
- **Update loop**: `update(dt)` handles collision detection, enemy AI, player input processing, power-up collection, scoring
- **Render loop**: `draw()` renders background, player, enemies, bullets, explosions, particles, UI overlays
- **Main loop**: `loop(now)` uses `requestAnimationFrame`, manages delta time clamping, tracks performance for adaptive quality settings

**Entity Classes/Patterns** (functions with associated data structures):
- Player: position, velocity, health, bombs, current weapon, fire rate
- Enemies: 3 types (grunt, interceptor, tank) with different HP, speed, fire rates defined in `ENEMY_TYPES`
- Bosses: 6 unique bosses in `BOSSES` array with custom renders and attack patterns
- Bullets: player and enemy bullets with velocity and collision
- Power-ups: life, bomb, weapon upgrade drops from defeated enemies
- Particles & explosions: decorative particle effects and explosion rings

**Data Structures:**
- `DIFFICULTIES`: Easy/Normal/Hard settings (spawn rates, enemy speed, lives, bombs, boss HP multiplier)
- `THEMES`: 6 sky color schemes that cycle by stage
- `ENEMY_TYPES`: Properties for each enemy variant
- `BOSSES`: Boss definitions with colors and HP scaling
- Game state: `running`, `paused`, `score`, `mult` (score multiplier), `stage`, `lives`, `bombs`, weapon level

**Key Mechanics:**
- **Adaptive quality**: Monitors frame time and downscales rendering (particle count, sprite complexity) on low-end devices
- **Weapon system**: VULCAN → LASER → GATLING, each with different fire patterns (straight, spread, rapid)
- **Scoring**: Base score × combo multiplier; chain breaks on player death or time (10 seconds)
- **Difficulty**: Affects spawn rates, enemy speed, enemy fire chance, starting lives/bombs
- **Particle effects**: Explosions, bullet trails, power-up drops
- **Sound design**: Synthesized SFX (no external audio files) for weapons, explosions, hits, bosses using Web Audio API

## Development Workflow

### Testing the Game
- **Local development**: Open `index.html` directly in a browser or use a local web server
  ```bash
  python3 -m http.server 8000
  # Then visit http://localhost:8000
  ```
- **Testing specific features**: Use browser DevTools console to set game state
  - Check `running`, `paused` to verify game state
  - Modify `score`, `mult`, `lives`, `weapons[weaponIdx].level` in console to test edge cases
  - Set `difficulty` to test difficulty modes
  - Trigger `resetGame()` to restart

### Common Edits

**Balancing/Tuning:**
- Enemy spawn rates: Edit `DIFFICULTIES` object (e.g., `spawnBase`, `spawnMin`)
- Enemy behavior: Modify `ENEMY_TYPES` (HP, speed multipliers, shoot chance)
- Boss difficulty: Adjust `hpMul` in `BOSSES` or edit boss-specific code in `spawnBoss()`
- Weapon values: `fireWeapon()` function defines fire patterns and spread angles
- Score multiplier decay: Edit the 10-second timeout in `breakChain()`

**Visual Changes:**
- Sky themes: Modify gradient colors in `THEMES` (RGB values in `sky`, `cloud`, `sun` arrays)
- Enemy colors: Edit color hex values in `ENEMY_TYPES` (e.g., `color: '#e53935'`)
- Boss colors: Edit `BOSSES` array color properties
- Animations: Edit CSS `@keyframes` or modify duration/easing in JavaScript (e.g., `cardIn`, `heartBeat`)

**Gameplay Loop:**
- Stage progression: `KILLS_PER_STAGE = 18` sets enemy kills needed to advance
- Boss encounters: Edit `spawnBoss()` when boss should appear
- Player position/bounds: `mouseTarget` handles movement; clamp values in movement code (~line 850+)

## Building & Deployment

No build step is required. The project is ready to deploy:
- Single file deployment: Just serve `index.html`
- Minification: Can minify CSS/JS inside the HTML file if size is a concern
- Testing: Works in any modern browser supporting Canvas, Web Audio API, and localStorage

## Debugging Tips

- **Browser DevTools Console**: Inspect `window` scope to check/modify game variables
- **Performance**: Check DevTools Performance tab → look for dropped frames during particle-heavy scenes (hit capacity at ~500 particles)
- **Sound issues**: Check if browser has autoplay policy restrictions (Chromium requires user gesture for audio)
- **Mobile testing**: Use Chrome DevTools device emulation; test safe-area insets on notched devices
- **Screen reader testing**: Verify `srLive` announcements fire at key events (new stage, game over, best score)

## Storage Keys

Game persists the following to localStorage:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (1 = muted)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1 = enabled)

## Accessibility

- Keyboard controls: Arrow keys / WASD for movement, Space for fire, B for bombs
- Touch controls: Joystick + on-screen buttons
- Screen reader announcements: Uses `aria-live` region for game state changes
- Reduced motion mode: Disables screen shake, flashes, and pulsing animations when enabled
- Color contrast: HUD uses light text on dark backgrounds with text-shadow for readability

## Notes on Single-File Architecture

Because everything is in one file:
- Changes are immediately testable by refreshing the browser
- Keep related functions/constants grouped logically
- Watch for global scope pollution—the entire game is in an IIFE to avoid it
- Large file size (~60KB) is acceptable for a web game; no build bundler is needed
- Comments are sparse; function names and structure should be self-documenting
