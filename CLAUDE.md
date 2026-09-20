# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`. The game features multiple difficulty levels, six stages with boss encounters, weapon upgrades, and is designed to work on both desktop and mobile devices.

## Quick Start

**Open the game in a browser:**
```
Open index.html directly in any modern web browser
```

The game is self-contained with no build step, server, or dependencies required.

## Architecture

### Code Organization

The entire game is a single HTML file (`index.html`) with:
- **Lines 1-421**: HTML structure and CSS styling (responsive, mobile-optimized)
- **Lines 422+**: JavaScript game engine within an IIFE (Immediately Invoked Function Expression)

### Key Game Systems

1. **Game State Management**
   - LocalStorage is used to persist: best score, mute preference, difficulty, reduced motion setting
   - Constants: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`

2. **Difficulty Configuration** (`DIFFICULTIES` object)
   - Three levels: easy, normal, hard
   - Each defines: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
   - Affects enemy spawn frequency, behavior, and player starting resources

3. **Adaptive Performance System**
   - Monitors frame times over 45 frames
   - Adjusts rendering quality (0=full, 1=medium, 2=low) to maintain stable framerate
   - Avoids frame rate drops on lower-end devices

4. **Entity Types**
   - **Enemy Types** (`ENEMY_TYPES`): grunt, interceptor, tank — each with distinct properties (size, HP, speed, score)
   - **Bosses** (`BOSSES` array): 6 boss encounters, each with unique visual themes, hull/core colors, and HP multipliers
   - **Themes** (`THEMES` array): 6 visual themes (sky gradients, cloud colors) applied per stage

5. **Game Canvas**
   - Uses 2D Canvas context for rendering sprites, enemies, bullets, effects
   - Responsive sizing with `max-width`/`max-height` to fit viewport
   - Touch-action disabled for game-specific gesture handling

6. **HUD and Overlays**
   - Real-time stats: score, best score, weapon name/level, multiplier, health hearts, bomb count
   - Boss health bar (visible during boss fights)
   - Combo notification system (`#comboPop`)
   - Stage banner announcements (`#stageBanner`)
   - Start/game-over screens (`#overlay`)

7. **Input Handling**
   - Keyboard and mouse for desktop (arrow keys or WASD for movement, spacebar to fire)
   - Touch controls for mobile with on-screen joystick and action buttons (`#touchControls`)
   - Joystick is virtual (drawn via canvas overlay) and tracks movement

8. **VFX and Polish**
   - Flash effects on hit/bomb (white flash overlays)
   - Vignette effect (red pulse during screen shake)
   - Combo pop animations (yellow text)
   - Smooth transitions and animations via CSS keyframes

9. **Accessibility**
   - Screen reader support via `#srLive` element
   - Reduced motion preference honored (via CSS `prefers-reduced-motion` media query)
   - High contrast HUD with text shadow for readability

## Common Tasks

### Adjusting Game Difficulty

Modify the `DIFFICULTIES` object (around line 485):
```javascript
easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 }
```
- `spawnBase`: Starting spawn delay (higher = fewer enemies)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability enemies fire at player (0.0–1.0)
- `lives`: Player health/lives count
- `bombs`: Starting bomb count
- `bossHpMul`: Boss HP multiplier (scales base HP)

### Adding a New Enemy Type

Add an entry to `ENEMY_TYPES` (around line 502):
```javascript
newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#abc123', dark: '#789abc', score: 20, minStage: 1 }
```
Then reference it in enemy spawning logic by its key.

### Adding a New Boss

Add an entry to `BOSSES` array (around line 508):
```javascript
{ name: 'NEWBOSS', hullTop: '#color1', hullBottom: '#color2', wing: '#color3', core: '#color4', hpMul: 1.1 }
```
Bosses spawn in order; each stage introduces the next boss when `stageCount` advances.

### Adding a New Stage Theme

Add an entry to `THEMES` array (around line 493):
```javascript
{ sky: ['#gradient1', '#gradient2', '#gradient3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```
- `sky`: Array of 3 hex colors for gradient stops
- `cloud` and `sun`: RGB values as comma-separated strings (used for particle effects)
- `name`: Display name for the stage

### Tweaking Performance

Adjust adaptive quality thresholds in `trackPerf()` function (around line 476):
- `avg > 1/40` triggers quality increase (lower FPS, more aggressive scaling)
- `avg < 1/56` triggers quality decrease (higher FPS, restore rendering)

### Changing Game Balance

Common values to adjust:
- **Score multiplier ramp**: Search for `multiplier` assignments
- **Bomb damage**: Look for bomb effect radius or damage calculations
- **Weapon upgrade scaling**: Find weapon level progression in update logic

## Styling and Responsiveness

- CSS uses `clamp()` for scalable sizing (responsive between min/max on all screen sizes)
- Dark theme with glassmorphism effects (backdrop-filter blur)
- Safe area insets respected for notched/safe-area devices (`env(safe-area-inset-*)`)
- Touch-action and user-select disabled to prevent accidental mobile browser behaviors

## Browser Compatibility

Requires modern Canvas 2D, ES6 support, and LocalStorage. Works on:
- Desktop: Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari, Chrome Android, Firefox Android

No build tools, transpilation, or polyfills needed.
