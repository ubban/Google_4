# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. It features:
- Canvas-based 2D graphics rendering
- WebAudio API for synthesized sound effects (no external audio files)
- Multi-stage gameplay with boss encounters
- Touch and keyboard input support with virtual joystick for mobile
- Adaptive difficulty levels (Easy, Normal, Hard)
- Local storage for high score and user preferences
- Reduced motion support for accessibility

## Architecture

### Single-File Structure
The entire game is contained in `index.html` (~1670 lines) with three sections:
1. **HTML markup** (~420 lines) — Canvas, UI overlays, HUD elements, controls
2. **CSS styles** (~350 lines) — Responsive layout, animations, theme colors
3. **JavaScript** (~900 lines) — Game engine, state management, rendering loop

### Key Game Modules

#### Game State & Constants
- **Difficulty modes**: Easy/Normal/Hard with configurable spawn rates, enemy speed, lives, and bombs
- **Enemy types**: Grunt, Interceptor (fast), Tank (armored) with different HP, speed, and scoring
- **Bosses**: 6 distinct boss types (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with unique visual designs and HP multipliers
- **Themes**: 6 visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) with theme-specific sky gradients
- **Weapon types**: Player can cycle through weapon levels (0-3) with different firing patterns

#### Core Game Loop
```
update(dt) → [check input, move entities, fire weapons, collision detection, spawn enemies/bosses]
render()   → [clear canvas, draw theme sky, draw entities, render HUD, apply visual effects]
requestAnimationFrame(gameLoop)
```

#### Key Functions

**Input & Controls:**
- `joystickStart/Move/End()` — Virtual joystick for touch input
- `canvasPointFromEvent(e)` — Convert mouse/touch to canvas coordinates
- Keyboard listeners for arrow keys, Z (fire), X (bomb), P (pause), difficulty keys

**Rendering:**
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Renders player or enemy aircraft
- `drawBossShip(b)` — Complex 6-part boss rendering (hull top/bottom, wings, core, exhaust)
- `spawnExplosion(x, y, color, count)` — Particle explosion effect

**Game Mechanics:**
- `fireWeapon()` — Player fires based on current weapon level
- `fireRadial(b, count, speed, offset)` — Boss fires bullets in circular pattern
- `fireAimedSpread(b, count, speed, spreadStep)` — Boss fires aimed spread pattern
- `useBomb()` — Clears bullets and damages enemies; visual vignette effect
- `applyPowerUp(type)` — Applies weapon upgrade, health, or bombs
- `rectsOverlap(a, b)` — AABB collision detection

**UI & Effects:**
- `flashHit/flashBomb()` — Screen flash animations
- `screenShake(mag, dur)` — Canvas offset shake effect
- `showComboPop(text)` — Floating combo text
- `showStageBanner(text)` — Stage introduction banner
- `updateMotionBtn()` — Toggle reduced motion accessibility setting

#### Storage Keys
- `skyfighter.bestScore` — High score persistence
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference

### Performance Optimization
- **Adaptive quality**: Tracks average frame time and reduces particle effects quality if FPS drops below 40
- **Quality levels**: Full (0), Medium (1), Low (2) — affects particle count in explosions
- **Canvas resize**: Optimizes canvas resolution based on device pixel ratio and viewport

## Common Development Tasks

### Testing the Game
Simply open `index.html` in a web browser. No build step required.
- Desktop: Arrow keys to move, Z to fire, X for bomb, P to pause
- Mobile: Virtual joystick for movement, tap Fire button, tap Bomb button
- Press 1/2/3 to set difficulty (Easy/Normal/Hard)

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (around line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` function to include new type in spawn pool logic
3. Consider adding `weave` property for sine-wave movement pattern

### Adding a New Boss
1. Add boss object to `BOSSES` array (line 508) with: `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul`
2. Update `spawnBoss()` to add spawn logic if needed
3. Modify `drawBossShip(b)` if boss needs custom rendering (currently all bosses share the same 6-part design)
4. Boss firing patterns are determined in `update()` — look for boss bullet firing logic around line 1200

### Adding a Visual Theme
1. Add theme object to `THEMES` array (line 493) with properties: `sky` (3-color gradient), `cloud`, `sun` (RGB values as strings), `name`
2. Update canvas rendering code to use theme colors when drawing background

### Changing Weapon Behavior
- Weapon firing patterns controlled by `fireWeapon()` function (~line 1124)
- Each weapon level (0-3) has its own firing pattern
- Modify bullet positions/counts/speeds in the switch statement based on `gameState.playerWeapon`

### Modifying Game Balance
- Difficulty settings: `DIFFICULTIES` object (line 485)
- Enemy stats: `ENEMY_TYPES` object (line 502)
- Boss multiplier: `hpMul` in `BOSSES` array
- Stage progression: `KILLS_PER_STAGE` constant (line 491)
- Scoring: Base score in `ENEMY_TYPES`, multiplier chain in `addScore()`

## Audio System
- WebAudio synthesis (no external files) using oscillators and noise
- `ensureAudio()` initializes and resumes audio context on user interaction
- `beep(freq, dur, type, vol, opts)` — Tone-based sounds (pitch, duration, waveform type, volume)
- `noiseBurst(dur, vol, opts)` — Noise-based sounds (e.g., explosions)
- `panFor(x)` — Spatial audio panning based on screen position
- All sound calls respect muted state and audio context availability

## Accessibility Features
- **Reduced motion support**: Toggle via 🚫/🎆 button; disables animations and particle effects
- **Screen reader support**: `#srLive` ARIA-live region announces game events
- **Keyboard controls**: Full game playable without mouse (arrow keys + Z/X)
- **Mobile touch**: Virtual joystick and large touch targets
- **High contrast**: Dark theme with readable text and clear visual feedback

## Browser Compatibility
- Requires HTML5 Canvas API
- Requires WebAudio API (audio degrades gracefully if unavailable)
- Requires localStorage for save data
- Responsive design works on desktop, tablet, and mobile (with safe area insets for notches)

## File Structure
```
Google_4/
├── index.html          (1669 lines: HTML + CSS + JS)
├── README.md          (Minimal project header)
├── CLAUDE.md          (This file)
└── .git/              (Version history)
```

## Key Metrics & Thresholds
- Canvas: Responsive, max-width/height with viewport scaling
- FPS target: 60 FPS with adaptive quality fallback to 40+ FPS
- Stage threshold: 18 enemies defeated per stage to advance
- Lives: 3 (Normal) or 4 (Easy), 2 bombs default
- Score multiplier chain: Resets on hit, increases on consecutive kills
- Boss HP calculation: Base HP × difficulty.bossHpMul × boss.hpMul
