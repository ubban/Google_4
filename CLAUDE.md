# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. No build process or dependencies are required—the game runs entirely in the browser with vanilla JavaScript, Web Audio API, and Canvas rendering.

## Quick Start

**To view the game:**
- Open `index.html` directly in a modern web browser
- No server or build step needed

**Browser support:**
- Chrome/Edge 90+, Firefox 88+, Safari 14+
- Desktop (mouse/keyboard) and mobile (touch controls) input

## Code Architecture

The entire game is in `index.html` (~1700 lines), organized into these logical sections:

### Core Game State (line ~864)
```
player - Ship position, health, weapon, cooldown
bullets - Player projectiles
enemies - Enemy ships in current wave
enemyBullets - Enemy projectiles
boss - Current boss object (when active)
particles - Visual effects (explosions, hits)
powerUps - Dropped items (health, bomb, weapon upgrades)
score, lives, stage, difficulty - Game progress
```

### Main Game Loop (line ~1520+)
- **`update(dt)`** (line ~1148) - Handles input, physics, collisions, game logic each frame
- **`draw()`** (line ~1520) - Renders all game elements to canvas
- **`requestAnimationFrame`** - Triggers loop at 60 FPS

### Key Systems

**Input Handling** (line ~768-860)
- Keyboard: Arrow keys / WASD for movement, Space to fire, Shift/X for bomb
- Mouse: Hover to steer, click to fire
- Touch: On-screen joystick and fire/bomb buttons
- Joystick vector (`joyVec`) normalized to [-1, 1] range

**Weapon System** (line ~1000+)
- Two weapon types: "vulcan" (rapid fire) and "laser" (single shot, high damage)
- Fire cooldown: 0.15s for vulcan, 0.1s for laser
- Weapon upgrades spawn as power-ups and increase weapon level

**Enemy Waves** (line ~1197-1400)
- Waves spawn sequentially until 18 kills reached
- Boss appears after each stage (every 18 kills)
- Enemy types: basic shooter, kamikaze, spreader patterns
- Enemies drop power-ups on death (health, bombs, weapon upgrades)

**Boss System** (line ~1411-1505)
- `boss` object has position, health, pattern
- `updateBoss(dt)` handles boss-specific bullet patterns and movement
- Boss health bar shown at top when boss active
- Boss defeat triggers stage clear animation and next wave

**Audio** (line ~545-620)
- `beep(freq, dur, type, vol, opts)` - Generates square/sine beeps
- `noiseBurst(dur, vol, opts)` - White noise for impacts
- Audio context created on first user interaction (security requirement)
- Mute state stored in localStorage

**Collision Detection** (line ~1240-1350)
- Rectangle-based hitboxes for player, enemies, bullets
- Collision checks for: enemy bullets vs player, player bullets vs enemies, power-ups vs player
- Chain multiplier system (resets after 1.2s without killing)

**Rendering** (line ~1520-1600)
- Canvas scaled to fit viewport (960×640 base, responsive)
- Background gradient with parallax cloud layers
- Plane drawing via `drawPlane()` with directional flame
- Boss drawing via `drawBossShip()` with animated patterns
- HUD displays score, lives, weapons, combo multiplier

### Constants & Configuration

**Difficulty Levels** (line ~485)
```
Easy: slower enemies, less bullets, lower difficulty scaling
Normal: balanced (default)
Hard: faster spawn, more projectiles, harder patterns
```

**Enemy Types** (line ~502+)
- Small: basic slow projectiles
- Medium: faster, spreads fire pattern
- Large: slow heavy hitter, tanky

**Theme Variations** (line ~493)
- 5 color themes for canvas gradient background, applied per stage

**Storage Keys** (line ~463)
```
skyfighter.bestScore - High score
skyfighter.muted - Audio mute state
skyfighter.difficulty - Last selected difficulty
skyfighter.reducedMotion - Accessibility flag
```

## Development Workflow

### Making Changes

**For game balance tweaks:**
- Enemy spawn rate: `KILLS_PER_STAGE` constant (line 491)
- Difficulty multipliers: `DIFFICULTIES` object (line 485)
- Weapon stats: `fireDelay` values (line 1177)
- Enemy bullet patterns: `updateBoss()` function (line 1411)

**For new features:**
1. Add game state variables near line 864
2. Update input handling if new controls needed (line 768-860)
3. Update `update()` for game logic
4. Update `draw()` for rendering
5. Add audio cues via `beep()` or `noiseBurst()`

**For bug fixes:**
- Most issues are in collision detection (~line 1240), input handling (~line 768), or render logic (~line 1520)
- Test on both desktop and mobile (touch device detection at line 778)
- Check browser console for errors

### Testing

**Quick manual testing:**
1. Open index.html in browser
2. Play through at least one full stage to verify:
   - Movement/aiming feels responsive
   - Enemies spawn and behave correctly
   - Collisions register properly
   - Audio plays (if volume is on)
   - HUD updates correctly
3. Test each difficulty level
4. Test on mobile if changes affect input/rendering

**Automated testing:**
- No test suite currently; consider adding one for collision logic if expanding complexity

**Performance:**
- Game tracks FPS via `trackPerf()` (line 476)
- Check browser DevTools Performance tab if seeing stutters
- Canvas rendering is typically the bottleneck—reduce particle effects if needed

## Browser & Environment Notes

- **Canvas size:** 960×640 (base), scales responsively via CSS
- **Target FPS:** 60 (requestAnimationFrame)
- **Mobile support:** Full—touch controls, safe-area-inset for notches
- **Accessibility:** Screen reader live region, reduced-motion support, color contrast on HUD
- **Storage:** localStorage for high score, settings (no IndexedDB needed)

## References in Code

- Player speed: 320 px/s (adjustable in `update()`)
- Bullet speeds: 400-600 px/s depending on type
- Enemy spawn delay: varies by difficulty, increases over time
- Chain multiplier max: 8x (resets if 1.2s without kill)
