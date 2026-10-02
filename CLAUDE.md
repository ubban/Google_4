# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/CSS/JavaScript application. It runs entirely in the browser with no external dependencies—all audio is synthesized using the Web Audio API.

## Running the Game

Since this is a single HTML file, there are no build steps or external dependencies:

1. **Open locally**: Open `index.html` directly in a web browser (desktop or mobile)
2. **Server (recommended)**: For local development, serve via HTTP to avoid CORS issues with audio context:
   ```bash
   # Python 3
   python -m http.server 8000
   # Then visit http://localhost:8000/
   
   # Or use any other simple server (Node, Ruby, etc.)
   ```
3. **Deploy**: Copy `index.html` to any static web host (GitHub Pages, Vercel, etc.)

## High-Level Architecture

The entire game is contained in one file with three main sections:

### 1. **Markup + Styling** (lines 1–359)
- HTML structure: canvas, HUD elements, overlay screens, touch controls
- CSS: responsive layout with clamp() for mobile scaling, animations for UI feedback, shader-less visual effects

### 2. **Game Constants** (lines 423–523)
- **DIFFICULTIES**: Three presets (easy/normal/hard) controlling spawn rates, enemy speed, lives, bombs, and boss HP
- **THEMES**: Six visual themes (sky gradients, cloud colors, sun effects) that cycle per stage
- **ENEMY_TYPES**: grunt, interceptor, tank—each with HP, speed multiplier, score, and behavior flags
- **BOSSES**: Six unique bosses with color designs and HP multipliers; attack patterns defined separately in BOSS_PATTERNS

### 3. **Core Game Loop** (lines 863–1665)
Orchestrates entity updates, collision detection, rendering, and state transitions:
- **Input**: Keyboard (WASD/Arrows for movement, Space/click to fire, X/Shift/click for bombs), mouse steering, and touch controls
- **Entities**: Player, bullets, enemies, enemy bullets, particles, power-ups, boss
- **Phases**: Title → Game Running → Pause/Game Over

## Key Systems

### Game State Machine
- **overlay.hidden**: Controls title/pause/game-over screens
- **running/paused/gameOver**: Flags that gate update and input processing
- **stage/stageKills**: Progress tracking; 18 kills trigger boss spawn

### Collision & Physics
- **AABB rectangles**: All entities use `.x`, `.y`, `.w`, `.h` with `rectsOverlap()` for broad-phase
- **Smooth movement**: Keyboard/joystick move freely; mouse-only input applies easing toward target
- **Bullet pierce**: Laser bullets (weapon type) can pass through `pierce` enemies; spread-pattern bullets (Vulcan) are one-hit

### Scoring & Progression
- **Chain multiplier**: Killing enemies within 2.2 seconds increases multiplier by 1 per 5 kills (max 10x)
- **Break chain**: Taking damage or waiting resets to 1x
- **Power-ups**: Life, bomb count, or weapon upgrade (Vulcan lv1→3, then toggle to Laser)

### Boss System (lines 1372–1454)
- **Design**: Six bosses cycle by stage (mod 6), each with unique color scheme and HP multiplier
- **Patterns**: 3 attacks per boss, played in sequence; stored in `BOSS_PATTERNS` array indexed by boss order
- **Telegraph**: 22-frame warning before attack fires, with boss core flashing and audio cue
- **Movement**: Sinusoidal horizontal drift bounded by canvas edges
- **Damage**: Bullets deal 3 HP per hit; bombs deal 24 HP (one-shot for most bosses at default difficulty)

### Audio (WebAudio Synth, no files)
- **Ensured on first user input**: `ensureAudio()` creates context and compressor
- **SFX functions**: `beep()` for tonal sounds, `noiseBurst()` for explosions; both support sweep, filter, pan
- **Mute state**: Stored in localStorage; checked before all sound generation
- **Reduced motion**: Setting dims screen shake, flash, and animation effects (respects system prefers-reduced-motion)

### Responsive & Accessible
- **Canvas sizing**: Base 480×720 pixel game area, scaled to fit viewport with max scale 1.6×
- **Touch controls**: Joystick on left 55% of screen, Fire/Bomb buttons on right 45%; shows on `ontouchstart` detection
- **Screen reader**: `#srLive` announcements for kill milestones, stage changes, life loss, game over
- **Reduced motion mode**: Toggleable via UI button; respects OS preference on first load

## Editing Guide

### Adding a New Enemy Type
Add to `ENEMY_TYPES` (line 502):
```javascript
myEnemy: { w: 32, h: 32, hp: 2, speedMul: 1.2, color: '#...', dark: '#...', score: 20, minStage: 3, weave: false }
```
Then handle spawning: filter by `minStage` in `spawnEnemy()` (line 1026) and add explosion SFX in `EXPLODE_SFX` (line 678) if unique.

### Adding a New Boss Attack Pattern
1. Add a function in the helper section (e.g., `fireWave()`) using `enemyBullets.push({ x, y, w, h, radial, dx, dy })`
2. Add a 3-attack array to `BOSS_PATTERNS` (line 1372) in boss order
3. Bosses are spawned in order mod 6; no need to register elsewhere

### Balancing Difficulty
- **Enemy spawn rate**: Adjust `spawnBase` and `spawnMin` in `DIFFICULTIES` (line 485)
- **Enemy damage**: Change `enemyShootChance` and firing pattern frequencies
- **Boss HP**: Multiply via `bossHpMul` per difficulty; each boss also has `hpMul` (e.g., TITAN is 1.7×)
- **Player HP/bombs**: Change `lives` and `bombs` in `DIFFICULTIES`

### Changing Visuals
- **Sky theme**: Edit color stops in `THEMES` array (line 493)
- **Enemy color**: Change `color` and `dark` fields in `ENEMY_TYPES`
- **Boss appearance**: Edit `hullTop`, `hullBottom`, `wing`, `core` in `BOSSES` array
- **Particles**: Adjust spawn count in `spawnExplosion()` quality scale (line 1074)

### Performance Tuning
- **Adaptive quality**: Controlled by `trackPerf()` (line 476); lowers particle count and shadow blur on slow frames
- **Quality levels**: 0=full, 1=medium (60% particles, no shadows), 2=low (35% particles)
- **Max shadow blur**: Set `ctx.shadowBlur` thresholds in `drawBossShip()` (line 1002) and `draw()` (line 1579)

## Common Workflows

**Testing a new boss attack pattern**:
1. Add pattern function and add to `BOSS_PATTERNS`
2. Temporarily set stage to the boss you want: `stage = 1` in `resetGame()` and `spawnBoss()` will auto-trigger after KILLS_PER_STAGE (18)
3. Or directly call `spawnBoss()` from console

**Debugging collisions**:
- Add a debug draw of bounding boxes: loop entities and stroke their (x-w/2, y-h/2, w, h) rects
- Add breakpoint in `rectsOverlap()` to inspect entity positions

**Audio troubleshooting**:
- Check `muted` flag and localStorage key `skyfighter.muted`
- Ensure `ensureAudio()` has been called (happens on first keydown or touch)
- Browser may suspend audio context; `ensureAudio()` calls `audioCtx.resume()` if needed

## Storage Keys

Game preferences are persisted in localStorage:
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Reduced motion toggle ('0' or '1')

## Mobile Notes

- **Touch steering**: Drag left side of screen to move; one finger per hand
- **Fire button**: Tap red circle on right
- **Bomb button**: Tap yellow button
- **Safe area insets**: UI respects notches/dynamic islands via `env(safe-area-inset-*)` in CSS
- **Viewport**: Locked to portrait with maximum-scale=1 to prevent zoom issues
