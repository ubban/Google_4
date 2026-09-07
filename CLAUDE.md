# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file (`index.html`). The game runs entirely in the browser with no external dependencies — all graphics are canvas-based and audio is procedurally generated using the WebAudio API.

## Running the Game

**To play the game:** Simply open `index.html` in a web browser. No build process, bundler, or server is required.

The game adapts to any window size and supports:
- **Desktop:** Mouse steering (move to steer, click to fire), keyboard controls (Arrow keys/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause)
- **Mobile/Touch:** On-screen joystick and button controls that appear automatically on touch devices

## Architecture Overview

The entire codebase is a single ~1670-line HTML file structured as:

1. **HTML markup** (lines 1–420): DOM structure for the canvas, HUD, overlay UI, and touch controls
2. **CSS styling** (lines 10–358): Layout, animations, responsive design, accessibility features
3. **JavaScript game engine** (lines 422–1669): Game loop, physics, rendering, input handling, and game logic

### High-Level System Design

The game is organized around these core concepts:

- **Game loop**: Runs via `requestAnimationFrame`, updates at variable frame rate with delta-time compensation
- **Entity system**: Player, enemies, bullets, particles, and bosses are simple objects with position, velocity, and type-specific behavior
- **Rendering**: Single canvas context renders background gradient, entities, and visual effects
- **Difficulty layers**: Three difficulty presets (easy/normal/hard) adjust spawn rates, enemy speed, and boss HP
- **Progressive stages**: Game progresses through numbered stages; after 18 enemy kills, a boss fight begins
- **Combo/multiplier system**: Consecutive kills within 2 seconds increase a multiplier; resets on player hit

## Key Systems

### 1. Input Handling (lines 767–861)
- **Keyboard:** `keys` object tracks held keys; keycodes like `KeyP`, `Space`, `KeyX` trigger pause/bomb
- **Mouse:** `mouseActive` flag and `mouseTarget` position enable mouse steering (glides toward pointer)
- **Touch:** Joystick in left zone (`moveZone`), fire/bomb buttons in right zone; `touchstart/touchmove/touchend` handlers update `joyVec` in range [-1, 1]
- Priority: keyboard/joystick > mouse (prevents fighting between input types)

### 2. Game State & Loop (lines 863–1050)
- **State variables:** `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`
- **Main loop** (`gameLoop` function): Updates all entities, checks collisions, manages spawn rates, and renders frame
- **Performance adaptation:** `trackPerf()` monitors frame time and adjusts `quality` level (0=full, 1=medium, 2=low) to maintain ~60 fps

### 3. Difficulty & Progression (lines 485–500)
- **`DIFFICULTIES` object:** Defines spawn rates, enemy speed multipliers, enemy shoot chance, lives, and boss HP scaling per difficulty
- **Stages:** Advance to next stage after 18 enemy kills. Boss appears at stage start with unique appearance and attack patterns
- **Boss types:** Six boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with varying HP multipliers and color schemes

### 4. Enemy AI (lines 1100–1200+)
- **Enemy types:** `grunt` (basic), `interceptor` (fast, weaves), `tank` (slow, multi-hit)
- **Spawn logic:** Enemies spawn randomly within time intervals determined by difficulty; min/max spawn times decrease as stage increases
- **Enemy behavior:** Move downward, shoot at player based on random chance, despawn if off-screen
- **Boss behavior:** Boss moves in patterns, telegraphs attacks with sound cues, spawns projectiles in waves

### 5. Player & Weapons (lines 875–1050)
- **Player object:** Position, dimensions (40×40), speed, cooldown timer, weapon type (`vulcan` or `laser`), weapon level, and invulnerability timer
- **Vulcan:** Rapid-fire bullet spread; fires when `cooldown ≤ 0`
- **Laser:** Single high-damage beam; replaces vulcan when weapon level ≥ 2 and laser pickup is collected
- **Weapon upgrades:** `+1` pickup increases level; `laser` pickup switches to laser weapon

### 6. Audio System (lines 542–676)
- **WebAudio synth:** No external audio files; all sounds generated procedurally using oscillators, noise, and filters
- **`beep()` function:** Creates pitched tones with optional frequency sweep, filter, stereo pan, and envelope
- **`noiseBurst()` function:** Creates filtered white noise for impact/explosion effects
- **Sound effects object (`sfx`):** Combines beeps and noise bursts for game events (shoot, explode, hit, bomb, power-up, boss telegraph, etc.)
- **Mute state:** Stored in localStorage; all audio checks `if (muted || !audioCtx)` before synthesis
- **Spatial audio:** Panning based on entity X position via `panFor()` helper

### 7. Rendering & Visual Effects (lines 1350–1500+)
- **Background:** Gradient from theme colors; changes per stage (6 themes: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- **Entities:** Drawn as simple colored rectangles with dark/light shading or custom shapes (boss hull/wings/core)
- **Animations:** CSS animations for HUD, combo pop, stage banner; JS-driven flash effects (white hit/bomb overlays), vignette pulse on critical health
- **Screen shake:** Reduced motion mode caps shake to 15% amplitude or 0.1s duration
- **Particles:** Simple fade-out rectangles for visual feedback on destruction

### 8. UI & Accessibility (lines 680–750)
- **HUD:** Score, best score, multiplier, weapon/level, hearts (lives), bomb count
- **Overlay:** Start screen with difficulty selection and controls hint
- **Screen reader support:** `#srLive` (aria-live) announces state changes via `announce()` helper
- **Reduced motion:** Stored in localStorage; toggles CSS class that disables all animations for users who request it

### 9. Local Storage (lines 463–527)
- **`skyfighter.bestScore`:** High score persists across sessions
- **`skyfighter.muted`:** Mute state
- **`skyfighter.difficulty`:** Last selected difficulty
- **`skyfighter.reducedMotion`:** Accessibility preference

## Common Modifications

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object (e.g., `{ w, h, hp, speedMul, color, dark, score, minStage }`)
2. Add spawn logic in the spawn timer section to randomly select the type
3. Add explosion SFX to `EXPLODE_SFX` mapping if desired
4. Update `createEnemy()` to initialize any type-specific behavior

### Add a new boss
1. Add entry to `BOSSES` array with name and color scheme
2. Extend boss draw logic to render hull/wings/core or use existing template
3. Add boss attack pattern in the boss update section
4. Boss health is automatically scaled by `bossHpMul` and difficulty multiplier

### Adjust difficulty balance
- Modify spawn rates in `DIFFICULTIES.easy/normal/hard`
- Tweak `enemySpeed` multiplier or `enemyShootChance`
- Adjust `lives`, `bombs`, or `bossHpMul` per difficulty

### Change visual theme
- Modify `THEMES` array to add new sky gradients and cloud/sun colors
- Themes cycle per stage; edit the selection logic in the render function if needed

### Add sound effects
- Write new SFX by combining `beep()` and `noiseBurst()` calls in the `sfx` object
- Call `ensureAudio()` first if adding new user interaction that triggers sound

## Technical Notes

- **Canvas scaling:** Base resolution is 480×720; CSS scaling preserves aspect ratio up to 1.6× on large screens
- **Frame rate independence:** Delta time (`dt`) is calculated and passed to update functions to ensure consistent behavior at any frame rate
- **Entity lifecycle:** Bullets and enemies are removed when off-screen or destroyed; particles fade over time and are culled when opacity ≈ 0
- **Collision detection:** Simple axis-aligned bounding box (AABB) checks between player/enemies, bullets/enemies, and player/projectiles
- **Boss health bar:** Shows only when a boss is active; updates smoothly via CSS transition
- **Combo window:** Multiplier increases on kill if previous kill was within 2 seconds; decays after 3 seconds of no kills

## Browser Support

The game requires:
- Canvas 2D context (`getContext('2d')`)
- WebAudio API (for audio; gracefully degrades to silent if unavailable)
- Touch events and `touchstart`/`touchmove`/`touchend` (for mobile)
- `requestAnimationFrame` (for game loop)
- LocalStorage (for high score and settings persistence)

Works on all modern browsers (Chrome, Firefox, Safari, Edge) and mobile browsers (iOS Safari, Chrome Mobile, Samsung Internet).
