# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Sky Fighter

A single-file Raiden-style vertical shoot-'em-up game built with HTML5 Canvas, CSS, and vanilla JavaScript. No build tools or external dependencies.

## Quick Start

**Run the game:**
```bash
# Open index.html in a browser, or use Claude Code's run skill
```

The game runs entirely client-side. Just open `index.html` in any modern browser.

## Architecture

The entire game exists in a single `index.html` file (~1670 lines):

### Core Sections (in order of appearance):

1. **HTML/DOM** (lines 362-420)
   - Canvas element for game rendering
   - HUD overlays (score, lives, bombs, boss bar)
   - Touch controls (joystick and action buttons)
   - Main menu/pause overlay with difficulty selector

2. **CSS Styles** (lines 10-358)
   - Responsive canvas sizing with safe-area insets (mobile notch support)
   - HUD styling with glassmorphic UI (blur backdrop, semi-transparent backgrounds)
   - Animations: screen shake (CSS), combo popups, stage banners, UI transitions
   - Reduced motion support via `prefers-reduced-motion` media query

3. **Game Script** (lines 423-1667) - Main game logic organized as follows:

   **Initialization & Constants (lines 423-523)**
   - DOM element references
   - LocalStorage keys for persistence (best score, mute state, difficulty, reduced motion)
   - `DIFFICULTIES` object: spawn rates, enemy speed, lives, bomb count, boss HP multiplier for Easy/Normal/Hard
   - `THEMES` array: 6 sky gradients with color schemes
   - `ENEMY_TYPES`: grunt (weak, fast), interceptor (faster), tank (tanky, slow)
   - `BOSSES` array: 6 boss designs with distinct visuals and HP multipliers
   - `KILLS_PER_STAGE = 18`: enemy kills needed to trigger boss

   **Adaptive Performance (lines 473-483)**
   - Frame time tracking to scale quality (0=full, 1=medium, 2=low)
   - Lowers particle count and visual effects on slower devices

   **Audio System (lines 542-676)**
   - WebAudio context with compressor and master gain
   - `beep(freq, dur, type, vol, opts)`: synthesizes tonal sound effects
   - `noiseBurst(dur, vol, opts)`: generates noise-based effects
   - `panFor(x)`: converts screen X position to stereo pan value
   - `sfx` object: pre-defined sound effects (shoot, laser, explode, hit, bomb, powerup, boss sounds, etc.)
   - No external audio assets—everything is synthesized in real-time

   **UI & Rendering Helpers (lines 680-729)**
   - `renderHearts(count, max)`: renders life indicators
   - `renderBombs(count, max)`: renders bomb indicators
   - `flashHit()` / `flashBomb()`: screen flash effects (respects reduced motion)
   - `showComboPop(text)`: animated kill-count popup
   - `showStageBanner(text)`: stage transition banner
   - `screenShake(mag, dur)`: applies camera shake (can be reduced for accessibility)

   **Input Handling (lines 767-862)**
   - Keyboard: arrow keys + WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
   - Mouse: hovering/clicking to steer and fire (with crosshair indicator)
   - Touch: joystick on left side, FIRE/BOMB buttons on right (split-screen design)
   - Separate tracking for keyboard, mouse, and joystick inputs (joystick takes priority for movement)

   **Game State (lines 864-912)**
   - `resetGame()`: initializes/resets all state for a new game
   - State variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Game flags: `running`, `paused`, `gameOver`
   - Scoring: `score`, `multiplier`, `chain` (kill streak mechanic)
   - Progression: `stage`, `stageKills`, `kills` (for stage progression)

   **Drawing Functions (lines 913-1020)**
   - `drawPlane(x, y, w, h, color, darkColor, flame)`: draws player/enemy planes with shading
   - `drawBossShip(b)`: draws boss with pulsing core and rotating side cannons
   - `drawClouds(speed, fillColor, rx, ry)`: parallax cloud layers

   **Enemy & Boss Spawning (lines 1022-1056)**
   - `spawnEnemy()`: creates enemy with random type (grunt/interceptor/tank), pattern (spread/aimed), position
   - `spawnBoss()`: creates boss for current stage with procedural HP and telegraphic attack pattern
   - Difficulty affects enemy speed, shoot chance, and boss HP

   **Weapons & Fire Patterns (lines 1124-1409)**
   - `fireWeapon()`: fires vulcan (spray) or laser (piercing, screen-wide)
   - Player weapon levels affect spread pattern and fire rate
   - Boss attack patterns: radial bursts, homing missiles, aimed spreads, teleport attacks, cross-fire, spirals, etc.
   - Pattern functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
   - `BOSS_PATTERNS`: 6 difficulty levels with 3 attack patterns each

   **Main Game Loop (lines 1148-1302)**
   - `update(dt)`: called each frame to update game state
     - Player movement (keyboard > joystick > mouse priority)
     - Player firing and bomb mechanics
     - Bullet/enemy movement and cleanup
     - Collision detection (bullets vs enemies/boss, enemies/boss vs player, enemy bullets vs player, power-ups vs player)
     - Chain/multiplier decay
     - Spawn logic with difficulty ramp (spawning frequency increases as frame count grows)
   - `draw()`: renders entire frame
     - Sky gradient with parallax cloud layers
     - Player/enemies/boss/bullets/particles with optional glow effects
     - HUD handled separately via DOM

   **Boss Logic (lines 1411-1454)**
   - `updateBoss(dt)`: boss movement (bouncing off walls) and attack telegraphing
   - Boss has `telegraph` countdown (22 frames) before firing pattern
   - Boss AI defined by `BOSS_PATTERNS` indexed by boss design
   - `killBoss()`: handles boss defeat, stage progression, audio, score

   **Collision & Scoring (lines 1069-1122)**
   - `rectsOverlap(a, b)`: simple AABB collision detection
   - `spawnExplosion(x, y, color, count)`: particle burst with quality scaling
   - `addScore(base)`: adds score with multiplier and increments chain
   - `breakChain()`: resets multiplier to 1

   **Game Over Flow (lines 1456-1504)**
   - `loseLife()`: handles damage, invulnerability period, chain reset, life loss
   - `endGame()`: stops game, shows final score, updates best score in localStorage
   - `togglePause()`: shows/hides pause overlay

### Key Game Mechanics:

1. **Difficulty Scaling**
   - Wave spawn rate increases with frame count (natural difficulty ramp)
   - Enemy spawn rate, speed, shoot chance vary by difficulty
   - Boss HP multiplier by difficulty

2. **Weapon System**
   - Vulcan: spread pattern increases with level (1: 1 stream, 2: 2-wide, 3: 3-wide)
   - Laser: piercing shots with lane spread and increased count at higher levels
   - Weapon type and level persist through power-ups; switching weapon resets to level 1

3. **Scoring & Progression**
   - Kill chain mechanic: consecutive kills within ~2.2 seconds increase multiplier (x1 → x10)
   - Multiplier decays if no kills for 2+ seconds
   - Stage progresses after 18 enemy kills
   - Boss defeated = 500 base points + stage advance

4. **Accessibility**
   - Reduced motion toggle: disables animations (shake, flash, pulse) for users with motion sensitivity
   - Screen reader announcements via `srLive` ARIA live region
   - Respects OS `prefers-reduced-motion` setting

## Editing & Extending

When modifying the game:

- **Adding enemy types**: Add to `ENEMY_TYPES` object (requires `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`)
- **Adding bosses**: Add to `BOSSES` array and corresponding pattern to `BOSS_PATTERNS`
- **New themes**: Add to `THEMES` array with sky gradient and cloud colors
- **Sound effects**: Define in `sfx` object using `beep()` and `noiseBurst()` functions
- **Visual effects**: Modify drawing functions or add new particle types
- **Game balance**: Tweak `DIFFICULTIES` constants, `KILLS_PER_STAGE`, boss HP multipliers, or spawn timers

All data-driven values are defined as constants at the top of the script for easy tweaking.

## Browser Compatibility

- Requires: HTML5 Canvas, ES6+ JavaScript, WebAudio API
- Tested on: Chrome/Chromium, Firefox, Safari, Edge
- Mobile: iOS Safari, Android Chrome with touch controls
- Safe-area insets support for notched devices

## Performance Considerations

- Adaptive quality system reduces particle count on low FPS
- Canvas size fixed at 480×720, scaled via CSS for responsive display
- No external image/audio files: all graphics drawn procedurally, audio synthesized
- RequestAnimationFrame loop with delta-time capping at 50ms per frame
