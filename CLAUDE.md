# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application. It features weapon upgrades, smart bombs, combo chains, boss battles, difficulty levels, and full support for both desktop and mobile platforms with synthesized sound effects.

## Quick Start

### Opening & Testing
- **Test the game**: Open `index.html` in any modern web browser. No build process or dependencies required.
- **File structure**: Everything is contained in `index.html` (1669 lines) — HTML structure, CSS styles, and JavaScript game logic all in one file.

### Development Guidelines
- **Single-file architecture**: All changes go into `index.html`. No separate build step needed.
- **No external dependencies**: The game uses vanilla JavaScript, Canvas 2D, and WebAudio API only.
- **Testing approach**: After any change, open `index.html` in a browser and play through at least one stage to verify the change doesn't break the game loop or input handling.

## Architecture

### Core Game Loop (lines ~1500+)
The game uses a requestAnimationFrame-based loop that:
- Updates game state (player position, enemy spawns, bullet movements)
- Handles input from keyboard, mouse, and touch
- Renders all entities to the canvas
- Manages adaptive performance (automatically scales quality based on frame times)

**Key variables**:
- `player`: Player ship state (position, weapon, health, cooldown)
- `enemies`: Active enemy entities
- `bullets`, `enemyBullets`: Projectiles
- `particles`: Visual effects (explosions, hits)
- `powerUps`: Weapon upgrades and item drops
- `boss`: Boss encounter state

### Entity Rendering System (lines 913-1000+)
Canvas-based drawing for:
- **Player plane** (`drawPlane`): Main ship with dynamic flame when firing
- **Enemy ships**: Grunt, Interceptor, Tank types with distinct visuals
- **Boss ships** (`drawBossShip`): 6 unique boss designs with animated cores and telegraph patterns
- **Bullets & particles**: Simple geometric rendering

### Input Handling (lines 767-861)
Three input methods with priority (keyboard/joystick > mouse):
- **Keyboard**: Arrow keys or WASD for movement, Space for fire, X/Shift for bombs, P/Esc for pause
- **Mouse**: Move to steer, click to fire (on desktop)
- **Touch/Mobile**: Virtual joystick on left, FIRE and BOMB buttons on right

### Audio System (lines 542-676)
Fully synthesized sound using WebAudio API:
- `beep()`: Generates tones with optional frequency sweeps and filters
- `noiseBurst()`: White noise burst with optional filtering
- `sfx` object: Named effects (shoot, explode, bomb, hit, etc.)
- All sounds are procedurally generated — no external audio files
- Mutable via localStorage, respects reduced-motion preferences

### Game State Management (lines 863-911)
**Difficulty presets** (Easy/Normal/Hard) affect:
- Enemy spawn rate and minimum spawn interval
- Enemy movement speed and shoot probability
- Player lives and bomb count
- Boss health multiplier

**Progression**:
- `KILLS_PER_STAGE = 18`: Number of enemy kills required to advance stage and spawn boss
- Stages cycle through 6 themed visual backgrounds
- Each stage spawns a unique boss from the BOSSES list

### Combo & Multiplier System (lines ~1050+)
- Consecutive enemy kills within a time window (`chainTimer`) build a combo counter
- Combo reaches maximum multiplier (4x) at 20 kills
- Multiplier is displayed and applied to all enemy kill scores
- Chain resets if player takes damage or is idle too long

### Weapon System
- **Vulcan** (default): Rapid single-shot bullets with cooldown
- **Laser**: Slower, high-damage single shot with sweep effect
- **Levels 1–3**: Each level increases cooldown reduction and/or damage
- **Powerups**: Drop from specific enemy types; cycle through weapon upgrades

### Boss AI & Patterns
- **Telegraph phase**: Boss shows bright core and plays warning sound
- **Attack patterns**: Each boss type has unique movement and projectile patterns
- **Health bar**: Displayed when boss is active; updates in real-time
- **Defeat**: Boss explosion triggers stage clear, next stage boss loads on next spawn

### Visual Effects & UI
- **HUD** (lines ~33-71): Displays score, best score, weapon name/level, health hearts, bomb count, multiplier
- **Boss bar**: Health bar and name shown when in boss encounter
- **Screen shake**: Triggered on damage or bomb detonation (reduced in low-motion mode)
- **Flash effects**: White flash on hit, brighter flash on bomb
- **Vignette**: Red pulsing vignette when player has 1 life remaining
- **Combo popup**: Floating text showing current combo count
- **Stage banner**: Appears at start of each stage

### Adaptive Performance (lines 473-483)
Monitors frame times and automatically:
- Quality level 0 = full quality (default)
- Quality level 1 = medium (fewer particles/effects)
- Quality level 2 = low (minimal effects)
Scales down when frame times exceed 1/40s, scales up when below 1/56s.

### Accessibility & Preferences
- **Reduced Motion**: Toggleable via 🎆 button; disables shake, flash, and some animations
- **Mute**: Toggleable via 🔊 button
- **localStorage**: Persists best score, mute state, difficulty, and reduced-motion preference

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502-506) with w/h/hp/speedMul/color properties
2. Update `EXPLODE_SFX` mapping if needed (line 678)
3. Modify enemy spawn logic around line 1100 to include the new type in available spawns
4. Add rendering code in the main draw loop

### Adding a New Boss
1. Add entry to `BOSSES` array (lines 508-515) with name, hull colors, wing color, core color, and hpMul
2. The boss encounter is automatically triggered after KILLS_PER_STAGE enemies are killed
3. Boss attack pattern logic is in `updateBoss()` function (~line 1200+)

### Tweaking Difficulty
Edit the `DIFFICULTIES` object (lines 485-489):
- `spawnBase`: Base spawn interval in frames (lower = faster spawning)
- `spawnMin`: Minimum spawn interval floor
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: 0.0–1.0 probability per frame an enemy fires
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Multiplier on boss max health

### Adjusting Visual Theme
Themes are defined in `THEMES` array (lines 493-500). Each theme has:
- `sky`: Array of gradient colors for background
- `cloud`: RGB color for cloud rendering
- `sun`: RGB color for sun
- `name`: Display name

Themes cycle once per stage.

### Modifying Audio
Examine the `sfx` object (lines 624-676):
- Call `beep(freq, duration, type, volume, opts)` for tonal sounds
- Call `noiseBurst(duration, volume, opts)` for percussive/explosion sounds
- Use `opts.sweepTo` for frequency sweep, `opts.filterFreq` and `opts.filterType` for filtering
- Use `opts.pan` to position sound left/right based on game coordinates
- Test muted mode: audio initialization is lazy (triggered on first input)

## File Locations (All in index.html)

| Section | Lines | Purpose |
|---------|-------|---------|
| HTML structure | 1–400 | Canvas, overlay, HUD elements, touch controls |
| CSS styles | 9–400 | Layout, animations, responsive design, dark theme |
| Constants & config | 422–524 | Game balance, difficulty, enemy/boss definitions, themes |
| Audio | 542–676 | WebAudio synthesis, sound effects |
| UI rendering | 680–750 | Hearts, bombs, combo pop-ups, screen shake |
| Canvas setup | 752–765 | Canvas sizing and responsive scaling |
| Input handling | 767–861 | Keyboard, mouse, touch, joystick input |
| Game state | 863–911 | Player, enemies, game loop variables |
| Drawing functions | 913–1050 | Entity rendering (player, enemies, bosses, bullets) |
| Game loop & update | 1050–1600 | Core loop, collision, spawn logic, state updates |
| Event listeners | 1600–1669 | Start button, pause, difficulty selection |

## Testing Checklist
- [ ] Game starts without console errors
- [ ] Player can move and fire with keyboard/mouse/touch
- [ ] Enemy spawning and movement works
- [ ] Collisions register correctly (player hit, enemy hit)
- [ ] Weapon upgrades drop and switch weapon type
- [ ] Bombs detonate and clear enemies in radius
- [ ] Combo counter increments on consecutive kills
- [ ] Boss appears after KILLS_PER_STAGE kills
- [ ] Stage progression and theme cycling works
- [ ] Game over screen shows final score and best score
- [ ] Difficulty selection affects game speed/enemy count
- [ ] Audio plays (if unmuted) and mute button toggles it
- [ ] Touch controls appear on mobile and work correctly
- [ ] Reduced motion toggle removes shake and flash effects
- [ ] Best score persists across page reloads
