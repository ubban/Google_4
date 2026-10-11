# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML game with no external dependencies. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 boss encounters with unique visual designs and attack patterns
- Weapon progression system with 5 weapon types
- Smart bomb mechanic with limited uses per life
- Visual themes with parallax scrolling clouds
- Web Audio API synthesized sound effects (no external audio files)
- Touch controls with virtual joystick for mobile/tablet play
- Accessibility features (reduced motion support, screen reader announcements)
- Responsive canvas scaling for various screen sizes

## Architecture Overview

The entire game lives in `/index.html` with:
- **HTML** (lines 1-420): Canvas element, HUD display, UI buttons, and controls layout
- **CSS** (lines ~10-380): Styling for game canvas, UI elements, animations, and responsive design
- **JavaScript** (lines 423-1669): Complete game logic in a single IIFE (Immediately Invoked Function Expression)

### Core Game Systems

**1. Game Loop & State Management**
- Global game state (score, lives, bombs, weapon level, multiplier, etc.)
- `update(dt)` — Main update function; handles player movement, enemy behavior, collision detection, and state transitions
- `draw()` — Renders all entities and HUD to canvas each frame
- Frame-rate independent updates using delta time (`dt`)

**2. Entities** (stored in global arrays: `player`, `enemies`, `projectiles`, `explosions`, `particles`, `boss`)
- Player ship with health, position, weapon state
- Enemy types: grunt, interceptor, tank (each with unique colors, speeds, HP, and behaviors)
- Boss ships with multi-phase attack patterns via `updateBoss(dt)` and `fireRadial`, `fireAimedSpread`, `fireColumns`
- Projectiles (player bullets and enemy shots)
- Visual effects: explosions, particle effects, screen shake

**3. Audio System** (Web Audio API, no external files)
- `ensureAudio()` — Lazy-initializes Web Audio context
- `beep(freq, dur, type, vol, opts)` — Synthesized tone with optional sweep, filter, and panning
- `noiseBurst(dur, vol, opts)` — Noise-based sounds for impacts and explosions
- Master gain and dynamics compressor for audio normalization
- Mute state persisted to localStorage

**4. Rendering**
- Canvas 2D context rendering at dynamic canvas size based on window dimensions
- `drawPlane()` — Draws player and enemy ships with colored hulls, wings, and directional flame effects
- `drawBossShip()` — Multi-part boss rendering with hull, wings, and glowing core
- `drawClouds()` — Parallax cloud backgrounds; called twice per frame at different speeds for depth
- Adaptive quality system that reduces particle counts and cloud density if frame rate drops

**5. Input System**
- Keyboard controls: Arrow keys for movement, Spacebar for fire, E for bomb
- Touch controls: Virtual joystick on left, fire/bomb buttons on right (mobile-specific)
- `joystickStart/Move/End()` — Track touch input and constrain joystick within a circular bounds
- `canvasPointFromEvent()` — Convert mouse/touch coordinates to canvas-relative positions

**6. Game Progression**
- Difficulty settings stored in localStorage: Easy (75 spawn base), Normal (58), Hard (40)
- Each stage requires 18 enemy kills to advance
- 6 boss encounters (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Boss HP scales by difficulty multiplier and unique boss multiplier
- Visual themes cycle through 6 different sky gradients and cloud colors

**7. Weapon & Power-up System**
- 5 weapon types: VULCAN (spread), VULCAN+ (wider spread), PULSE (single), BEAM (sustained), BLASTER (wide spread)
- Weapon levels increase bullet spread/damage; levels reset on boss defeat
- Power-up types: health restore, weapon upgrade, score multiplier, bomb restore
- `applyPowerUp()` — Updates game state based on dropped power-up type

**8. Scoring & Chains**
- Base score per kill multiplied by current combo multiplier
- Multiplier increases for consecutive kills without taking damage; resets on hit
- `addScore(base)` — Adds score and updates best score in localStorage
- Combo displays via `showComboPop()` with visual feedback

**9. Visual Effects & Feedback**
- `screenShake(mag, dur)` — Camera shake effect (disabled if reducedMotion is true)
- `flashHit/flashBomb()` — Screen flash on damage or bomb use (respects reducedMotion)
- `spawnExplosion()` — Creates particle effects at coordinates
- Vignette effect (red pulse) when boss is near
- Stage banners and combo popups via DOM elements

**10. Accessibility & Performance**
- Screen reader announcements via `announce()` for game state changes
- Reduced motion preference detection and user override
- Adaptive performance tracking: monitors frame times and reduces quality (particle count, cloud detail) if FPS drops
- Storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Difficulty Constants

Located at lines 485-515:
- `DIFFICULTIES` — Object defining spawn rates, enemy speed, enemy shoot chance, lives, bombs, boss HP multiplier for each difficulty
- `KILLS_PER_STAGE` — Number of enemy kills needed to progress (set to 18)
- `ENEMY_TYPES` — Defines visual and behavioral properties (width, height, HP, speed, color, score, min stage)
- `BOSSES` — 6 boss ship templates with unique hull colors, wing colors, core colors, and HP multipliers
- `THEMES` — 6 visual themes with gradient colors for sky and clouds

## Development Workflow

### Building & Testing

Since this is a single HTML file with no build step:
1. Open `/index.html` in a modern web browser (Chrome, Firefox, Safari, Edge all supported)
2. Game loads immediately; no server required
3. Changes to HTML/CSS/JS are live — refresh the page to test

### Testing a Specific Feature

**To test difficulty balance:**
- Click difficulty buttons on start screen; difficulty is saved to localStorage
- Check spawn rates and enemy behavior in `update()` function (around line 1200)

**To test weapons/power-ups:**
- Add quick power-up spawn in-game: modify `maybeSpawnPowerUp()` to always spawn
- Test each weapon type by checking bullet patterns in `fireWeapon()` (line 1124)

**To test boss patterns:**
- Use browser DevTools Console: `quality = 0` (set to lowest quality), `boss = {hp: 1}` (instant kill), then start game
- Alternatively, modify `spawnBoss()` to override boss HP: set `b.hpMax = 10` for rapid boss cycling

**To test audio:**
- Toggle mute with the speaker icon; verify Web Audio context initializes in DevTools Console: `audioCtx !== null`
- Check `beep()` and `noiseBurst()` functions for parameter tweaks (frequency, duration, volume)

**To test input/controls:**
- Use keyboard: Arrow keys, Spacebar (fire), E (bomb)
- On mobile/touch: Swipe joystick on left side; tap fire/bomb buttons on right side
- For joystick testing: verify `joystickBase` and `joystickStick` elements move correctly in DevTools

### Common Edits & Their Locations

| Change | File | Location |
|--------|------|----------|
| Adjust spawn rate | index.html | Lines 485-489 (DIFFICULTIES object) |
| Modify enemy colors | index.html | Lines 502-506 (ENEMY_TYPES object) |
| Tweak boss patterns | index.html | Lines 1411-1506 (updateBoss function) |
| Change weapon behavior | index.html | Lines 1124-1147 (fireWeapon function) |
| Adjust visual theme | index.html | Lines 493-500 (THEMES object) |
| Modify audio parameters | index.html | Lines 566-595 (beep function) |
| Change particle effects | index.html | Lines 1073-1086 (spawnExplosion function) |
| Update screen shake | index.html | Lines 727-730 (screenShake function) |

### Performance Debugging

The game includes adaptive quality tracking:
- Frame times are sampled in `trackPerf(dt)` (lines 476-483)
- Quality level (0=full, 1=medium, 2=low) adjusts particle counts and cloud rendering
- Check `quality` variable in DevTools Console during gameplay to monitor adaptive scaling

## Key Implementation Notes

1. **Single IIFE Scope** — All game code is wrapped in `(function() { ... })()` to avoid global namespace pollution; all state is scoped inside

2. **Canvas Resizing** — The `resize()` function (line 754) handles window resizing; canvas maintains 16:9 aspect ratio with max dimensions

3. **Delta Time** — All movement and physics use `dt` (delta time in seconds) for frame-rate independence; see `update()` function

4. **Collision Detection** — `rectsOverlap()` (line 1069) uses AABB (axis-aligned bounding box) for simple fast checks

5. **Boss State Machine** — Each boss has a multi-stage attack pattern defined in `updateBoss()` with counters for timing attacks

6. **localStorage Persistence** — Best score, mute state, difficulty, and reduced motion preference are all saved; keys are defined at lines 463-466

7. **Web Audio Context** — Audio context is created on first use (lazy-initialized in `ensureAudio()`) due to browser autoplay policies

8. **Touch/Joystick Handling** — Virtual joystick is constrained to a circular region; touch events are normalized to canvas coordinates via `canvasPointFromEvent()`

## Git Development Branch

Development should be done on the feature branch: **`claude/inspiring-planck-igrvql`**

Commit messages should follow the format:
```
<description>

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01EywhmGRfsDopAaKP9iW32Z
```
