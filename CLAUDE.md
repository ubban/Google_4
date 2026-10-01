# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file web game (HTML with embedded JavaScript and CSS) that runs entirely in the browser with no external dependencies or build system. The game features adaptive performance scaling, multiple difficulty levels, boss encounters, and procedural audio synthesis.

## Development Setup

There is no build process or external dependencies. The game runs directly by opening `index.html` in a browser:

```bash
# macOS/Linux - open in default browser
open index.html
# or
xdg-open index.html

# Python built-in server (any OS)
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Running and Testing

- **Development**: Open `index.html` directly in a browser. Use browser DevTools (F12) to debug.
- **Manual testing**: Test across desktop (mouse/keyboard) and mobile (touch) inputs.
  - **Desktop controls**: Mouse (move), click/Space (fire), X/Shift (bomb), P/Esc (pause), Arrows/WASD (alternative move)
  - **Mobile controls**: Drag left side (move), tap FIRE, tap BOMB
- **Performance**: The game includes adaptive quality tracking (see `trackPerf()` in code). Monitor FPS drops using browser DevTools.

## Code Architecture

### Main Game Loop
The game uses a single `requestAnimationFrame` loop (`loop()` function, ~line 1630) that:
1. Calculates delta time from the previous frame
2. Updates game state via `update(dt)`
3. Renders via `draw()`
4. Tracks performance and adapts quality

### Core Game Objects

**Player** (player object, ~line 900)
- Position, health, weapon state, bomb count
- Collision detection with enemies and projectiles
- Weapon switching logic (cyclical through weapon types)

**Enemies** (~line 750)
- Three types: `grunt`, `interceptor`, `tank` (defined in `ENEMY_TYPES`)
- Each has different speed, HP, and behavior
- Interceptors have weaving movement; tanks are slower but durable
- Enemies shoot at random intervals based on difficulty

**Bosses** (~line 750 for types, ~line 1200 for spawn logic)
- Six boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each stage has one boss encounter after 18 enemy kills
- Bosses have unique color schemes and health multipliers
- Boss health and difficulty scale based on selected difficulty and stage count

**Projectiles**
- Player shots: standard fire, laser (weapon upgrade)
- Enemy shots: basic projectiles
- Collision detection against enemies and player
- Removed when they leave the canvas bounds

**Particles** (rendering at ~line 1520)
- Floating text (score popups, combo notifications)
- Debris particles after explosions with gravity and fade-out

### Game Systems

**Difficulty & Balance** (~line 485)
- Three levels: easy, normal, hard
- Each affects spawn rate, enemy speed, shoot chance, lives, bombs, boss HP
- Difficulty is persisted in localStorage

**Weapon & Upgrade System** (~line 900)
- Weapon levels (0–3) that increase projectile count and fire rate
- Weapon drops from specific enemy kills
- Power-up types: weapon upgrade, health, bombs
- Weapon and power-up pickups are shown as collectible objects

**Adaptive Performance** (~line 474)
- Tracks frame times over a 45-frame window
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Scales particle effects and visual quality based on FPS
- Adjusts dynamically to maintain smooth gameplay

**Audio System** (~line 542)
- Procedural WebAudio synthesis—no external audio files
- Separate SFX for each action (shoot, laser, explosions, power-ups, boss defeat, etc.)
- Synth parameters: oscillator type, frequency sweeps, filters, panning
- Can be muted and persisted in localStorage

**Controls & Input** (~line 1420)
- Keyboard: WASD/Arrows for movement, Space/click for fire, X/Shift for bomb
- Mouse: move to steer, click to fire
- Touch: left-side drag zone for movement, FIRE and BOMB buttons
- Pause with P or Esc

**UI & HUD**
- Score, best score, multiplier, weapon name/level display
- Heart icons for remaining lives; bomb icons for available bombs
- Boss health bar (shown only during boss encounters)
- Stage transition banner
- Combo popup animation
- Mute and motion-reduction buttons in top-right

**Themes** (~line 493)
- Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Each theme has a background gradient, cloud color, and sun/accent color
- Themes cycle through stages for visual variety

**localStorage Persistence**
- Best score: `skyfighter.bestScore`
- Mute state: `skyfighter.muted`
- Difficulty: `skyfighter.difficulty`
- Reduced motion preference: `skyfighter.reducedMotion`

### Key Functions to Know

| Function | Purpose |
|----------|---------|
| `loop(now)` | Main game loop; calls update and draw each frame |
| `update(dt)` | Game logic: move entities, handle collisions, manage spawning |
| `draw()` | Render all game objects to canvas |
| `resetGame()` | Initialize new game; reset player, enemies, score |
| `spawnEnemies()` | Spawn waves of enemies based on spawn rules |
| `drawBoss()` | Render the current boss with color-coded hull/core |
| `trackPerf(dt)` | Adaptive quality system; adjusts based on frame times |
| `beep(freq, dur, type, vol, opts)` | WebAudio beep; used for most SFX |
| `noiseBurst(dur, vol, opts)` | WebAudio noise; used for explosions and complex sounds |

## Performance Considerations

- **Canvas rendering**: All graphics are drawn via 2D canvas context; no WebGL
- **Particle limits**: If quality is reduced, particle effects are disabled
- **Collision detection**: Simple AABB (axis-aligned bounding box) checks; O(n²) for bullets vs. enemies, but kept manageable by culling off-screen objects
- **Audio synthesis**: WebAudio synthesis runs synchronously; avoid adding many simultaneous SFX
- **Mobile touch**: Touch controls use pointer events to track movement; joystick base size is adaptive

## Common Edits

### Changing Difficulty Balance
Modify `DIFFICULTIES` object (line 485). Each property affects spawn timing, enemy speed, and boss health.

### Adding a New Weapon Type
1. Add new type to weapon colors and logic in the player object
2. Update weapon rendering in `draw()` to show the new projectile
3. Add SFX entry in `sfx` object

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508) with name, colors, and hpMul
2. Boss spawn logic in `update()` will automatically cycle through them
3. Adjust boss health scaling via the `hpMul` factor

### Adjusting Game Speed
- Modify `KILLS_PER_STAGE` (line 491) to change when bosses spawn
- Adjust enemy spawn rates in `DIFFICULTIES`
- Tweak player/enemy speed multipliers in their update logic

### Audio Tweaks
See `sfx` object (line 624) for all synthesized sounds. Tweak frequency, duration, filter, and volume for each SFX.

## Testing Checklist

When making changes to core gameplay:
- [ ] Test desktop mouse and keyboard controls
- [ ] Test mobile touch controls (joystick movement)
- [ ] Play at least one full stage on each difficulty
- [ ] Verify boss encounters work correctly
- [ ] Check that score/best score persist across sessions
- [ ] Confirm particle effects and visual feedback on enemy kills
- [ ] Test mute and motion-reduction toggles
- [ ] Monitor performance: no obvious frame drops on decent hardware

## Debugging Tips

- Use browser console (F12) to inspect or modify game variables (e.g., `bestScore`, `running`, `lives`)
- Temporarily lower `KILLS_PER_STAGE` to reach boss encounters faster
- Check localStorage in DevTools for persistence issues
- Audio context may be suspended on some browsers—check audioCtx state in console
- Use performance profiler to identify bottlenecks in `update()` or `draw()`

## Accessibility & Compliance

- Screen reader announcements via `announce()` function and `srLive` element (ARIA live region)
- Reduced motion preference detected via `prefers-reduced-motion` media query; disables shake, flash, and vignette
- Touch controls auto-enable on mobile; keyboard controls always available
- High-contrast text with text-shadow for readability on dynamic background
