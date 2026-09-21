# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single HTML file with no external dependencies. It's a fully self-contained browser game with WebAudio sound synthesis, responsive design for desktop and mobile, and accessibility features.

## Quick Start

No build process or dependencies required. Simply open `index.html` in a modern web browser:
- **Desktop**: Drag `index.html` into a browser window or use `file://` URL
- **Development**: Use a local HTTP server (e.g., `python3 -m http.server 8000`) to avoid CORS issues
- **No build step**: All code is inline in the single HTML file

### Testing the Game

To verify changes work correctly:
1. Run a local dev server: `python3 -m http.server 8000` or `npx serve`
2. Open browser to `http://localhost:8000`
3. Test controls (mouse, keyboard, touch for mobile)
4. Verify audio works (WebAudio synthesis)
5. Check browser console for errors

## Architecture Overview

The entire game is contained in a single `<script>` block (lines 422-1667) using a self-executing function with strict mode. Key architectural patterns:

### Game State
- **Player object**: position, health, weapon, cooldown timers, invulnerability frames
- **Game arrays**: bullets, enemies, enemyBullets, particles, powerUps, boss
- **Global state**: score, lives, stage, running flag, paused flag, difficulty, UI elements
- **LocalStorage persistence**: best score, muted state, difficulty choice, reduced motion preference

### Core Game Loop
```
requestAnimationFrame(loop)
  → update(dt)  [game logic]
  → draw()      [canvas rendering]
  → shakeTime   [screen shake decay]
```

The game maintains 60 FPS target with adaptive quality scaling based on frame timing (quality levels 0-2 for full/medium/low performance).

### Key Game Systems

**Input Handling** (lines 767-861):
- Keyboard: Arrow keys, WASD for movement; Space for fire; X/Shift/P/Esc for special actions
- Mouse: Position-based steering; click to fire
- Touch: Left side drag for movement (joystick), right side buttons for fire/bomb
- All input systems work independently without conflict

**Rendering** (lines 1520-1638):
- Canvas 2D context with base resolution 480×720 (scaled to fit screen)
- Sky gradient + sun glow + parallax clouds (per-stage theme)
- Custom ship drawing functions for player and enemies
- Boss has dedicated drawing logic with telegraph animation
- Particles for explosions, with adaptive count based on quality setting

**Enemy AI** (lines 1022-1041, 1210-1228):
- Enemies spawn based on difficulty timer with increasing spawn rate
- Three types: grunt (weak/fast), interceptor (dodgy), tank (tanky)
- Two shot patterns: "spread" (radial) and "aimed" (homing toward player)
- Interceptor enemies weave back/forth with sine wave pattern
- Enemies despawn when off-screen

**Boss System** (lines 1043-1056, 1411-1439):
- 6 unique boss designs, each with distinct attack patterns
- BOSS_PATTERNS array (lines 1372-1409) defines pattern functions
- Each pattern fires bullets with different speeds/spreads/formations
- Boss enters from top, moves left/right, telegraphs before attacking
- HP bar displays at top during boss fight

**Weapons & Upgrades** (lines 1124-1146, 1304-1322):
- Vulcan: rapid fire spread projectiles (spread increases at levels 2-3)
- Laser: slower, piercing projectiles that hit multiple enemies
- Power-ups: weapon type toggle, level increase (max 3), extra life, bomb
- Switching weapon resets level to 1

**Scoring & Multiplier** (lines 1087-1101):
- Score multiplier based on consecutive kills (chain)
- Chain resets if no kills within 2.2 seconds (chainTimer)
- Multiplier formula: `1 + floor(chain / 5)` (max 10x at 50+ chain)
- Visual feedback: combo counter at 10-kill intervals, multiplier display in HUD

**Audio** (lines 542-676):
- Uses WebAudio API to synthesize all SFX (no audio files)
- Master gain with dynamics compressor for consistent volume
- beep(): tone generation with optional frequency sweep, filtering, panning
- noiseBurst(): noise-based sounds with filter envelope
- 13 different SFX types (shoot, laser, explode variants, hit, bomb, powerup, boss sounds)
- Panning based on screen position (stereo spatial audio)
- Can be muted via button; persists in localStorage

**UI & Overlays** (lines 34-358):
- HUD shows score, best, multiplier, weapon, hearts, bombs
- Overlay modals for start screen, pause, game over
- Stage and combo popups with scale animations
- Boss name and health bar with smooth transitions
- Reduced motion mode disables all animations when enabled
- Screen reader announcements via aria-live region

**Difficulty System** (lines 485-489):
- Easy, Normal, Hard with different spawn rates, enemy speeds, player lives
- Affects: spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Selected difficulty persists in localStorage
- Can change difficulty from main menu

**Responsive Design**:
- Canvas scales to fit window while maintaining 480×720 aspect ratio
- Touch controls enabled on mobile devices
- Safe area insets for notched phones
- Clamp() functions for responsive font sizing
- Works at 16:9 and taller aspect ratios

## File Structure

Only one file to modify:
- `index.html` — Contains all HTML markup, CSS, and JavaScript

## Development Guidelines

### Adding New Enemy Types

1. Add entry to `ENEMY_TYPES` object (line 502-506):
   ```js
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.1, color: '#..', dark: '#..', score: 20, minStage: 2 }
   ```
2. Update spawn logic in `spawnEnemy()` to include in choices
3. Add explosion SFX mapping to `EXPLODE_SFX` if needed (line 678)
4. Update `BOSS_PATTERNS` if enemy should trigger special patterns

### Adding New Boss Patterns

1. Define pattern function in `BOSS_PATTERNS` array (line 1372)
2. Each pattern function receives boss object `b` and fires bullets via enemyBullets.push()
3. Radial bullets have `{ x, y, w, h, radial: true, dx, dy }` structure
4. Call helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
5. Boss cycles through patterns in order; pattern index increments after each use

### Adding New Stages/Themes

1. Add theme to `THEMES` array (line 493):
   ```js
   { sky: ['#...', '#...', '#...'], cloud: '...', sun: '...', name: 'Theme Name' }
   ```
2. Sky array is [top color, middle, bottom] for gradient
3. Cloud/sun use RGB values (no #) for dynamic rgba
4. Themes cycle per stage: stage 1→theme 0, stage 7→theme 0 again, etc.

### Adding New Power-ups

1. Update `maybeSpawnPowerUp()` spawn chances (line 1058-1067)
2. Add type string to power-up object: `{ x, y, w: 20, h: 20, speed: 90, type: 'newType' }`
3. Handle in `applyPowerUp()` switch statement (line 1304-1322)
4. Add color to rendering logic in `draw()` powerup section (line 1621)

### Audio Debugging

- `beep()` parameters: freq (Hz), duration (s), type ('sine'/'square'/'sawtooth'/'triangle'), vol (0-1), opts
- `opts` can include: sweepTo (target freq), filterFreq, filterType ('lowpass'/'highpass'), pan (-1 to 1), delay
- Set `muted = true` to silence during development
- Check browser console for audio context state errors (may be 'suspended' on user gesture)

### Performance Optimization

- Quality levels (0=full, 1=medium, 2=low) auto-scale based on frame timing
- Particle count scales with quality setting (line 1074)
- Shadow blur and glow effects disabled at quality > 0
- Monitor `frameTimes` array tracking last 45 frame durations
- Quality increases if avg > 1/40 (25ms), decreases if avg < 1/56 (~18ms)

### Mobile Considerations

- Touch device detection via `('ontouchstart' in window)` (line 778)
- Two-finger control zones: left 55% for movement, right 45% for actions
- Joystick base shows while dragging, has 50px radius constraint
- Buttons sized at 78px for touch accuracy (line 311)
- Test with browser DevTools mobile emulation

## Key Constants & Formulas

- **Base canvas resolution**: 480×720 (lines 753)
- **Kills per stage**: 18 enemies to trigger boss (line 491)
- **Base boss HP**: `70 + stage * 35` (line 1046)
- **Combo multiplier**: `1 + floor(chain / 5)`, capped at 10x
- **Chain timeout**: 2.2 seconds between kills (line 1089)
- **Max quality levels**: 0-2 based on frame performance
- **Invulnerability frames**: 1.5 seconds after taking damage

## Browser APIs & Features Used

- **Canvas 2D**: Drawing shapes, text, gradients, transformations, shadows
- **WebAudio**: Oscillators, filters, gainNodes, dynamics compressor
- **LocalStorage**: Persistent game state (best score, settings)
- **RequestAnimationFrame**: Game loop with vsync
- **Touch Events**: Multi-touch support with tracking by identifier
- **Media Queries**: Reduced motion preference detection, safe area insets
- **Screen Reader**: aria-live region for announcements

## Debugging Tips

- Check browser console for any errors (F12)
- Verify canvas size matches viewport: `canvas.width/height` vs CSS dimensions
- Test audio via `ensureAudio()` — may fail if AudioContext is suspended
- Use `console.log(quality)` to monitor adaptive performance scaling
- Check localStorage: `localStorage.getItem('skyfighter.bestScore')`
- Test with network throttling to simulate slow devices

## Commit Message Guidelines

When committing changes:
- Describe the WHAT and WHY, not the HOW
- Reference affected systems (e.g., "Boss patterns", "Audio", "Mobile controls")
- Keep messages concise and descriptive
- Example: "Add new tank enemy type with explosive SFX and splash damage"

## Future Enhancement Ideas

- Additional stages/themes beyond 6
- New weapon types or upgrade paths
- Leaderboard persistence (IndexedDB)
- Gamepad/controller support
- Replay system for recording/playback
- Visual effects (screen tint on critical health)
- Special events or themed bosses
- Achievements/unlockables
