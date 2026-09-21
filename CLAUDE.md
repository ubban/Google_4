# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, JavaScript) is contained in `index.html`. It's a fully self-contained, browser-based game with no build system or external dependencies.

**Key Features:**
- Vertical shoot-'em-up gameplay with player ship, enemies, and stage bosses
- Weapon system (Vulcan and Laser with upgrade levels)
- Smart bomb mechanic for clearing bullets and damaging enemies
- Combo/chain system for scoring multipliers
- Adaptive difficulty (Easy/Normal/Hard)
- Accessibility features (reduced motion support, screen reader announcements)
- Touch and keyboard controls
- Procedural audio synthesis (no audio files)
- Adaptive quality settings based on frame performance

## How to Run the Game

1. **Open in Browser:** Double-click `index.html` or open it in any modern web browser
2. **Local Development Server (optional):** Run `python3 -m http.server 8000` (or similar) and visit `http://localhost:8000` to test locally
3. **Mobile Testing:** Open on mobile devices to test touch controls and responsive layout

## Codebase Structure

The entire codebase is in a single file: `index.html`

**File breakdown:**
- **Lines 1-359:** HTML structure and CSS styling (layout, HUD, animations, buttons, touch controls)
- **Lines 422-1670:** JavaScript game implementation (all game logic, rendering, audio)

## High-Level Game Architecture

The game follows a classic game loop pattern with these main systems:

### 1. **Game Loop** (lines 1640-1653)
- Runs via `requestAnimationFrame` for smooth 60 FPS rendering
- Each frame: calculate delta time → update game state → render to canvas
- Tracks performance and adapts quality (lines 473-482) to maintain frame rate

### 2. **Game State** (lines 863-870)
Central state variables track:
- Player object (position, weapon, health, cooldown)
- Bullets array (player projectiles with pierce values)
- Enemies array (with health, shooting, movement patterns)
- Enemy bullets array (incoming projectiles)
- Power-ups array (life, bombs, weapon upgrades)
- Boss object (current stage boss)
- Score, lives, stage, chain/multiplier counters

### 3. **Update System** (lines 1148-1302)
The `update(dt)` function handles all game logic each frame:
- **Input Processing** (lines 1152-1181): Keyboard, mouse, and joystick input
- **Entity Movement** (lines 1188-1195): Bullets, enemy bullets, power-ups
- **Spawning** (lines 1197-1207): Enemy and boss spawn timing based on stage and difficulty
- **Collision Detection** (lines 1232-1294): Enemy-bullet, enemy-player, boss-bullet, powerup-player collisions
- **Scoring & Chain System** (lines 1087-1101): Tracks consecutive kills for multiplier bonus

### 4. **Rendering System** (lines 1520-1637)
The `draw()` function renders everything to canvas:
- Background gradient with animated clouds (lines 1529-1544)
- Particle effects (line 1628-1636)
- Player ship with weapon aura (lines 1546-1559)
- Bullets, enemies, boss (lines 1580-1615)
- Power-ups (rotating square icons, lines 1617-1626)
- **Screen shake effect** (line 1523-1527): Applied to canvas translation when `shakeTime > 0`
- **Glow effects:** Conditionally rendered when `quality === 0` (high-quality mode)

### 5. **Input System** (lines 767-861)
Handles multiple input methods with priority:
1. **Keyboard/Joystick:** Direct movement vectors
2. **Mouse:** Player glides toward cursor when mouse is active (lines 1161-1169)
3. **Touch:** Virtual joystick on left side, FIRE/BOMB buttons on right (lines 778-834)
- Fire action is continuous (Space key, mouse click, or FIRE button held)
- Bomb is discrete (X, Shift, or BOMB button tap)

### 6. **Audio System** (lines 542-676)
WebAudio API synthesis (no audio files):
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone generation with pitch sweep, filtering, panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope and filtering
- All SFX defined in `sfx` object (lines 624-676)
- Muted state persists in localStorage

### 7. **Boss System** (lines 1041-1055, 1372-1439)
- Boss spawns after `KILLS_PER_STAGE` (18 kills) with warning banner
- Each stage gets a themed boss from `BOSSES` array (lines 508-515)
- Boss has unique firing pattern from `BOSS_PATTERNS` array (lines 1372-1409)
- Boss telegraph system: windup phase (white core flash) → attack pattern fires
- Boss health bar updates in HUD (lines 1299-1301)

### 8. **Difficulty System** (lines 485-489)
Three difficulty presets (Easy/Normal/Hard) affect:
- Enemy spawn rate and minimum spawn frequency
- Enemy speed and shoot accuracy
- Player starting lives and bombs
- Boss HP multiplier

## Key File Sections for Common Tasks

| Task | Lines | Notes |
|------|-------|-------|
| Add a new SFX | 624-676 | Add function to `sfx` object, use `beep()` or `noiseBurst()` |
| Add enemy type | 502-506 | Add to `ENEMY_TYPES` object, include `w`, `h`, `hp`, `speedMul`, `score`, `minStage` |
| Add boss pattern | 1372-1409 | Add pattern array to `BOSS_PATTERNS`, use firing helper functions |
| Modify weapon behavior | 1124-1146 | Edit `fireWeapon()` function, adjust spread/pierce/speed |
| Change game balance | 485-489, 491 | Modify `DIFFICULTIES`, `KILLS_PER_STAGE`, base values |
| Add visual effect | 1073-1084 | Particle system in `spawnExplosion()`, customize color and count |
| Modify HUD display | 367-374 | HTML elements, update in `update()` at lines 899-906 |

## Important Implementation Details

### Collision Detection
`rectsOverlap()` (line 1069-1070) uses AABB (axis-aligned bounding box) collision. All entities must have `x`, `y`, `w`, `h` properties.

### Coordinate System
- Origin (0, 0) is top-left
- Canvas size: 480×720 (BASE_W, BASE_H at line 753)
- Canvas scales responsively to fit window (lines 754-765)

### Invulnerability & Protection
- Player gains invulnerability after taking damage: `player.invuln = 1.5` seconds (line 1459)
- Invulnerability period causes blinking (line 1546): skip draw every 4 frames
- Boss entering phase (lines 1413-1416): boss moves down but can't be hit

### Weapon Piercing
- Vulcan bullets have `pierce: 0` (destroyed on hit)
- Laser bullets have `pierce` equal to weapon level (can hit multiple enemies)
- Boss takes 3 damage per hit (line 1259), not affected by pierce

### Chain/Combo System
- Kills increment `chain` counter and reset `chainTimer` (lines 1088-1089)
- Multiplier = 1 + floor(chain / 5), max 10x at 50+ chain (line 1090)
- Chain breaks if timer reaches 0 without new kills (lines 1183-1186)
- Milestone messages at 10, 20, 30, etc. kills (line 1248)

### Quality/Performance Adaptation
- Game tracks frame times over last 45 frames (lines 475-482)
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Higher quality = more particles, glow effects, shadows
- Reduced motion mode disables animations, screen shake, flash effects

### Local Storage
Four keys persist user preferences (lines 463-466):
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — audio mute state
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — accessibility preference

## Development Workflow

1. **Edit the game:** Modify `index.html` in your editor
2. **Test in browser:** Reload the page (Ctrl+R or Cmd+R)
3. **Check console:** Press F12 to open DevTools and see any errors
4. **Performance:** Toggle quality/effects via the motion button (🎆) to test at different quality levels
5. **Mobile testing:** Use DevTools device emulation or test on actual device

## Performance Considerations

- **Canvas rendering:** Draw calls are batched per entity type for efficiency (bullets, enemies, etc.)
- **Particle limits:** Automatic particle count scaling based on quality setting
- **No object pooling:** Entities are created/destroyed freely; garbage collection is acceptable for this game scale
- **Audio:** WebAudio synthesis runs only when muted is false; oscillators are stopped immediately
- **DOM updates:** Minimal DOM thrashing; HUD updates only when values change

## Themes and Styling

The game includes 6 visual themes (lines 493-500) that cycle per stage:
- Each theme defines: sky gradient colors, cloud color, sun color, theme name
- Clouds drawn twice with different speeds for parallax effect (lines 1543-1544)
- Theme selection: `THEMES[(stage - 1) % THEMES.length]`

## Testing Checklist for Modifications

When adding features or fixing bugs:
- [ ] Test on desktop (keyboard input, mouse steering)
- [ ] Test on mobile (touch controls, reduced-motion setting)
- [ ] Test all three difficulty levels
- [ ] Verify audio works with mute toggle
- [ ] Check frame rate with DevTools (should stay 60 FPS)
- [ ] Test boss health bar accuracy
- [ ] Confirm collision detection is precise (no clipping through walls)
- [ ] Verify high score persistence across page reloads
