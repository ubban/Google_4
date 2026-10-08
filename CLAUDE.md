# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, canvas-based vertical shoot-'em-up game (Raiden-style), built entirely in HTML5 with no external dependencies. The game runs in `index.html` and delivers a complete arcade experience in ~1670 lines of code.

### Key Features
- Multiple enemy types (grunts, interceptors, tanks) with difficulty scaling
- 6 unique boss encounters with distinct attack patterns
- Dual weapon system (Vulcan rapid-fire, Laser piercing)
- Three difficulty levels: Easy, Normal, Hard
- Touch controls and mouse steering + keyboard support
- Synthesized sound effects (WebAudio, no external assets)
- Score/best score tracking via localStorage
- Responsive canvas scaling
- Reduced motion & accessibility support

## Architecture

The entire game logic runs in a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure is:

1. **DOM Setup** (lines 426-461): Cache all HTML element references
2. **Storage & Persistence** (lines 463-527): Best score, mute state, difficulty, reduced motion preferences
3. **Utility Functions**: Audio system, motion, UI helpers
4. **Configuration Objects** (lines 485-515):
   - `DIFFICULTIES`: Spawn rates, enemy speed, HP multipliers per difficulty
   - `THEMES`: 6 sky backgrounds with gradient colors
   - `ENEMY_TYPES`: Grunt, interceptor, tank properties
   - `BOSSES`: 6 boss designs with different HP multipliers
5. **Input Handling** (lines 768-862): Keyboard, mouse, touch/joystick
6. **Game State** (lines 863-869): Global game variables
7. **Game Logic**:
   - `update(dt)` (line 1148): Core tick update
   - `draw()` (line 1520): Canvas rendering
   - `loop(now)` (line 1640): requestAnimationFrame loop
8. **Event Listeners & Game Start** (line 1655+)

## Key Game Mechanics

### Spawning & Progression
- Difficulty determines spawn rate, enemy speed, shoot chance
- Stages advance after defeating 18 enemies (`KILLS_PER_STAGE`)
- Boss appears after warning delay when stage target reached
- Each stage picks a new boss from the 6 designs and a sky theme

### Weapon System
- **Vulcan** (default): Single → dual → triple spread bullets
- **Laser**: Single → dual lanes → dual + center, bullets pierce multiple enemies
- Power-up (`type: 'weapon'`) increments level 1→3, then toggles weapon type
- Firing is cooldown-based: Vulcan 0.15s, Laser 0.1s

### Collision & Scoring
- Enemies and bullets use axis-aligned bounding boxes (`rectsOverlap()`)
- Chain multiplier: increases 1x → 10x based on consecutive kills within 2.2s
- Boss defeat grants 500 points + applies to final score

### Boss Patterns
Each of 6 bosses has 3 attack patterns stored in `BOSS_PATTERNS` array. Patterns include:
- Radial bursts, aimed spreads, columns, spirals, crosses, homing missiles
- Telegraphing phase (22 frames visual warning) before attack fires
- Boss moves horizontally and bounces off canvas edges

### Performance Adaptation
`quality` variable (0-2) adjusts particle count and glow effects based on frame time averaging. Helps maintain 60fps on slower devices.

## Development Notes

### Making Changes
- **Game balance**: Tweak `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `BOSS_PATTERNS`
- **Visuals**: Modify drawing functions like `drawPlane()`, `drawBossShip()`, `drawClouds()`
- **Audio**: Add new sfx in the `sfx` object; use `beep()` and `noiseBurst()` helpers
- **New enemies**: Add to `ENEMY_TYPES`, update type selection in `spawnEnemy()`
- **Controls**: Modify input event listeners (lines 768-862) or joystick logic

### Canvas & Sizing
- **Native resolution**: 480×720 (BASE_W, BASE_H at line 753)
- **Scaling**: Maintains aspect ratio up to 1.6× zoom to fit screen
- Canvas is re-rendered every frame via `requestAnimationFrame`

### Storage Keys
- `skyfighter.bestScore` — Best score
- `skyfighter.muted` — Mute state (0/1)
- `skyfighter.difficulty` — Selected difficulty
- `skyfighter.reducedMotion` — Reduced motion enabled (0/1)

### Testing
- No build step or tests; open `index.html` directly in a browser
- Test all three difficulties by clicking buttons before starting
- Verify touch controls on mobile or with DevTools device emulation
- Check screen reader announcements via `announce()` for accessibility

### Common Tweaks
- **Enemy spawn speed**: `DIFFICULTIES[].spawnBase` and `spawnMin`
- **Boss HP**: Modify base HP at line 1046 or per-boss `hpMul`
- **Particle effects**: Adjust `spawnExplosion()` particle count
- **Screen shake**: Tweak magnitude/duration in `screenShake()` calls

## Performance Considerations
- Uses single canvas 2D context (no WebGL)
- Particles and bullets are pooled implicitly by filtering
- No frame-rate locking; adapts to 60fps or device refresh rate
- Reduced motion mode cuts visual effects to ~15% to respect preferences
