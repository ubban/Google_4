# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, self-contained Raiden-style vertical shoot-'em-up game built with vanilla HTML, CSS, and JavaScript. The entire game lives in `index.html` (~1670 lines) with no external dependencies.

## Quick Start

- **View the game**: Open `index.html` in a web browser
- **No build step**: This is a plain HTML file — just open it
- **No dependencies**: Pure HTML5 Canvas, Web Audio API, and vanilla JS

## Code Architecture

The game logic is organized into logical sections within the `<script>` tag:

### Core Game State & Constants
- **Game variables** (lines 864-869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, stage tracking
- **Difficulty settings** (lines 485-489): Three difficulty presets (easy/normal/hard) controlling spawn rates, enemy behavior, and HP multipliers
- **Enemy types** (lines 502-506): Grunt, Interceptor, Tank — each with different HP, speed, and scoring
- **Boss designs** (lines 508-515): Six unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with different visual styles and attack patterns
- **Visual themes** (lines 493-500): Six themed skies with different colors and clouds

### Audio System (Lines 542-676)
- **WebAudio synthesis**: Uses oscillators, filters, and noise bursts to generate all sound effects procedurally (no asset files)
- Key functions:
  - `beep(freq, dur, type, vol, opts)` — Tone generation with optional sweep and filtering
  - `noiseBurst(dur, vol, opts)` — Procedural noise for explosions and impacts
  - `sfx` object: Contains all 16 sound effects (shoot, laser, explode variants, bomb, powerup, bossTelegraph, etc.)
- Audio is muted via `muted` flag and respects the mute button state

### Rendering (Lines 507-638, 913-1020, 1520-1638)
- **Canvas setup** (lines 752-765): Responsive 480×720 canvas that scales to fit window while maintaining aspect ratio
- **Plane drawing** (lines 913-968): Draws player and enemy aircraft with gradient fills and layered paths
- **Boss drawing** (lines 970-1020): Complex multi-part boss ships with rotating components and animated cores
- **Draw function** (lines 1520-1638): Main render loop that draws sky gradient, clouds, all game objects, and particles

### Input Handling (Lines 767-861)
- **Keyboard input**: Arrow keys / WASD for movement, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse input**: Move cursor to steer, click to fire (has priority when no keyboard/joystick input is active)
- **Touch input**: Left half of screen is movement zone with joystick UI, right half has FIRE and BOMB buttons
- **Firing state**: Managed separately for keyboard and mouse (allows continuous fire)

### Game Loop & Physics (Lines 1148-1302)
The `update(dt)` function handles all frame-by-frame logic:
- **Player movement**: Blends keyboard/joystick input with mouse-steering when keyboard is idle
- **Weapon firing**: Different fire rates for vulcan (0.15s) vs laser (0.1s)
- **Enemy spawning**: Accelerates over time; spawn rate controlled by difficulty and frame count
- **Bullet collision**: Checks bullets against all enemies and boss; enemy bullets against player
- **Enemy behavior**: Weaving pattern for interceptors; aimed vs spread fire patterns
- **Chain mechanics**: Killing enemies within 2.2s increases multiplier (x1 to x10 based on kill count)

### Boss System (Lines 1372-1439)
- **Boss patterns** (lines 1372-1409): Each boss has 3 unique attack patterns rotated in sequence
  - Pattern functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- **Boss behavior** (lines 1411-1439): Enters from top, bounces horizontally, telegraphs attacks (visual + audio), cycles patterns
- **Transition**: Boss appears after killing 18 regular enemies per stage

### UI & Overlay (Lines 368-419, in CSS)
- **HUD elements**: Score, best score, combo multiplier, weapon level, hearts (lives), bombs
- **Overlay**: Game-over screen, pause screen, stage banners with animations
- **Accessibility**: Screen-reader live region (`srLive`) announces key events
- **Reduced motion support**: Disables animations, screen shake, and flash effects when enabled

### State Persistence (Lines 517-527)
Uses `localStorage` to persist:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Motion reduction preference

## Key Design Decisions

1. **Single-file architecture**: Keeps the entire game in one file for easy deployment and modification; no build tools needed
2. **Procedural audio**: All sound effects are synthesized via Web Audio API — no audio files, no licensing concerns
3. **Adaptive quality**: Frame-time tracking auto-adjusts particle count and shadow effects based on device performance
4. **Chain multiplier system**: Rewards continuous play with increasing score multiplier (up to 10×) to encourage engaging gameplay
5. **Difficulty scaling**: Different difficulty presets adjust spawn timing, enemy behavior, and boss HP; player can change difficulty anytime
6. **No external assets**: Everything (graphics, audio) is generated in code for instant loading and small footprint

## Development Notes

### Making Changes
- The game state is managed in the closure around the IIFE (Immediately Invoked Function Expression)
- Global constants like `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES` are easy to tweak
- Add new enemy types by extending `ENEMY_TYPES` and updating enemy spawn logic
- Add new bosses by extending `BOSSES` and `BOSS_PATTERNS` in parallel
- To add a new sound effect, create a new function in the `sfx` object using `beep()` and `noiseBurst()`

### Canvas Rendering
- Canvas coordinate system: (0,0) is top-left; positive x goes right, positive y goes down
- All drawing uses `ctx` (the 2D context); save/restore state with `ctx.save()` / `ctx.restore()`
- Screen shake is applied by translating the canvas during draw, controlled by `shakeTime` and `shakeMag`

### Testing & Debugging
- Open `index.html` in a browser's developer tools
- The game respects system prefers-reduced-motion; test with devtools emulation if needed
- Audio context requires user interaction to start (ensureAudio() is called on input)
- Mobile testing: Use device emulation or actual mobile device; touch controls activate automatically on touch devices

### Performance
- `quality` variable (0/1/2) tracks device performance and reduces particle count / shadow effects
- Frame time tracking in `trackPerf()` auto-adjusts quality if avg frame time exceeds 1/40s or drops below 1/56s
- Canvas max scale is clamped to 1.6× to avoid performance issues on very large screens

## Browser Compatibility

- Requires ES6 (arrow functions, const/let, template literals, destructuring)
- Web Audio API (for synth sound effects)
- Canvas 2D API
- Touch API (optional; keyboard always works)
- localStorage (optional; game works without it, just no high score persistence)

Works well on modern browsers (Chrome, Firefox, Safari, Edge). Gracefully degrades if Web Audio is unavailable (mutes audio).

## File Structure

```
Google_4/
├── index.html          # Entire game (1669 lines)
├── README.md           # Project metadata
└── CLAUDE.md          # This file
```

No external JavaScript files, no CSS files, no asset folders — everything in index.html.
