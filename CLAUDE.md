# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is a single-file HTML5 application (`index.html`, ~1669 lines) with:
- Canvas-based rendering
- Procedural enemy spawning with AI behaviors
- Boss encounters across multiple stages
- Weapon and bomb progression systems
- Touch and keyboard controls
- WebAudio synthesis (no external sound assets)
- Difficulty levels with balance parameters
- Persistent high-score tracking via localStorage
- Adaptive performance tuning

## Running and Development

### Quick Start
Simply open `index.html` in a web browser to play the game. No build step, server, or external dependencies required.

### Local Development
1. **Edit `index.html`** directly in any text editor
2. **Refresh the browser** (Cmd+R / Ctrl+R) to see changes
3. **Open DevTools** (F12) to debug via JavaScript console

### Testing Changes
- Use DevTools **Console** to test individual game systems
- Use **Pause (⏸ button)** in-game to freeze the simulation for frame-by-frame inspection
- Monitor **Network tab** for any external resource loads (should be none)

### Performance Considerations
- The game includes an adaptive performance system (`quality` variable: 0=full, 1=medium, 2=low)
- Frame times are tracked over a 45-frame window; rendering quality auto-adjusts if avg FPS drops below ~40
- Canvas rendering occurs every frame; use DevTools **Performance** tab to profile bottlenecks

## Code Architecture

### High-Level Structure
The JavaScript (lines 422–1669) is organized within an IIFE with these logical sections:

1. **Global State & DOM References** (lines 422–461)
   - All DOM element caches (canvas, HUD, overlay, controls)
   - Storage keys for localStorage persistence

2. **Utility & Core Subsystems** (lines 463–627)
   - Accessibility announcements (`announce()`)
   - Adaptive performance tracking (`trackPerf()`)
   - Game constants (difficulties, themes, enemy/boss definitions)
   - WebAudio synthesis system (`beep()`, `noiseBurst()`)

3. **Game State Management** (lines ~628–850)
   - Player state (position, health, bomb count, weapon, combo)
   - World state (active enemies, bullets, particles, bosses)
   - Stage progression and difficulty settings

4. **Entity Systems** (lines ~851–1200)
   - Enemy spawning logic with difficulty scaling
   - Enemy AI (movement patterns, evasion weaving, shoot decisions)
   - Player bullet firing and weapon progression
   - Boss behavior and stage transitions

5. **Rendering Pipeline** (lines ~1201–1450)
   - Canvas background gradient setup (themed skies)
   - Draw functions for player, enemies, bosses, effects, UI
   - HUD updates (score, health, weapon, boss health bar)

6. **Input Handling & Controls** (lines ~1451–1550)
   - Keyboard/mouse input
   - Touch joystick and button management
   - Input-to-game-action mapping

7. **Game Loop & State Machine** (lines ~1551–1669)
   - Main animation frame loop
   - Game state transitions (menu → play → game over)
   - Overlay management (start screen, pause, game over)

### Key Game Systems Explained

#### **Entity Architecture**
Enemies, bullets, particles, and the player are managed as arrays/objects in the game state. No class-based OOP; data is plain objects:
- **Player**: `{ x, y, width, height, vx, vy, lives, score, weapon, bombCount, combo, comboTime, multiplier }`
- **Enemies**: `{ x, y, type, hp, vx, vy, angle, shootCooldown, weaveTime, ... }`
- **Bullets**: `{ x, y, vx, vy, radius, team (player/enemy), damage, explodes }`

#### **Enemy Spawning**
- Enemies spawn at the top of the screen at randomized intervals
- Spawn rate decreases as a stage progresses (harder over time)
- `minStage` on each enemy type gates when they first appear
- Difficulty (easy/normal/hard) scales spawn rate, enemy speed, shoot chance, and boss HP

#### **Weapon System**
Four weapons available, each upgradable to 3 levels:
- **Vulcan** (red): Rapid fire, straight shot
- **Spread** (green): 3-way spread shot, longer cooldown
- **Laser** (blue): Piercing continuous laser, high energy
- **Missile** (yellow): Homing projectiles with arc motion

Weapon upgrades progress across **KILLS_PER_STAGE** (18 kills per weapon level).

#### **Bomb Mechanics**
- Limited bombs per life (varies by difficulty)
- On activation, kills all on-screen enemies and clears bullets
- Invincibility window follows bomb
- Sounds and visual feedback (white flash)

#### **Boss Battles**
- Trigger after stage completion (all regular enemies killed and encounter threshold met)
- Six boss types with distinct visual color schemes and HP multipliers
- Boss AI: moves in sinusoidal patterns, fires projectile spreads, cycles through attack phases
- Health bar displayed while boss is active

#### **Difficulty & Balance**
Three difficulty tiers in `DIFFICULTIES` object (easy/normal/hard):
- Govern spawn rates, enemy speed, shoot chance, starting lives/bombs, boss HP scaling
- Selectable from the overlay menu before game start
- Stored in localStorage to persist player choice

#### **Visual Themes**
Six environment themes cycle through stages, each with a sky gradient, cloud color, and sun tint. Themes are applied to the canvas background. Adding a new theme requires adding an entry to the `THEMES` array.

#### **Audio Synthesis**
All sound is procedurally generated via WebAudio API:
- `beep()`: Sine/square oscillator with frequency sweep, filter, and pan
- `noiseBurst()`: Noise generator with filter envelope
- Master gain node with dynamics compressor for leveling
- Mute toggle persisted to localStorage

#### **Scoring & Combos**
- Each enemy kill adds base points × combo multiplier
- Consecutive kills within a short time window increase combo meter (visual popup)
- Bonus multiplier displayed in HUD; decays when no enemies killed for ~1 second

#### **UI State Machine**
Three main overlays:
1. **Start screen**: Displays best score, difficulty selector, controls hint
2. **Pause**: Pauses game; resume or return to menu
3. **Game Over**: Shows final score, best score achieved, stage reached; option to restart

### Common Modification Points

#### **Adding a New Enemy Type**
1. Add entry to `ENEMY_TYPES` object (line ~503), e.g.:
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.5, color: '#0066cc', dark: '#003366', score: 25, minStage: 3, weave: true }
   ```
2. Handle spawn in `spawnEnemy()` call when enemy type is selected
3. Add custom AI logic in the enemy AI section if needed (weaving, special shot patterns, etc.)
4. Add corresponding draw logic in the render loop to paint the enemy's hull/details

#### **Adding a New Boss**
1. Add entry to `BOSSES` array (line ~508) with hull/wing/core colors and HP multiplier
2. Boss is auto-selected when boss phase triggers; only add the data definition
3. Custom boss rendering is already parameterized by colors; no new draw code needed unless visual complexity increases

#### **Adding a New Visual Theme**
1. Add entry to `THEMES` array (line ~493):
   ```javascript
   { sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
2. Themes auto-cycle through stages; canvas background gradient is set from current theme's sky array

#### **Adjusting Game Balance**
- **Enemy difficulty**: Modify values in `DIFFICULTIES` (spawn rate, speed, shoot chance, bomb/life count)
- **Weapon balance**: Adjust weapon stats in the weapon firing section (fire rate, damage, spread angle, speed)
- **Scoring**: Modify `KILLS_PER_STAGE` (kills needed per weapon upgrade) or base enemy `score` values
- **Boss difficulty**: Increase `bossHpMul` in difficulty presets or individual boss `hpMul` in `BOSSES` array

#### **Extending Controls**
Touch controls are handled in a separate section (lines ~1451+). Keyboard/mouse are checked in the input loop. To add a new input:
1. Add listener in the input setup section
2. Map the input to a game action (e.g., fire, bomb, move)
3. Update the corresponding player state in the game loop

#### **Adding a New Weapon**
1. Add weapon definition to the weapons data structure (includes firing pattern, cooldown, visual)
2. Add draw logic for the new projectile type in the render loop
3. Update weapon cycling logic in input handling
4. Add corresponding kill counter/upgrade logic

## Storage & Persistence

The game uses localStorage for:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Mute toggle (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (string: easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

Clearing localStorage resets all saved state.

## Accessibility Features

- **Screen reader support**: Live region (`#srLive`) announces game events
- **Reduced motion**: Toggle button respects `prefers-reduced-motion` media query and disables animations
- **Keyboard controls**: Fully playable without mouse
- **Touch controls**: On-screen joystick and buttons for mobile play

## Testing Gameplay

To verify changes work correctly:
1. **Play a full game** through multiple stages to ensure spawning, progression, and win/loss conditions work
2. **Test each difficulty** to ensure balance changes affect the right parameters
3. **Verify audio** by toggling mute and checking sound synthesis works
4. **Check touch controls** on mobile or via DevTools device emulation
5. **Inspect localStorage** in DevTools Application tab after gameplay to confirm persistence

## Performance Profiling

Use browser DevTools **Performance** tab:
1. Record a 5–10 second gameplay session
2. Look for frame drops (should stay above 30 FPS even on low-end devices)
3. Identify bottlenecks in rendering (canvas operations) vs. game logic (entity updates)
4. If FPS dips, the adaptive `quality` system will reduce effect complexity automatically

## Git Workflow

- Make targeted changes to `index.html`
- Test in browser before committing
- Commit messages should reference which system changed (e.g., "Add new interceptor enemy type", "Balance bomb cooldown on hard difficulty")
- No external assets to manage; the entire game ships as a single file
