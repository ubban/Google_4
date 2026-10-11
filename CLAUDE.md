# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-em-up game built entirely in a single HTML file. It features:
- Classic arcade-style vertical scrolling gameplay
- Multiple enemy types with varying difficulty and behavior
- 6 procedurally-drawn boss encounters
- 2 weapon types (Vulcan bullets and Laser)
- Smart bomb power-ups for clearing enemies
- 3 difficulty levels (Easy, Normal, Hard)
- 6 visual themes for stages
- Procedural audio using Web Audio API (no external audio files)
- Full keyboard/mouse and touch/joystick controls
- Responsive scaling to any device
- Persistent high score tracking

## File Structure

**index.html** — The complete game implementation (~1,670 lines)
- **Lines 1-359**: HTML markup and CSS styling
  - Canvas-based rendering
  - UI overlays (start screen, game over screen)
  - Touch control elements (joystick, fire/bomb buttons)
  - HUD elements (score, weapon, lives, bombs)
- **Lines 422-end**: JavaScript game engine (IIFE pattern)

## JavaScript Architecture

The entire game engine is wrapped in a single IIFE (Immediately Invoked Function Expression) for scope isolation. The logic is organized into these sections:

### 1. DOM References & Configuration (Lines 426-527)
- Canvas and UI element caching
- Game configuration constants:
  - `DIFFICULTIES`: 3 difficulty presets with spawn rates, enemy speed, lives, bombs
  - `ENEMY_TYPES`: grunt, interceptor, tank with HP, speed, score values
  - `BOSSES`: 6 boss variants with unique visuals (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
  - `THEMES`: 6 visual themes with sky gradients and cloud colors
  - `KILLS_PER_STAGE`: Enemy count needed to trigger boss fight (18)
- Storage keys for persistence (best score, mute state, difficulty, reduced motion preference)

### 2. Audio System (Lines 542-676)
**Web Audio API synthesizer** — generates all sounds procedurally:
- `ensureAudio()`: Lazy-initializes audio context with compressor
- `beep()`: Oscillator-based tone generation with envelope, sweep, filter, pan
- `noiseBurst()`: White noise generator with envelope and filter
- `sfx` object: Named sound effects (shoot, laser, explode, bomb, powerup, etc.)
- `panFor()`: Converts screen X position to stereo pan value

### 3. Performance Adaptation (Lines 473-483)
- Tracks last 45 frame times in `frameTimes` array
- Adjusts `quality` level (0=full, 1=medium, 2=low) based on 40-56 FPS thresholds
- Affects rendering (glow effects) and visual polish, never gameplay logic

### 4. Drawing & Rendering (Lines 913-1606)
- `drawPlane()`: Renders player/enemy aircraft with body, wings, flame
- `drawBossShip()`: Draws procedural boss with unique hull colors and animated patterns
- `drawClouds()`: Procedural parallax scrolling clouds
- `draw()`: Main render function that clears canvas, draws all entities with adaptive quality

### 5. Game Loop & Update Logic (Lines 1148-1506)
- `update(dt)`: Core game tick
  - Player movement from keyboard/joystick/mouse input
  - Player shooting with cooldown and weapon switching
  - Enemy spawning, movement, and AI (random weave patterns)
  - Collision detection (player/enemies, bullets/enemies, power-ups)
  - Particle effects and cleanup
  - Combo chain decay
  - Boss phase transitions
- `updateBoss(dt)`: Boss-specific logic
  - Boss telegraph pattern (warning before attacks)
  - Boss bullet patterns and movement
  - Boss health tracking and defeat

### 6. Entity Management
**Game State Variables** (Line 864):
- `player`: Position, velocity, weapon, level, cooldown, invulnerability timer
- `bullets`: Player projectiles with type (vulcan/laser)
- `enemies`: Array with type, HP, spawn stage constraints
- `enemyBullets`: Enemy projectiles (circles)
- `powerUps`: Weapon and bomb pickups with drop chance
- `boss`: Current boss with health, pattern state, visual state
- `particles`: Debris and effects for cleanup

### 7. Input Handling (Lines 767-862)
**Keyboard/Mouse**:
- Arrow keys or WASD for movement
- Space or click to fire
- X or Shift for bombs
- P/Escape to pause
- Mouse steering: plane glides toward cursor when hovering canvas

**Touch/Mobile**:
- Left zone: Joystick for movement
- Right zone: FIRE and BOMB buttons
- Visual feedback with joystick base and stick elements

### 8. Game Flow
- **resetGame()**: Initialize all entities, state, and difficulty settings
- **Start**: Display overlay with difficulty select, wait for button
- **Playing**: update() → draw() loop at 60 FPS
- **Stage progression**: Kill 18 enemies → boss spawns
- **Boss defeat**: Next stage, spawn new enemies
- **Pause**: Game state frozen, can resume or restart
- **Game Over**: Show final score, best score check, restart option

### 9. Scoring & Progression
- Each enemy type grants fixed score (grunt=10, interceptor=15, tank=35)
- **Combo chain**: Consecutive kills without gaps increment `multiplier` (1-4x)
- **Weapon progression**: Vulcan → Laser on power-up pickup, weapon level increases
- **Best score**: Persistent storage in localStorage, shown on game over

### 10. Difficulty Scaling
Each difficulty (easy/normal/hard) affects:
- Enemy spawn rate (higher = more frequent)
- Enemy movement speed multiplier
- Enemy shoot chance
- Starting lives and bombs
- Boss HP multiplier
- Selected at start, persisted in localStorage

## Key Implementation Details

### Collision Detection
- AABB (Axis-Aligned Bounding Box) used throughout
- Player/enemy collision: circular approximation with width/height
- Bullet/enemy: projectile rect vs enemy rect
- Uses `Math.hypot()` for distance calculations in joystick logic

### Enemy Spawning
- Enemies spawn at random top-screen positions
- Type selection filters by `minStage` to gate enemy varieties
- Interceptors weave horizontally while falling
- Tank enemies spawn at stage 2+

### Boss AI
- Boss enters after killing `KILLS_PER_STAGE` enemies
- Telegraph phase: 0.8s warning with sound before attack pattern
- Bullet patterns: circular or targeted spreads
- Boss moves down slowly while firing
- Health displayed in top bar with color lerp (red to green)

### Power-ups
- 10% drop chance per enemy kill
- Type 'weapon': Toggles between Vulcan/Laser, increments weapon level
- Type 'bomb': Grants additional bomb (up to max)
- Visible as small rotating squares

### Bombs (Smart Bombs)
- Clears all enemies and enemy bullets on screen
- Explosion sound and visual flash
- Cooldown is instant, no rapid re-use
- Limited supply (2-3 depending on difficulty)

### Visual Polish
- Vignette edge darkening (activated when 1 life remaining)
- Screen flash on hits
- Glow effects on bullets (quality-dependent)
- Boss health bar with color gradient
- Combo pop-up text animation
- Stage clear banner animation
- Reduced motion support (disables animations if system prefers)

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats (w, h, hp, speedMul, color, dark, score, minStage)
2. Add explosion sound variant to `sfx` if needed (e.g., `explodeNewType`)
3. Update `EXPLODE_SFX` mapping if custom sound
4. Adjust enemy spawn logic if needed (e.g., weave behavior)
5. Test at each difficulty level, especially minStage gating

### Add a New Boss
1. Add entry to `BOSSES` array with name, hull/wing/core colors, hpMul
2. Create unique bullet pattern in `updateBoss()` based on boss name
3. Adjust telegraph timing if needed
4. Test health scaling across difficulties

### Modify Difficulty Scaling
1. Edit `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)
2. Test spawn rates and progression pacing
3. Verify player has reasonable survival chance

### Add New Sound Effect
1. Create new function in `sfx` object using `beep()` and/or `noiseBurst()`
2. Call from relevant game event (e.g., `sfx.powerup()` when picking up)
3. Handle muted state (functions check `muted` before synthesis)

### Adjust Game Balance
- **Difficulty**: Modify spawn rates, enemy speed, lives
- **Weapon**: Change Vulcan/Laser fire delay or damage behavior
- **Player speed**: Modify `player.speed` in `resetGame()`
- **Collision tolerance**: Adjust entity width/height values for hitbox sizes

## Testing & Debugging

### Run the Game
- Open `index.html` in any modern browser
- Game runs immediately, no build step or dependencies
- Check browser console for any errors

### Performance Profiling
- Open DevTools, run game, watch frame times
- Quality level auto-adjusts based on FPS; check `quality` variable in console
- Disable glow effects (quality 2) on very slow devices

### Touch Testing
- Test on actual mobile devices or use browser device emulation
- Joystick and buttons appear automatically on touch devices
- Verify joystick tracking and button responsiveness

### Audio Testing
- Test sound effects in all scenarios (hits, explosions, power-ups, boss fights)
- Verify mute button silences all audio
- Check panning (left/right stereo positioning of explosions)
- AudioContext may require user interaction before first sound; `ensureAudio()` handles this

## Browser Compatibility

- **Canvas 2D**: All modern browsers
- **Web Audio API**: Chrome, Firefox, Safari, Edge
- **Touch Events**: iOS Safari, Android Chrome
- **LocalStorage**: All modern browsers for persistence
- **Viewport units (clamp, min, max)**: Modern browsers; consider fallbacks for older devices

## Performance Considerations

- Single canvas draw per frame (~60 FPS target)
- No external assets; all graphics are procedural
- Procedural audio synthesis is CPU-intensive; reduce quality if muted to save cycles
- Entity arrays cleaned up after each frame (dead enemies/bullets/particles removed)
- Particle count is capped implicitly by enemy defeat rate
- Large frame time variance may trigger quality reduction

## Notes for Future Development

- The entire game is self-contained in one file for portability; splitting is possible but not necessary
- Add `data-*` attributes to buttons for easier testing (e.g., `data-test="fireBtn"`)
- Consider adding accessibility improvements: currently has screen reader announcements (srLive element)
- Mobile viewport settings are optimized; test scaling on various device pixel ratios
- Boss patterns can be extended with more variety per boss instance (currently deterministic per boss name)
- Weapon progression could expand to 3+ weapon types; currently hard-coded to Vulcan/Laser toggle
