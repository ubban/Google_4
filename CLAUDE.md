# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`. There is no build process, bundler, or external dependencies—the game runs directly in any modern browser.

## Code Structure

The `index.html` file has three main sections:

1. **HTML Head & Metadata** (lines 1-9)
   - Page title, favicon, and meta tags
   - Viewport configuration for mobile responsiveness

2. **CSS Styles** (lines 10-359)
   - Root styling and dark theme
   - Game canvas layout and styling
   - HUD (heads-up display) with score, health, weapon, and multiplier stats
   - Animations: combo popup pulses, stage banner reveals, boss health bar transitions
   - Mobile touch controls UI (joystick, fire/bomb buttons)
   - Overlay screens (start menu, game over, pause)
   - Reduced-motion support for accessibility (`@media prefers-reduced-motion`)
   - Effects: flash overlays (hit/bomb), vignette (damage pulse), button animations

3. **JavaScript Game Logic** (lines 422+)
   - Organized as a single IIFE (immediately invoked function expression)
   - Core systems:
     - **Canvas rendering**: 2D context for game drawing
     - **Game state**: difficulty, score, lives, weapons, enemies, bullets
     - **Input handling**: keyboard (arrows/WASD, space, X, P, Esc), mouse (move/click), touch (drag/tap)
     - **Difficulty system**: Easy/Normal/Hard with different spawn rates, enemy behavior, and boss HP multipliers
     - **Theme system**: 6 visual themes that cycle per stage (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
     - **Enemy AI**: spawn patterns, movement, shooting behavior based on difficulty
     - **Weapon system**: Vulcan (base), Spray (wide), Laser (fast), Thunder (piercing)—upgraded via power-ups
     - **Boss encounters**: Boss appears after `KILLS_PER_STAGE` (18) enemy kills; boss health bar shown while active
     - **Combo/Multiplier system**: score multiplier increases on consecutive kills
     - **Smart bombs**: invulnerability + area damage, limited per stage
     - **Collision detection**: player-enemy, bullet-enemy, bullet-boss, player-powerup
     - **Visual/audio feedback**: animations, sound effects (if audio context initialized), screen shake, flash overlays
     - **Storage**: localStorage for best score, difficulty, mute state, reduced motion preference
     - **Accessibility**: screen reader live region announcements, ARIA roles, keyboard navigation, reduced motion support
     - **Mobile support**: touch joystick on left side, fire/bomb buttons on right; auto-enables on touch input
     - **Pause & menu**: pause overlay, difficulty selection, game over/win screens

## Common Development Tasks

### Testing & Playing the Game

Since this is a single-file HTML game with no build step:

1. **Open in browser**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Test controls**:
   - **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bomb, P or Esc to pause
   - **Mouse**: Move mouse to steer (when using keyboard for fire), click to fire
   - **Touch** (mobile): Drag left side to move, tap FIRE button (red circle right side), tap BOMB button (yellow circle)
3. **Test features**:
   - Difficulty selection on start screen (Easy/Normal/Hard)
   - Sound toggle (🔊 button top-right)
   - Reduced motion toggle (🎆 button) for accessibility
   - Pause (⏸ button)
   - Weapon power-ups (Spray/Laser/Thunder) by destroying specific enemies
   - Boss encounters at stage progression
   - Combo multiplier on consecutive kills
   - Smart bomb (limited per stage)

### Making Changes

**Canvas rendering logic**: Lines ~500–1400 contain the main `render()` function and drawing utilities. Changes to visual appearance (colors, sizes, animations) go here.

**Game state & update logic**: The `update(dt)` function (lines ~1400–1700) handles game tick logic, collision detection, and state changes. Modify enemy behavior, weapon balance, or difficulty here.

**Input handling**: `setupInput()` (lines ~1800–2000+) binds keyboard, mouse, and touch events. Add new input methods or key bindings here.

**Difficulty/Theme constants**: Lines ~485–500 define `DIFFICULTIES` and `THEMES` objects. Tweak numbers here for balance changes (spawn rates, enemy speeds, lives, bomb counts).

**Game initialization & main loop**: `init()` and `gameLoop()` near the end of the script handle setup and frame updates. Frame time is tracked for adaptive quality in `trackPerf()`.

### Debugging Tips

- **Browser DevTools**: Open F12 (or Cmd+Option+I on Mac) to access the JavaScript console. The game logs minimal output; add `console.log()` statements in the update/render functions as needed.
- **Pause button**: Use the in-game pause (⏸ button or P key) to freeze the game and inspect state.
- **localStorage**: Game state is persisted in localStorage (keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Clear it in DevTools if you need a fresh start.
- **Screen reader announcements**: Text announced via `announce()` appears in a hidden live region (`#srLive`) for testing with screen readers.

## Key Constants & Systems

### Difficulty Levels
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```
- **spawnBase** / **spawnMin**: Frame count for enemy spawning (lower = more frequent)
- **enemySpeed**: Multiplier on enemy movement speed
- **enemyShootChance**: Probability per frame that an enemy fires (0–1)
- **lives** / **bombs**: Starting player lives and smart bombs per stage
- **bossHpMul**: Boss health multiplier

### Weapon Types
- **Vulcan** (default): Balanced fire rate and spread
- **Spray**: Wider spread, more bullets
- **Laser**: Narrow, fast, piercing shots
- **Thunder**: Slow but damages in an area; pierces enemies

### Visual Themes
6 themes cycle through stages, each with sky gradient, cloud color, and sun color. Affects only visuals, not gameplay.

### Combo System
Consecutive enemy kills without a miss increment `comboMult`. Resets to 1 if the player takes damage or a shot misses. Higher multiplier = higher score per kill.

### Boss Encounter
Appears after `KILLS_PER_STAGE` (18) regular enemy kills. Boss health bar shown at top while active. Boss defeated when HP ≤ 0; stage progresses.

## Performance Considerations

- **Adaptive quality**: Game tracks frame times and adjusts `quality` level (0=full, 1=medium, 2=low) to maintain ~40–56 FPS. Affects particle count and animation smoothness.
- **Canvas clearing**: Uses `fillRect()` with gradient background each frame for performance over `clearRect()`.
- **Collision grid**: Enemies/bullets checked against player position and each other in a simple loop (O(n²) at worst, but suitable for typical game scales).

## Accessibility Features

- **Keyboard navigation**: All game modes accessible via keyboard
- **Touch support**: Full mobile UI with on-screen joystick and buttons
- **Screen reader announcements**: Game events announced via `announce()` to a live region
- **Reduced motion**: Toggle (🎆 button) disables animations for users with motion sensitivity; respects `prefers-reduced-motion` media query
- **Color contrast**: UI text uses drop shadows and opacity layers to ensure readability on gradient backgrounds
- **Semantic HTML**: Buttons use `<button>` tags with titles and ARIA roles where relevant

## File Locations

- **Game canvas**: `#gameCanvas` (line 363)
- **HUD elements**: `#hud` and child `.stat` divs (lines 367–374)
- **Boss health bar**: `#bossBarWrap` (lines 376–379)
- **Overlay (menus)**: `#overlay` and `#overlayCard` (lines 401–419)
- **Touch controls**: `#touchControls` with `#moveZone`, `#joystickBase`, `#fireBtn`, `#bombBtn` (lines 392–399)
- **Buttons**: `#muteBtn`, `#motionBtn`, `#pauseBtn` (lines 384–388)
- **Visual effects**: `#vignette` (damage pulse), `#flash` (hit/bomb flash), `#comboPop` (combo animation), `#stageBanner` (stage intro) (lines 364–365, 381–382)

## Notes for Future Work

- The game runs at a variable frame rate controlled by `requestAnimationFrame`. Frame time (`dt`) is passed to `update()` for time-step-independent movement.
- Audio is stub-only (no actual sound playback) but can be expanded by initializing an `AudioContext` and adding Web Audio API calls.
- Mobile responsiveness is achieved via viewport meta tags and CSS `clamp()` functions for responsive font/size scaling.
- No external libraries—all game logic is vanilla JavaScript. Keeps the file self-contained and fast-loading.
