# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style), built as a single-file HTML5 application with embedded CSS and JavaScript. No build process, no dependencies, no external assets.

## Quick Start

- **Open the game:** Open `index.html` directly in a web browser
- **No build or server needed** — it runs immediately as a static HTML file
- **Responsive design** — works on desktop, tablet, and mobile with adaptive touch controls

## Architecture

The entire game logic is in `index.html` as a single IIFE (`(function () { 'use strict'; ... })()`). The structure is:

### Game Mechanics

- **Player** — plane that moves and shoots, with 2 weapon types (Vulcan/Laser) and bombs
- **Enemies** — multiple types (grunt, interceptor, tank) spawn in waves based on difficulty
- **Bosses** — stage bosses with varied attack patterns; defeat to advance stages
- **Scoring** — kill chains increase multiplier (1-10x based on kills/5); chain timer resets after 2.2s without kills
- **Difficulty settings** — Easy/Normal/Hard adjust spawn rate, enemy speed, shoot chance, lives, bombs, boss health
- **6 unique bosses** rotating per stage with distinct color schemes and attack patterns
- **6 visual themes** (sky gradients) that rotate per stage

### Key Systems

**Adaptive Performance (lines 473–483)**
- Tracks frame times over 45 frames; if average > 1/40 ms, reduces quality (disables shadows/blur)
- Three quality levels: 0 (full), 1 (medium particle count), 2 (low particle count)
- Ensures smooth gameplay on lower-end devices

**Input Handling (lines 768–861)**
- Keyboard: Arrow keys/WASD for movement, Space/X/Shift for fire/bomb
- Mouse: Hover to steer, click to fire (mouse input prioritized over keyboard when active)
- Touch: Joystick in bottom-left, FIRE/BOMB buttons in bottom-right
- No input conflicts — keyboard/joystick always override mouse

**Collision Detection (lines 1069–1071)**
- Simple rect-overlap function used for all collisions (bullets, enemies, bosses, power-ups)
- Enemy bullets removed from array after hit; player bullets pierce based on weapon type

**Weapon System**
- **Vulcan** (level 1–3): fires 1, 2, or 3 bullets straight up; fast cooldown (0.15s)
- **Laser** (level 1–3): fires 1–3 laser beams with pierce property; slower cooldown (0.1s); auto-switches after level 3

**Boss Attack Patterns (lines 1372–1409)**
- 6 pattern sets (one per boss), each with 3 different attacks that rotate
- Patterns include: radial bursts, aimed spreads, columns, teleport + attack, spirals, homing shots
- Telegraph phase (visual warning) before each attack; wave mechanics via `radial` property on bullets

**Audio (lines 542–676)**
- Web Audio API synth — no MP3/WAV files
- Master compressor + gain control; muted flag persists to localStorage
- Pan property on sounds; frequency sweeps for pitch effects
- Separate SFX functions for each event (shoot, explode, hit, bomb, powerup, boss, gameover)

**Particle System (lines 1073–1085)**
- Explosions spawn particles with random velocity and 0.5s lifetime
- Particle count scales with quality level for performance

**UI State (lines 461–467)**
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- HUD updates dynamically (score, best, multiplier, weapon, hearts, bombs, boss bar)
- Overlay card for menu/pause/game-over; animations use CSS

### Game Loop (lines 1640–1653)

1. Measure delta-time; clamp to max 0.05s
2. Track perf (adaptive quality)
3. If running & not paused: update game state → draw
4. RequestAnimationFrame continues loop

### Update Phase (lines 1148–1302)

1. Handle input (keyboard, joystick, mouse) → move player
2. Update cooldowns, invuln, chain timer
3. Move bullets, enemies, enemy bullets, power-ups
4. Spawn enemies (based on difficulty/spawn timer)
5. Spawn boss after 18 kills (KILLS_PER_STAGE)
6. Update boss state/attacks
7. **Collision detection**: enemy bullets vs player, enemies vs player, bullets vs enemies/boss, power-ups vs player
8. Update particles

### Draw Phase (lines 1520–1638)

1. Apply screen shake if active (random offset)
2. Draw sky gradient (theme-based)
3. Draw sun glow + clouds (two layers at different speeds)
4. Draw player (with laser aura if laser weapon)
5. Draw mouse cursor crosshair (if mouse active)
6. Draw bullets (with shadows/glow if quality=0)
7. Draw enemy bullets (red circles)
8. Draw enemies + tank HP bars
9. Draw boss (if active)
10. Draw power-ups (rotating squares, color-coded)
11. Draw particles (expanding circles with fade)

## Development Notes

### To Add Features

- **New enemy type** — add to `ENEMY_TYPES` object (lines 502–506), add to `DIFFICULTIES` if needed, add spawn logic
- **New weapon** — add fire function, add to `fireWeapon()` cooldown/type check, add audio SFX
- **New boss pattern** — add function to BOSS_PATTERNS array (lines 1372–1409), update pattern fire functions if needed
- **New power-up type** — add to `maybeSpawnPowerUp()` logic (1058–1067), add sprite draw in draw phase, handle in `applyPowerUp()` (1304–1322)

### Performance Considerations

- **Particle cap** — adaptive quality reduces particle count on slow devices
- **Enemy spawn cap** — limited by difficulty spawn timer, naturally caps active enemies
- **Collision checks** — O(n²) for bullets vs enemies; acceptable with reasonable enemy count
- **Canvas size** — fixed 480×720 logical, scaled to window; never exceeds 1024×1536 logical
- **Audio context** — created on-demand on first user interaction (ensureAudio)
- **localStorage** — minimal use (4 keys); accessed only on init and pause/setting changes

### Testing/Debugging

- Open DevTools; no errors in console expected during normal gameplay
- Try each difficulty (Easy/Normal/Hard) to verify spawn/enemy speed changes
- Reach stage 2+ to verify boss spawning and pattern variation
- Test on mobile device or use DevTools device emulation for touch controls
- Toggle reduced motion (button in top-right) to verify animation changes
- Toggle mute (button in top-right) to verify audio works

### Accessibility

- **Reduced motion preference** — detected on init; respects `prefers-reduced-motion: reduce` media query
- **Screen reader support** — ARIA live region (`#srLive`) for game status announcements
- **Keyboard controls** — full game playable without mouse
- **Color contrast** — HUD text has text-shadow for readability over any background

## Known Limitations / Gotchas

- **No infinite scaling** — quality levels are fixed; very low-end devices may still struggle
- **Boss count** — only 6 bosses defined; stage 7+ will cycle through (modulo)
- **Theme count** — only 6 themes; stage 7+ cycles (modulo)
- **Touch doesn't show joystick base** — base only visible during active touch (intentional to avoid clutter)
- **No cloud parallax speed adjustment** — always move at fixed speeds regardless of stage/difficulty

## Files

- `index.html` — entire game (HTML, CSS, JS)
- `README.md` — minimal project info with timestamp
- `CLAUDE.md` — this file
