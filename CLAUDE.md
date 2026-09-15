# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. The entire codebase is contained in `index.html` (~1670 lines): HTML structure, CSS styling, and vanilla JavaScript game logic. No build tools, frameworks, or external assets are required.

## Running the Game

- **Open directly:** `open index.html` or drag `index.html` into a browser
- **Local server:** `python -m http.server 8000` then visit `http://localhost:8000/index.html`
- **No installation needed:** The game is self-contained and works offline

## Code Architecture

### High-Level Structure

The game follows a classic **game loop pattern** (RAF-based frame-by-frame update and draw):

1. **Setup (Lines 423-527):** Global state, DOM references, settings init
2. **Audio Engine (Lines 542-676):** WebAudio synth for procedural SFX, no external audio files
3. **Rendering Functions (Lines 680-1638):** Canvas drawing for planes, particles, UI
4. **Game Logic (Lines 863-1504):** Collision, scoring, boss AI, difficulty
5. **Input Handling (Lines 767-861):** Keyboard, mouse, touch + joystick support
6. **Main Loop (Lines 1640-1665):** `requestAnimationFrame` driving updates and renders

### Key Game State Variables

```javascript
// Player
player = { x, y, w, h, speed, weapon, level, cooldown, invuln, ... }

// Collections (parallel arrays)
bullets, enemies, enemyBullets, particles, powerUps, boss

// Scoring & progression
score, lives, maxLives, chain, multiplier, stage, stageKills, kills

// UI & settings
difficulty, muted, reducedMotion, quality (0=full, 1=medium, 2=low)
```

### Difficulty System

Three presets in `DIFFICULTIES` (line 485): **easy**, **normal**, **hard**. Each modifies spawn rate, enemy behavior, lives, bombs, and boss HP. Difficulty is persistent in localStorage.

### Enemies & Bosses

- **3 enemy types** (line 502): grunt, interceptor, tank — each with unique HP, speed, score, and appearance
- **6 boss types** (line 508-515): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Boss patterns** (line 1372-1409): Each boss has 3 unique attack patterns using procedural bullet spawning

### Weapons & Upgrades

- **Vulcan:** Wide spread fire (1 → 2 → 3 lanes), default weapon
- **Laser:** Piercing shots, tighter grouping (1 → 2 → 3 lanes), obtained from weapon pickups
- Weapon upgrades loop: pick up enough to reach level 3, then next pickup switches weapon and resets to level 1

### Scoring System

- **Base points:** Enemies (10-35), bosses (500)
- **Chain multiplier:** Consecutive kills increase multiplier (1x → 10x every 5 kills), broken by taking damage
- Combo at 10 kills triggers UI announcement

### Performance Adaptation

Lines 473-483: Tracks frame times and adjusts `quality` level (0=full glow/effects, 1=medium, 2=minimal) to maintain ~60fps on slower devices. Shadows, bloom, and particle counts scale with quality.

### Input Priority

1. **Keyboard/Joystick:** Takes priority (prevents mouse conflict)
2. **Mouse:** Player glides toward pointer when hovering canvas
3. **Touch:** Full joystick on left, fire/bomb buttons on right (mobile auto-enabled)

## Key Implementation Details

### Collision Detection

`rectsOverlap(a, b)` (line 1069): AABB (axis-aligned bounding box) overlap check using half-widths.

### Particle System

Generic particles (line 1073): Position, velocity, life timer. Used for explosions, colorized per event type. Count scales with quality setting.

### LocalStorage Persistence

Four keys (lines 463-466):
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Respects system `prefers-reduced-motion` query by default

### Canvas Sizing

Base canvas is 480×720 (portrait). Scaled to fit window up to 1.6× (line 757), responsive to resize and orientation changes. Touch-action and user-select disabled to prevent mobile gestures interfering with gameplay.

### Audio Synthesis

`beep()` and `noiseBurst()` functions generate all SFX on-the-fly using oscillators, filters, and noise buffers. Panning applied per event (e.g., explosions pan to screen position). Master gain and dynamics compressor for balancing. No audio context until first user interaction.

### Reduced Motion Support

When enabled, animations are disabled via `animation: none` CSS, screen shake is minimized, and particle counts drop. Respects system preference and allows user toggle via button.

## Common Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update enemy spawn logic (line 1025-1030) if needed for frequency
3. Add explosion SFX variant in `EXPLODE_SFX` (line 678) mapping type → sfx function name
4. Implement sfx function (e.g., `explodeCustom`) in the `sfx` object (line 624)

### Adding a Boss Attack Pattern

1. Add pattern function to `BOSS_PATTERNS` array, indexed by boss design
2. Use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
3. Boss telegraph (flashing core) fires the pattern; timing controlled by `attackTimer` in `updateBoss()` (line 1411)

### Adjusting Difficulty

Edit `DIFFICULTIES` (line 485):
- `spawnBase` / `spawnMin`: Enemy spawn timing
- `enemySpeed`: Multiplier on enemy movement
- `enemyShootChance`: 0-1 probability enemy fires
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Modifying Visual Appearance

- **Sky themes:** Edit `THEMES` array (line 493) with sky gradient stops, cloud/sun colors, and theme name
- **Enemy planes:** Modify `drawPlane()` (line 913) for custom geometry
- **Boss ships:** Modify `drawBossShip()` (line 970) for hull/wing/core design
- **HUD colors:** Edit CSS color values in `<style>` (lines 10-358)

### Adding SFX

Create function in `sfx` object (line 624):
```javascript
customSound: () => {
  beep(freq, dur, 'sine', vol, { sweepTo, filterFreq, pan, delay });
  noiseBurst(dur, vol, { filterFreq, pan, delay });
}
```
Trigger with `sfx.customSound()`. Use `panFor(x)` for positional audio.

## Testing & Validation

- **No automated tests:** Manual play-testing is the standard approach
- **Browser DevTools:** Use Console to inspect `player`, `enemies`, `score`, etc., or call game functions directly
- **Performance:** Monitor frame rate (target 60fps); quality scales automatically on slower devices
- **Accessibility:** Test with reduced motion ON (via button), with keyboard/touch only, and screen reader announcements via `srLive`

## File Organization

Single file: `index.html` (entire game). All logic, styling, and game data defined inline. No external dependencies.

## Notes

- **Canvas context:** 2D only (no WebGL)
- **Audio:** Web Audio API synthesized (no files)
- **Storage:** LocalStorage only (no server backend)
- **Mobile:** Full touch support; auto-enables when `ontouchstart` detected
- **Responsive:** Scales to window, respects safe-area insets (notch support)
