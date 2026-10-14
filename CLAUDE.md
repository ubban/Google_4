# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. No build tools, bundlers, or external dependencies—just pure HTML, CSS, and JavaScript in `index.html`.

## Running the Game

**Browser:** Open `index.html` directly in any modern web browser (desktop or mobile). The game is fully responsive and supports touch controls on mobile devices.

**Development Workflow:** Since there are no build tools, development is direct editing:
1. Edit `index.html` in your editor
2. Refresh the browser (F5 or Cmd+R) to see changes
3. Use browser DevTools for debugging (F12)
4. localStorage persists high scores and settings across sessions

## Code Architecture

The entire game lives in one ~1400-line `index.html` file structured as:

### 1. **HTML Structure** (top of file)
- Container divs for game canvas, HUD, overlay screens, and touch controls
- Minimal semantic structure; most layout is CSS

### 2. **CSS Styling** (substantial block)
- Dark space theme using dark blues and accent colors (#1976d2, #ffd54f)
- Responsive design using `clamp()` for fluid scaling
- Safe-area support for notches/rounded corners via `env(safe-area-inset-*)`
- CSS animations for UI elements (card entrance, heart beat, vignette pulse, combo pop)

### 3. **JavaScript Game Code** (main logic, organized by system)

#### **Constants & Configuration**
- `BASE_W` / `BASE_H`: Game canvas dimensions (480×720)
- `STORAGE_*`: localStorage keys for persistence
- `ENEMY_TYPES`: Enemy definitions (grunt, interceptor, tank) with hp, speed, score, appearance
- `BOSSES`: Six boss definitions with hull colors and health multipliers
- Enemy spawn probabilities and stage progression rules

#### **Audio System**
- WebAudio API synthesized sound effects (no audio files)
- `beep()`: Oscillator-based tones with optional frequency sweep, filter, pan
- `noiseBurst()`: White noise with envelope, used for impact sounds
- `sfx` object: Named sound effects (shoot, laser, explode variants, bomb, hit, boss effects)
- Audio panning based on horizontal position for spatial audio
- Respects muted state and reduced-motion preference

#### **Rendering & UI**
- `renderHearts()` / `renderBombs()`: HUD stat updates
- Flash effects (hit/bomb) via CSS animation trigger
- Screen shake effect via canvas translation in the game loop
- Combo pop and stage banner animations
- Boss health bar with smooth width transitions

#### **Input Handling**
- Keyboard input: Arrow keys for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Touch input: Virtual joystick (circular pad with stick) on mobile
- Both controlled via `joyVec` (0–1 normalized direction vector)
- Joystick visual feedback with base and stick elements

#### **Game State Management**
- `running`, `paused`: Game flow flags
- `player`: Player object with x, y, hp, bombs, weapon type, powerup timer
- `entities`: Array of all game objects (enemies, projectiles, explosions, pickups)
- `stage`, `stageProgress`: Current level and spawn counter for progressive difficulty
- Difficulty modes: easy, normal, hard (affect enemy spawn rate and health)

#### **Game Loop**
- Single animation frame loop via `requestAnimationFrame()`
- **Update phase**: Input polling, entity updates (movement, aging, collisions), spawning
- **Render phase**: Clear canvas, draw sky gradient, render all entities, HUD
- Screen shake applied via canvas context transform
- Runs at ~60fps (browser refresh rate)

#### **Core Game Systems**

**Player Physics:**
- Movement: Bounded to canvas with pixel-perfect controls
- Weapon cooldown tracking for firing rate limits
- Bomb cooldown after use
- Powerup duration tracking (temporary weapon/health boosts)
- 3-heart health with game over on reaching 0

**Enemy System:**
- Entity-based: Each enemy is an object with type, hp, position, velocity
- Spawn logic: Different enemy types appear by stage; spawn rate increases with difficulty
- Movement: Linear downward by default; `weave` flag adds sine-wave horizontal motion
- Collision detection: Circular hitbox for simplicity (radius ~half width)
- Score award on death, displayed via floating text

**Weapon System:**
- `weapon` types: 'normal' (spread), 'laser' (concentrated), 'beam' (wide/slow)
- Projectiles are entities with lifespan; removed when off-canvas or aged out
- Weapon cooldown prevents spamming; adjusted per weapon type
- Powerups cycle through weapons on pickup

**Boss System:**
- Bosses spawn at stage milestones (stages 2, 4, 6)
- Boss object includes hull/core colors, health scaling, attack patterns
- Telegraph mechanic: Sound + visual warning before attacks
- Boss projectile patterns: Circular bursts (arc), aimed shots, spread patterns
- Health bar rendered with smooth animation
- Stage clear on boss defeat; drops high-value pickups

**Collision Detection:**
- Circular hitbox collision between projectiles/enemies and player
- Circle-circle distance check for simplicity (no pixel-perfect)
- Enemies killed by player projectiles; pickups collected on overlap

**Scoring & Progression:**
- Score from enemy kills (varies by type)
- Combo multiplier (x2–x5) based on rapid kills without taking hits
- Stage progression: More enemies, new types, bosses
- Difficulty scaling: Health multiplier per difficulty mode

## Important Implementation Details

### Canvas Sizing & Responsiveness
- Base canvas: 480×720 pixels (internal resolution)
- CSS-scaled via `canvas.style.width/height` to fit window while maintaining aspect ratio
- Max scale of 1.6× (capped so game doesn't overwhelm on large desktop)
- `resize()` called on window resize and orientation change

### Reduced Motion Support
- `reducedMotion` preference from localStorage or system `prefers-reduced-motion` media query
- Applied to all animations: screen shake magnitude reduced, duration capped
- Respects user accessibility preferences

### localStorage Persistence
- Best score
- Muted state
- Selected difficulty
- Reduced motion preference

### Touch Support
- Detects touch device via `ontouchstart` or `navigator.maxTouchPoints`
- Virtual joystick appears on mobile for movement and firing
- Joystick base hidden when inactive, appears on touch

## Development Tips

- **Editing the game:** Most tweaks are constants at the top of the JavaScript section. Enemy values, boss health, spawn probabilities, weapon cooldowns are all configurable.
- **Sound design:** The `sfx` object uses WebAudio synthesis. Modify frequency sweeps, filter cutoffs, and noise bursts to create new effects without audio files.
- **Visual feedback:** CSS animations in the style section handle UI. Canvas animations (shake, screen flash) are triggered from the game loop via element class toggles.
- **Adding enemies/bosses:** Add entries to `ENEMY_TYPES` or `BOSSES` objects; spawn logic will auto-detect new types via `minStage` field.
- **Debugging:** Use DevTools console to inspect `player`, `entities`, `stage`, or pause the game (P key) to inspect state mid-run.

## Browser Compatibility

- Requires Canvas 2D, WebAudio API, localStorage, and ES6+ (arrow functions, template literals, const/let)
- Tested on Chrome, Firefox, Safari, Edge (modern versions)
- Mobile: iOS Safari 13+, Android Chrome 80+
