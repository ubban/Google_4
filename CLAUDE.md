# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single, self-contained HTML5 file with embedded CSS and JavaScript (1669 lines total). The game features:

- Multiple difficulty levels (Easy, Normal, Hard)
- Weapon progression system with multiple weapon types
- Boss encounters with health bars
- Combo/multiplier system for scoring
- Visual themes with different color schemes
- Adaptive performance (frame-rate aware quality adjustment)
- Touch and keyboard controls
- Accessibility features (reduced motion, screen reader support)

## Running the Game

Simply open `index.html` in any modern browser. No build process, server, or dependencies required.

- **Desktop**: Open the file directly or serve locally with any HTTP server (`python -m http.server`)
- **Mobile**: Responsive design supports both touch and keyboard input
- **Testing**: Play through multiple difficulty levels, test boss fights, try different weapon types, verify touch controls on mobile devices

## Architecture Overview

The entire game runs in a single JavaScript IIFE (Immediately Invoked Function Expression) within the `<script>` tag starting at line 422. There are no external modules or build steps.

### Main Sections of the Game Logic

1. **Canvas & UI Setup** (lines 426–461): DOM element references, storage keys
2. **Performance Tracking** (lines 473–483): Adaptive quality system based on frame times
3. **Game Constants** (lines 485–533):
   - `DIFFICULTIES`: Spawn rates, enemy behavior, HP multipliers per difficulty level
   - `KILLS_PER_STAGE`: Enemies to defeat before boss appears
   - `THEMES`: 6 visual themes with sky gradients and color schemes
4. **Game State Object** (around line 534+): Central state store with player, enemies, score, etc.
5. **Entity Classes** (Player, Enemy, Projectile, etc.): Collision detection, rendering, behavior
6. **Game Loop** (via `requestAnimationFrame`): Update → Render cycle
7. **Input Handling**: Keyboard, mouse, and touch input listeners
8. **Rendering Pipeline**: Clear canvas, draw background/clouds, render entities, UI updates

### Key Game State Variables

- `player`: Object with `x, y, width, height, hp, maxHp, shootCooldown, moveState`
- `enemies`: Array of spawned enemy objects
- `projectiles`: Array of player and enemy projectiles
- `score`, `bestScore`: Persistent via localStorage
- `difficulty`: Current difficulty ("easy", "normal", "hard")
- `stage`: Current level (increments after boss defeat)
- `weapon`: Current weapon type and level
- `bombs`: Count of available smart bombs
- `multiplier`: Combo multiplier for scoring

### Collision Detection

Uses AABB (axis-aligned bounding box) collision with:
- `hitTest()`: General rectangle intersection
- Enemy-to-player, projectile-to-enemy, projectile-to-boss collisions checked each frame

### Rendering

**Performance Adaptive**: The `quality` variable (0–2) controls:
- Quality 0: Full render with all effects and animations
- Quality 1: Reduced particle effects
- Quality 2: Minimal effects (triggered if average frame time exceeds 1/40 seconds)

**Visual Effects**:
- Vignette (red pulse on player hit)
- Screen flash (white on enemy hit, brighter on bomb)
- Combo popup text
- Stage banner on level start
- Boss health bar (appears when boss spawns)

### Control Input

Three input methods coexist:
1. **Keyboard**: Arrow keys/WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
2. **Mouse**: Move cursor to fly player, click to fire
3. **Touch** (mobile): Drag left half of screen to move (joystick visualization), tap buttons for Fire/Bomb

Touch controls are shown/hidden based on touch device detection.

## Important Implementation Details

### Weapon System
Different weapons (VULCAN, LASER, SPREAD, HOMING) with:
- `shootCooldown` determines fire rate
- Projectile behavior (straight, spread, homing)
- Weapon upgrades increase damage and fire rate
- Weapon pickups drop from enemies

### Enemy Behavior
Enemies:
- Spawn based on difficulty-adjusted rates
- Move downward with slight variations
- Shoot at the player with difficulty-based accuracy
- Drop weapon pickups or bomb upgrades on death
- Deal damage on collision with player

### Boss Mechanics
- Appear after defeating `KILLS_PER_STAGE` regular enemies
- Have substantial health (scaled by difficulty and game progression)
- Use multi-phase attack patterns
- Display health bar while active
- Trigger stage transition on defeat

### Scoring & Multiplier
- Base score from enemy kills (modified by weapon type)
- Multiplier increases with consecutive hits without taking damage
- Resets on player hit
- Combo popup displays multiplier milestones

### Local Storage
Persists across sessions:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Development Notes

### Making Changes

**Single File**: All changes go in `index.html`. There are no separate CSS/JS files or build steps.

**Game Constants** (easy to tweak):
- Difficulty parameters: lines ~485–489
- Themes: lines ~493–499
- Spawn rates, enemy speed, shoot chances all in `DIFFICULTIES` object
- `KILLS_PER_STAGE` at line ~491

**Common Modifications**:
- To add a new weapon: Define behavior in the weapon class section and add to weapon type checks
- To adjust difficulty: Modify `DIFFICULTIES` object values
- To add a visual theme: Add entry to `THEMES` array with new sky/cloud/sun colors
- To change player stats: Look for `maxHp`, `speed` in the player initialization

### Performance Considerations

- **Frame rate tracking**: Disabled automatic quality downgrades by removing/commenting `trackPerf()` if consistent frame times aren't critical
- **Particle effects**: Reduce or disable cloud rendering for lower-end devices
- **Canvas size**: Responsive via CSS `max-width`/`max-height` but native resolution scales with window

### Accessibility

The game includes:
- Screen reader announcements (via `srLive` aria-live region)
- Reduced motion mode (disables animations when `prefers-reduced-motion` is set)
- High contrast colors for UI elements
- Keyboard-only playable (no mouse-only mechanics)

### Testing Touch Controls

Use browser DevTools (Chrome/Firefox):
1. Open DevTools (F12)
2. Toggle device toolbar (Ctrl+Shift+M)
3. Play on simulated mobile device to verify joystick and button responsiveness

## File Structure

```
/index.html           # Complete game (HTML + CSS + JS in one file)
/README.md            # Project title and timestamp
/.git/                # Version control
```

There are no additional configuration files, package.json, or build scripts.
