# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features adaptive difficulty, procedural enemy waves, boss battles across 6 visual themes, and built-in sound synthesis using the Web Audio API.

### Key Features
- **Single HTML file**: Complete game (CSS + JavaScript) embedded in `index.html`
- **Canvas-based rendering**: Uses 2D canvas for all graphics
- **Web Audio synthesis**: Generated sound effects and music (no external audio files)
- **Adaptive performance**: Real-time frame-rate monitoring with automatic quality degradation
- **Touch and mouse controls**: Responsive input handling with virtual joystick for mobile
- **Multiple difficulty levels**: Easy (4 lives), Normal (3 lives), Hard (3 lives)
- **Persistent state**: Best score, mute state, difficulty, and motion preferences saved to localStorage

## Codebase Structure

The entire game is implemented in a single **IIFE (Immediately Invoked Function Expression)** starting at line 423 in `index.html`. All game state, entities, and logic are contained within this closure for namespace isolation.

### Major Components

1. **Game Loop** (`loop` function, line 1640)
   - Uses `requestAnimationFrame` for 60 FPS target
   - Delta-time calculation with 50ms cap to prevent skipped frames
   - Calls `update(dt)` for game logic, then `draw()` for rendering
   - Performance tracking via `trackPerf()` adjusts quality based on frame times

2. **State Management**
   - Player entity: position, health, weapon level, ammo
   - Enemies array: spawned enemies with type, position, velocity, health
   - Boss entity: active boss with multi-phase behavior
   - Bullets array: player shots and enemy projectiles
   - Particles: explosions and visual effects
   - Game flags: `running`, `paused`, `gameOver`

3. **Entity Types**
   - **Player**: Controllable ship with health, weapons, bomb ability
   - **Enemies**: `grunt` (basic), `interceptor` (fast, weaves), `tank` (heavy)
   - **Boss**: 6 unique boss variants (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - **Bullets**: Player shots and enemy projectiles with collision detection

4. **Rendering** (`draw` function, line 1484)
   - Sky background with parallax scrolling clouds
   - Enemy and boss sprites drawn with `drawPlane()` and `drawBossShip()`
   - Projectiles, explosions, and HUD elements
   - Performance quality affects particle count and visual effects

5. **Audio System** (lines 542-678)
   - `ensureAudio()`: Lazy initializes AudioContext on first user interaction
   - `beep()`: Oscillator-based sound generation with frequency sweep and filtering
   - `noiseBurst()`: White noise generation for explosions and power-ups
   - All SFX defined in `sfx` object with named presets for different game events
   - Panning support based on screen position

6. **Input Handling**
   - Mouse/touch: `canvasPointFromEvent()` converts screen coordinates
   - Virtual joystick for mobile: `joystickStart()`, `joystickMove()`, `joystickEnd()`
   - Keyboard support via `keys` object (W/A/S/D or arrow keys for movement)
   - Fire button: left-click or touch on fireBtn
   - Bomb button: right-click, spacebar, or touch on bombBtn

7. **Game Mechanics**
   - **Scoring**: Base points + difficulty multiplier + combo chain multiplier
   - **Weapon progression**: 4-level weapon system (VULCAN → SPREAD → HOMING → RAPID)
   - **Enemy spawning**: Difficulty-based spawn rates with stage progression (18 kills per stage)
   - **Boss encounters**: Occur every 6 stages with phase-based attack patterns
   - **Power-ups**: Weapons, health, bombs spawned from defeated enemies
   - **Themes**: 6 visual themes cycle through stages, each with unique color scheme

### Key Constants and Configuration

- **DIFFICULTIES** (line 485): Spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- **THEMES** (line 493): 6 visual themes with sky gradients and cloud colors
- **ENEMY_TYPES** (line 502): Grunt, Interceptor, Tank with dimensions, HP, speed, scoring
- **BOSSES** (line 508): 6 boss variants with hull/wing/core colors and HP multipliers
- **KILLS_PER_STAGE** (line 491): 18 kills advance to next stage and spawn boss every 6 stages
- **STORAGE_*** constants** (lines 463-466): localStorage keys for persistence

## Common Development Tasks

### Running the Game
Simply open `index.html` in a web browser. No build step or server required. For development, use a local server to avoid CORS issues:
```bash
python3 -m http.server 8000
# then visit http://localhost:8000/
```

### Testing Changes
1. Open the game in a browser
2. Use browser DevTools (F12) to inspect and modify variables in the console
3. Key variables to debug:
   - `player`: Player entity with x, y, health, weapon, ammo
   - `enemies`: Array of active enemies
   - `boss`: Current boss entity (null if no active boss)
   - `score`, `combo`, `stage`: Game state
   - `reducedMotion`, `muted`: Accessibility settings
   - `difficulty`: Current difficulty level

### Adding New Enemy Types
1. Define in `ENEMY_TYPES` object (line 502)
2. Add spawn logic in `spawnEnemy()` function (line 1022)
3. Update enemy drawing in `draw()` function if visual differs from standard

### Adding New Boss Variants
1. Define in `BOSSES` array (line 508)
2. Implement attack pattern in `updateBoss()` function (line 1411)
3. Boss fire patterns use dedicated functions (`fireRadial`, `fireAimedSpread`, etc.)

### Adjusting Difficulty Balance
- Modify `DIFFICULTIES` object for spawn rates, speed, health
- Adjust `KILLS_PER_STAGE` (line 491) for stage length
- Tune weapon damage and cooldown in `fireWeapon()` function (line 1124)
- Boss HP multiplier in boss definition affects all bosses proportionally

### Audio/SFX Customization
- Define new sounds in `sfx` object (line 624) using `beep()` and `noiseBurst()`
- Call via `sfx.soundName()` to trigger
- Mute state respects `muted` flag; test with `muteBtn` toggle

## Performance Considerations

- **Adaptive quality system**: Frame times tracked over 45 frames; quality adjusts if avg > 25ms
- **Quality levels**: 0 = full effects, 1 = medium (reduced particles), 2 = low (minimal particles)
- **Collision detection**: Uses simple `rectsOverlap()` (line 1069) for all entity collisions
- **Particle limits**: Explosion particle count varies by quality level
- **Canvas size**: Responsive; `resize()` called on window resize (line 754)

## Accessibility

- **Screen reader support**: Announcements via `announce()` function to `#srLive` element
- **Reduced motion**: Toggle via motionBtn; disables screen shake, flash, and pulse effects
- **Keyboard navigation**: Full game playable with keyboard (arrow keys + spacebar/Z)
- **Touch-friendly**: Virtual joystick and large buttons for mobile/touch devices
- **Storage**: Persistence respects user preferences across sessions

## File Structure

```
/
├── index.html           # Complete single-file game (HTML + CSS + JS)
├── README.md            # Basic project info
└── CLAUDE.md            # This file
```

## localStorage Keys

- `skyfighter.bestScore`: Best score achieved (Number)
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Motion preferences (0 or 1)

## Debugging Tips

- **Pause the game**: Press ⏸ button or call `togglePause()` to freeze for inspection
- **Screen reader output**: Check console logs from `announce()` calls
- **Performance metrics**: `frameTimes` array contains last 45 frame durations
- **Event logging**: Add `console.log()` in `update()` to track entity state changes
- **Visual debugging**: Use browser DevTools to highlight canvas collisions or overlay grids
