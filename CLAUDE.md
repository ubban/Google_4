# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/JavaScript application. Players fly a fighter jet through stages, collect weapon power-ups, use bombs to clear enemies, and face off against boss ships.

## Architecture & Key Concepts

### Single-File Structure
The entire game is implemented in `index.html` with three integrated sections:
- **HTML/CSS** (lines 2–359): UI layout, HUD display, overlay screens, and responsive styling
- **Canvas Rendering** (lines 520–1638): 2D graphics drawing using `canvas.getContext('2d')`
- **JavaScript Game Logic** (lines 422–1666): Core game mechanics, input, physics, collision detection

### Core Game Loop
The game runs on `requestAnimationFrame` (line 1665) firing `loop()` which:
1. Calculates delta time from the last frame
2. Calls `update(dt)` if game is running/unpaused (line 1648)
3. Calls `draw()` to render the current frame state (line 1650)

The `update()` function (starting at line 1148) handles:
- Player movement and collision bounds checking
- Weapon firing with cooldown
- Enemy spawning, positioning, and bullet patterns
- Enemy-to-player bullet collision detection
- Power-up collection
- Chain multiplier decay for scoring

### Key Game Objects
- **player**: Position, weapon type/level, cooldown, invulnerability timer, dimensions
- **enemies**: Array of enemy ships with type-specific stats (hp, speed, attack pattern, color)
- **boss**: Single boss object when active, with attack patterns indexed into `BOSS_PATTERNS` array
- **bullets/enemyBullets**: Player and enemy projectiles with velocity
- **powerUps**: Collectible items (life, bomb, weapon upgrades)
- **particles**: Explosion effects with position, velocity, lifetime

### Difficulty & Progression
The `DIFFICULTIES` object (line 485) defines spawn rates, enemy behavior, and lives for three modes: easy, normal, hard. Each stage increments when the player defeats a boss after killing `KILLS_PER_STAGE` (18) enemies. Bosses are selected round-robin from the `BOSSES` array and have stage-appropriate difficulty scaling.

### Audio System
Synthesized sound effects via WebAudio API (`beep()`, `noiseBurst()`, `sfx.*` objects). No external audio files. Audio context initialized on first user interaction to comply with autoplay policies. Master gain routed through a dynamics compressor for punch.

### Input Handling
- **Keyboard**: Arrow keys, WASD for movement; Space, X, Shift for fire/bomb; P/Esc to pause
- **Mouse**: Hover to move player toward cursor (lowest priority input)
- **Touch**: Virtual joystick on left half of screen for movement; FIRE/BOMB buttons on right half
- Priority: Keyboard/joystick > mouse steering

### Performance Adaptation
Tracks frame time over 45 frames and adjusts `quality` level (0 = full, 1 = medium, 2 = low) to reduce particle/shadow effects under framerate pressure.

### Canvas Scaling
Base resolution is 480×720px (landscape phone). Scales up by 1.6× max using CSS transform while keeping logical canvas size fixed for collision/drawing math.

## Development Notes

### No External Dependencies
The game has **zero external dependencies**—no libraries, frameworks, or CDN scripts. Pure HTML, CSS, JavaScript using native APIs (Canvas, WebAudio, localStorage, `requestAnimationFrame`).

### Difficulty Balancing
Adjust `DIFFICULTIES` object (line 485) for spawn/enemy rates and lives. Boss HP multiplier applies based on stage and difficulty. Enemy types unlock at specific `minStage` values (line 505).

### Boss Attack Patterns
Boss patterns are arrays of functions that fire bullets in specific formation. Edit `BOSS_PATTERNS` (line 1372) to change boss behavior. Each boss design has unique hull colors and core glow color defined in `BOSSES` array (line 508).

### Visual Theming
Six sky themes cycle by stage in `THEMES` array (line 493). Each theme has a gradient and cloud/sun color. Themes repeat after stage 6.

### Scoring & Chains
Killing enemies increments a chain counter that resets after 2.2 seconds of no kills. Chain multiplier = 1 + floor(chain / 5), up to 10×. Boss kills award flat 500 points.

### Reduced Motion
Respects `prefers-reduced-motion` media query (line 520) and allows users to toggle. When enabled, screen shake is reduced to 15% magnitude and capped at 0.1s duration; animations are disabled via CSS (line 350–358).

### UI Overlays
The overlay element (`#overlay`) shows title screen, pause, and game-over screens. Difficulty selection persists to localStorage. Best score is also persisted.

## Development Commands

**No build process required.** Simply open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge).

For **local development**, use any static file server:
```bash
# Python 3
python -m http.server 8000

# Node.js (with http-server)
npx http-server

# Or open directly: file:///path/to/Google_4/index.html
```

**Testing**: Open browser DevTools (F12) and check:
- Console for any JS errors
- Network tab (should be empty—no external requests)
- Application tab → LocalStorage for persisted best score and settings

## Code Style Notes

- Single IIFE wrapping the entire game (line 423) prevents global scope pollution
- Dense one-file structure; future modularization would split rendering, physics, input, audio
- Drawing functions (`drawPlane`, `drawBossShip`, `draw`) use canvas state save/restore for isolation
- Physics uses simple AABB (axis-aligned bounding box) collision detection via `rectsOverlap()`
- Enemy behaviors are parameterized (weave pattern, shoot cooldown, type-specific visuals)
