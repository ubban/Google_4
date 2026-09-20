# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (HTML, CSS, JavaScript) lives in `index.html` (~1670 lines).

The game features:
- Canvas 2D rendering with procedurally generated enemies, bosses, and themes
- Three difficulty levels (easy, normal, hard) with persistent settings in localStorage
- WebAudio synthesized sound effects (no external assets)
- Keyboard, mouse, and touch input (including joystick for mobile)
- Particle effects, screen shake, and visual feedback
- Multiple boss encounters with varied attack patterns
- Weapon and bomb power-ups with progression system

## Architecture

The game uses a traditional game loop pattern within a single IIFE (Immediately Invoked Function Expression):

### Core Game Objects

All game state is managed as JavaScript objects with position (x, y), dimensions (w, h), and type-specific properties:

- **player**: Tracked separately; includes weapon, level, invulnerability frames
- **enemies[]**: Array of enemy ships spawned with stage-dependent types
- **boss**: Single active boss object (null when absent); includes health and attack patterns
- **bullets[]**: Player projectiles (laser vs vulcan, pierce through multiple enemies)
- **enemyBullets[]**: Boss and enemy fire; radial bullets move via dx/dy, others via speed
- **powerUps[]**: Life, bomb, and weapon upgrades
- **particles[]**: Explosion and effect sprites for visual feedback

### Key Code Sections

1. **Constants & Config** (lines 463–524)
   - Storage keys for persistence
   - `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
   - Difficulty parameters affect spawn rate, enemy behavior, lives, bombs, boss HP

2. **Audio System** (lines 542–676)
   - `ensureAudio()`: Lazily creates and resumes AudioContext on first interaction
   - Synthesized sounds via `beep()` (oscillator + optional filter) and `noiseBurst()`
   - SFX library keyed by event type (shoot, explode, bomb, etc.)
   - Respects muted state and prefers-reduced-motion

3. **Canvas & Input** (lines 752–861)
   - Responsive canvas scaling via `resize()`
   - Three input methods:
     - **Keyboard**: Arrow keys or WASD for movement, Space/X for fire/bomb
     - **Mouse**: Move to steer, click to fire; priority over keyboard when active
     - **Touch**: Left side for joystick, right side for FIRE/BOMB buttons
   - All input triggers `ensureAudio()` to resume suspended AudioContext

4. **Game Loop** (lines 863–1651)
   - **`resetGame()`**: Initializes all arrays and player state from difficulty config
   - **`update(dt)`**: Main simulation (lines 1148–1302)
     - Player movement and weapon firing
     - Enemy spawning and behavior (weaving, shooting)
     - Collision detection (bullet–enemy, bullet–boss, enemy–player, powerup–player)
     - Chain/combo multiplier system (decays if kills >2.2s apart)
     - Boss spawns after 18 kills per stage; stage advances on boss defeat
   - **`draw()`**: Canvas rendering (lines 1520–1638)
     - Theme-based sky gradients, clouds, and sun glow
     - Ships drawn with `drawPlane()` and `drawBossShip()`
     - Particle effects faded by life remaining
     - Screen shake applied via canvas translation
   - **`loop(now)`** (lines 1640–1652): requestAnimationFrame loop; clamped to 50ms max delta

5. **Boss Logic** (lines 1372–1439)
   - `BOSS_PATTERNS`: Six distinct attack pattern sets (one per unique boss)
   - Each pattern is a function that fires `enemyBullets` with various geometries
   - Attack types: radial bursts, aimed spreads, columns, spirals, crosses, homing, rings
   - Boss has telegraph phase (visual telegraph on core) before firing

6. **UI & Overlays** (lines 428–461, 1479–1504)
   - Game-over screen shows final score, stage, and best score
   - Pause overlay (P or Esc)
   - Difficulty buttons active only when not running or paused
   - HUD updates in real-time (score, weapon, hearts, bombs)
   - Accessibility: Screen reader live region for announcements

### Performance Adaptation

Lines 473–483 track frame times and adjust `quality` (0=full, 1=medium, 2=low):
- Quality affects particle counts, glow effects (shadowBlur), and cloud density
- Auto-scales if avg frametime > 1/40 or < 1/56

### Collision Detection

`rectsOverlap(a, b)` (line 1069): AABB collision using distance-from-center approach.  
Works for all object types without requiring normalized sizes.

## Development Notes

### Single-File Structure

All game code is in one file. To make changes:
1. Find the relevant section by searching for a function or constant name
2. Most UI elements are grabbed by `getElementById` near the top; properties are set inline or via CSS classes
3. Game state arrays and objects are global to the IIFE; mutations happen in `update()`

### Adding Features

- **New boss**: Add to `BOSSES` constant, add pattern functions to `BOSS_PATTERNS` array
- **New enemy type**: Add to `ENEMY_TYPES` constant, update `spawnEnemy()` to include it
- **New weapon**: Add firing logic to `fireWeapon()`, add ammo rendering in `draw()`
- **New sound effect**: Define in `sfx` object using `beep()` or `noiseBurst()`, call in game logic
- **UI change**: Modify inline styles (lines 10–358) or grab elements and add/remove classes

### Debugging

- Check the browser's DevTools Console for errors
- `quality` variable (line 474) shows performance headroom; values >0 mean slow frames
- Collision checks happen in `update()` after position updates; visualize with canvas debug rects if needed
- Sound effects use pan based on position for stereo effect; check `panFor()` if spatial audio seems off

### Accessibility

- `announce()` (line 468) updates screen reader live region for game state changes
- Reduced-motion toggle respects `prefers-reduced-motion` media query and disables shake/flash animations
- All buttons have title attributes for keyboard navigation
