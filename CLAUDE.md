# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single HTML file (`index.html`) with embedded CSS styling and JavaScript game logic. No build process or external assets are required—the game runs directly in any modern web browser.

## Running and Testing

- **Open in browser**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **Local testing**: Use a local server if needed: `python -m http.server 8000` or `npx serve`
- **Mobile/touch support**: Full touch controls work on iOS/Android with joystick input
- **Accessibility**: Supports screen readers and reduced motion preferences

## Architecture Overview

The game is built as a single monolithic file (~1670 lines) with these main sections:

1. **DOM/UI Setup** (lines 1-421): HTML structure, CSS animations, HUD elements
2. **Game Logic** (lines 422+): IIFE-wrapped JavaScript containing all game state and systems

### Key Game Systems

**Game State** - Variables tracking:
- Player position, health, weapons, bomb count
- Active enemies and bullets (player and enemy)
- Score, combo multiplier, current stage/difficulty
- Current game running state and pause status

**Game Loop** - Main update/render cycle:
- `tick()` function handles all state updates (player movement, enemy spawning, collision detection, bullet updates)
- `draw()` function renders canvas, HUD, and UI elements
- Runs via `requestAnimationFrame` for smooth 60fps animation
- Adaptive quality system (3 levels) that degrades rendering based on frame time to maintain performance

**Rendering System** - Canvas-based 2D graphics:
- Dynamic theme system with gradient skies (6 different environments)
- Procedurally animated clouds
- Drawing functions for player plane, enemies (3 types), bosses (6 types), bullets, effects
- Visual feedback: screen shake, flash effects, vignette on damage, combo popups
- Responsive sizing: all dimensions scale with viewport using CSS `clamp()`

**Audio System** - Web Audio API synthesis (no external audio files):
- Procedural sound generation via `beep()` (frequency sweep oscillators) and `noiseBurst()` (white noise)
- Contextual sfx object groups effects by category (uiClick, shoot, hitEnemy, bossDamage, etc.)
- Master gain and dynamics compressor for consistent loudness
- Spatial panning based on screen position
- Can be muted via UI toggle; preference persisted to localStorage

**Input Handling**:
- Keyboard: WASD/Arrow keys for movement, Space/E/Ctrl for fire/bomb
- Mouse: Click to fire, mouse position for targeting
- Touch: Virtual joystick (left) and fire/bomb buttons (right)
- Detects input method and switches UI hints accordingly

**Enemy/Boss System**:
- Enemy spawning based on difficulty (randomized timing from configurable ranges)
- Three enemy types: Grunt, Interceptor, Tank (each with HP, speed, score, appearance)
- Six boss types appearing at stage transitions, with unique visual designs and scaling HP
- Enemy weaving/movement patterns (interceptors weave side-to-side)
- Enemy shooting and despawn logic

**Collision/Damage System**:
- AABB collision detection for bullets vs entities
- Damage application with temporary invulnerability frames
- Screen edge wrapping for player
- Game over on 0 health, win condition on all kills destroyed in a stage

**UI/HUD System**:
- Real-time score, best score, weapon/level, heart/bomb displays
- Boss health bar (appears when boss active)
- Screen reader announcements via `announce()` helper
- Overlay screens for title, game over, pause menu
- Difficulty selector with visual active state

**Storage/Preferences** (via localStorage):
- Best score
- Muted state
- Difficulty selection
- Reduced motion preference

## Key Constants and Configuration

These are defined early in the script and control game behavior:

**Difficulty Parameters** (line ~485):
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
}
```
Each difficulty configures spawn rates, enemy speed multiplier, shoot chance, starting lives/bombs, and boss HP multiplier.

**Enemy Types** (line ~502):
```javascript
ENEMY_TYPES = {
  grunt: { w: 34, h: 34, hp: 1, speedMul: 1, color: '#e53935', score: 10, ... },
  interceptor: { ... },
  tank: { ... }
}
```
Each type defines visual appearance, HP, speed, point value, and min stage where it appears.

**Bosses** (line ~508):
Six boss definitions with unique hull colors, core colors, and HP multipliers. Stage cycles through them.

**Themes** (line ~493):
Six visual themes with sky gradients, cloud colors, and sun colors. Stage determines which theme.

**Progression Constants**:
- `KILLS_PER_STAGE = 18`: Number of enemies per stage
- `STORAGE_*`: Key names for localStorage persistence

## Common Customization Points

**Difficulty Tweaking**: Modify `DIFFICULTIES` object properties:
- `spawnBase`/`spawnMin`: Adjust enemy spawn rate (lower = more frequent)
- `enemySpeed`: Multiplier on all enemy movement speed
- `enemyShootChance`: Probability (0-1) enemies fire per frame
- `bossHpMul`: Difficulty multiplier on boss health

**Adding/Modifying Enemies**: Add new entry to `ENEMY_TYPES`:
```javascript
myNewType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ff6b6b', dark: '#c92a2a', score: 25, minStage: 3 }
```
Then update enemy spawning logic to randomly pick this type when appropriate.

**Adding/Modifying Bosses**: Add new entry to `BOSSES` array:
```javascript
{ name: 'CUSTOM', hullTop: '#abc123', hullBottom: '#def456', wing: '#def456', core: '#fff000', hpMul: 1.4 }
```

**Adding Visual Themes**: Add to `THEMES` array with new gradient stops and cloud colors:
```javascript
{ sky: ['#...', '#...', '#...'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```

**Weapon Progression**: Weapon upgrades (burst→laser) happen via `buyWeapon()` function. Costs and damage values are hardcoded; search for `LASER_COST` or `BURST_COST` to adjust.

**Screen Effects**: Screen shake, flash, and vignette intensity controlled by `shakeMag`, `shakeTime`, and animation CSS values in the `<style>` section.

## Performance Considerations

- **Adaptive Quality System**: The game automatically reduces rendering quality (shadow/glow effects) if average frame time exceeds 1/40s, restoring quality when performance improves
- **Canvas Size**: Scales to viewport via CSS; performance sensitive on large screens with many enemies
- **Audio Context**: Lazily initialized on first sound; runs synthesis in real-time on demand
- **Collision Detection**: O(n²) AABB checks each frame; manageable because enemy/bullet counts stay reasonable even on hard difficulty

## Code Style Notes

- Single IIFE wraps all game logic to avoid global namespace pollution
- Heavy use of object literals for configuration (difficulties, enemy types, bosses, themes)
- Functional approach for sound synthesis and UI callbacks
- No external dependencies; pure vanilla JavaScript and Canvas API
