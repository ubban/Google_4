# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It's a canvas-based game with synthesized audio (Web Audio API), responsive design, and touch/keyboard controls.

### File Structure

- `index.html` — The entire game (~1,670 lines): HTML layout, CSS, and JavaScript game engine
- `README.md` — Minimal project metadata
- No build tools, dependencies, or external assets required

## Running and Testing

Since this is a static HTML file, there's no build step required.

**To play locally:**
- Open `index.html` directly in a web browser, or
- Serve it via a local server: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`

**Testing changes:**
- Changes to `index.html` take effect immediately on browser refresh (no build required)
- Test on multiple devices/viewports since the game is fully responsive
- Verify both keyboard (`WASD` + `Space` for shoot, `Z` for bomb) and touch controls
- Test on mobile devices to verify touch UI appears and joystick works

## Game Architecture

The entire game runs in a single JavaScript IIFE (Immediately Invoked Function Expression) within `index.html`, starting at line 422.

### Core Systems

**Game Loop**
- Runs via `requestAnimationFrame` at ~60 FPS
- Tracks frame times for adaptive performance quality scaling
- Main update flow: player input → update entities → collision detection → rendering

**Game States**
- **Menu**: Overlay showing welcome screen, difficulty selection, stats
- **Playing**: Active gameplay with HUD showing score, health, bombs, weapons
- **Game Over**: Final stats display with score/best score/stage reached
- Uses `overlay` element to show/hide states

**Difficulty Settings** (line 485-489)
- Three presets: `easy`, `normal`, `hard`
- Each controls: enemy spawn rate, movement speed, fire rate, lives, bombs, boss HP scaling
- Stored in `localStorage` under key `skyfighter.difficulty`

**Enemy Types** (line 502-506)
- `grunt` — Basic red enemy, hp:1, score:10
- `interceptor` — Fast orange enemy with weaving pattern, hp:1, score:15
- `tank` — Purple heavy unit, hp:4, score:35 (appears from stage 2)
- Each has color, dimensions, speed multiplier, shoot chance, and minimum stage

**Bosses** (line 508-515)
- Six boss types: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Cycle through as player completes stages (18 kills per stage)
- Each has hull colors, core color, and HP multiplier
- Boss health bar shown at top during encounters

**Weapon System**
- Player starts with basic weapon, can level up to 5
- Higher levels spread more bullets, faster fire rate
- Dropped by enemies (occasional pickups)
- Displayed in HUD with level number and name

**Bomb System**
- Player starts with limited smart bombs (2 or 3 depending on difficulty)
- Triggered by `Z` key or mobile bomb button
- Clears many projectiles, damages enemies
- Visual feedback via flash animation and vignette effect

### Audio System (line 542-620+)

Uses Web Audio API exclusively—no external audio files.

- `ensureAudio()` — Initializes AudioContext on first user interaction
- `beep(freq, dur, type, vol, opts)` — Core synth function; creates square/sine waves
- Options: `sweepTo` (frequency sweep), `filterType`/`filterFreq` (lowpass), `pan` (stereo position), `delay`
- Sounds for: shooting, hits, explosions, power-ups, stage transitions, game over

### HUD and Accessibility

- **HUD Display** (top center): Score, best score, weapon level, multiplier, health (hearts), bombs
- **Screen Reader Support**: `#srLive` element announces important events (state changes, new best score)
- **Reduced Motion**: Checkbox to disable animations/shake for accessibility; stored in localStorage
- **Touch Controls**: Virtual joystick + fire/bomb buttons appear on mobile; keyboard always available on desktop

### Themes (line 493-500)

Six color themes that change sky gradient, cloud color, sun color:
1. Daylight Front (bright blue)
2. Sunset Corridor (warm orange/pink)
3. Night Approach (dark blue)
4. Storm Front (greenish)
5. Arctic Wastes (light blue)
6. Volcanic Ridge (orange/red)

Themes cycle as player progresses through stages.

### Local Storage Keys

- `skyfighter.bestScore` — Best score ever achieved
- `skyfighter.muted` — '1' if sound is muted, '0' otherwise
- `skyfighter.difficulty` — 'easy', 'normal', or 'hard'
- `skyfighter.reducedMotion` — '1' if reduced motion is on, '0' otherwise

## Common Modifications

**Adjust Difficulty** — Edit the `DIFFICULTIES` object (line 485-489) to tweak spawn rates, enemy speeds, or HP multipliers.

**Add/Modify Enemy Types** — Add a new entry to `ENEMY_TYPES` object (line 502-506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`.

**Add/Modify Bosses** — Edit `BOSSES` array (line 508-515) to change colors or HP multipliers; add new entries with `name`, hull colors, core color, and `hpMul`.

**Change Themes** — Edit the `THEMES` array (line 493-500) to add or modify sky gradients and colors.

**Adjust Game Constants**
- `KILLS_PER_STAGE` (line 491): enemies required to clear a stage and face a boss
- Canvas dimensions and responsive scaling handled by CSS and viewport meta tag

**Audio Changes** — Modify `beep()` calls throughout the code to adjust pitch, duration, or effect (sweep, filter). Look for calls like `beep(800, 0.1, 'square', 0.5)` to understand current sound effects.

## Performance Notes

- **Adaptive Quality** (line 473-483): Game automatically adjusts rendering quality (full → medium → low) based on frame time performance
- **Touch Optimization**: Disabled tap highlighting and text selection for smoother mobile experience
- **Canvas Rendering**: Uses 2D canvas context for all drawing; no WebGL or external libraries
- **Collision Detection**: Simple bounding-box checks between player bullets, enemies, bosses, and pickups

## Browser Compatibility

- Requires HTML5 Canvas, Web Audio API, and modern CSS (flexbox, viewport units)
- Works on modern Chrome, Firefox, Safari, Edge
- Mobile-friendly with touch event handling and responsive viewport
- Gracefully handles missing Web Audio (mutes if not available)
