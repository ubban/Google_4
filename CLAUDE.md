# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a complete, self-contained browser game implemented as a single HTML file with vanilla JavaScript (no external dependencies). The game runs in Canvas and features:

- Player-controlled aircraft with multiple weapon types and levels
- Procedurally varied enemy types and spawning patterns
- Multi-stage progression with boss encounters
- Difficulty settings (Easy, Normal, Hard)
- WebAudio-based sound synthesis
- Full mobile touch support (drag-to-move joystick + on-screen buttons)
- Adaptive performance quality scaling
- Accessibility features (reduced motion support, screen reader announcements)
- Local storage for persistent best score and settings

## Development Workflow

### Running the Game

1. **Quick test**: Open `index.html` directly in a browser (file:// protocol or via a local server)
2. **Live server** (recommended for development): Use any local server:
   ```bash
   # Python 3
   python -m http.server 8000
   # Or Node.js (if available)
   npx http-server
   # Then open http://localhost:8000/
   ```

### Making Changes

- **All game code is in one file**: `index.html` (HTML, CSS, and JavaScript all in one)
- **No build step required**: Changes take effect immediately on page reload
- **Test in browser**: Use DevTools console to check for errors (`F12`)
- **Test both desktop and mobile**: 
  - Desktop: Arrow keys / WASD to move, Space/X to fire/bomb
  - Mobile: Simulated in DevTools (toggle device emulation) or real device

### Key Testing Scenarios

When making changes, verify:
1. **Game mechanics**: Start a game, progress through stages, reach a boss
2. **Different difficulties**: Test Easy, Normal, and Hard modes
3. **Audio**: Check both muted and unmuted states (WebAudio synthesis)
4. **Reduced motion**: Toggle motion effects on/off; ensure animations disable properly
5. **Mobile touch**: Test joystick dragging, fire/bomb button taps
6. **Screen shake & vignette**: Verify visual effects on player hit (reduced motion should minimize)
7. **High score persistence**: Play a game, close tab, reopen to verify best score saved
8. **Canvas resize**: Resize browser window; canvas should scale proportionally

## Architecture & Key Structures

### State Management

The game has two main states:
- **Menu/overlay state** (`running === false`): Start screen with difficulty selection, best score display
- **Active game state** (`running === true`): Game loop running, entities spawning/moving/colliding

Core game state variables:
- `score`, `mult` (multiplier), `combo` – player progression
- `lives`, `bombs` – player resources (persist until game over)
- `stage` – current level (increments when kill count reaches `KILLS_PER_STAGE`)
- `difficulty` – selects spawn rates, enemy speed, player health, boss HP multiplier
- `paused` – pause overlay (toggleable with P/Esc during gameplay)

### Game Loop Architecture

The main loop runs in `animationFrame`:
```
1. Update: dt-based movement, input handling, game state
2. Collision detection: bullet-enemy, bullet-boss, enemy bullet-player, bomb blast radius
3. Entity updates: enemies shoot, projectiles travel, spawn new enemies/bosses
4. Render: draw background, all entities, HUD stats
5. Performance tracking: adjust quality (particle/effect density) based on frame time
```

**Key variables**:
- `running` – game active flag
- `paused` – pause overlay shown
- `entities` – array of all game objects (player, enemies, bullets, effects)
- `spawner` – tracks spawn cooldown and difficulty curve
- `theme` – selected background gradient/visual style
- `quality` – performance level (0=full effects, 1=medium, 2=reduced for slow devices)

### Core Game Objects (Entities)

Each entity in the `entities` array has:
- `type` – "player", "enemy", "boss", "bullet", "bomb_blast", "floatText", etc.
- `x, y, vx, vy` – position and velocity
- `w, h` – bounding box width/height
- Additional fields depend on type (see entity spawning functions)

**Rendering happens via canvas 2D context** (`ctx`). Each entity's render method draws it; look for patterns like `ctx.fillStyle`, `ctx.fillRect`, `ctx.arc`, etc.

### Key Systems

#### Enemy Spawning
- `ENEMY_TYPES`: Defines stats (HP, speed, color, score, which stage they appear)
- `spawnBase, spawnMin`: Difficulty-dependent spawn rates (lower = more frequent)
- Enemies shoot at random intervals; `enemyShootChance` determines probability
- Tank enemies have higher HP and lower speed; interceptors are fast and weak

#### Boss System
- `BOSSES`: Array of 6 boss templates with colors, names, HP multipliers
- Triggered when `stage` advances and `killedThisStage >= KILLS_PER_STAGE`
- Boss enters from top, travels downward, fires patterns of bullets, has health bar
- Boss defeat clears stage and respawns normal enemies

#### Weapon System
- Player starts with **VULCAN** (rapid fire, weak)
- Pickup spawns after enemy kill; cycles through VULCAN → LASER → SPREADER
- Each level increases fire rate or spread angle
- Weapon level persists across enemy spawns until player dies

#### Audio (WebAudio Synthesis)
- No external audio files; all sounds generated procedurally via oscillators, noise bursts, filters
- `ensureAudio()` initializes context on first user interaction (browser autoplay policy)
- `beep()` – frequency sweep with oscillator type control
- `noiseBurst()` – filtered white noise for explosions, etc.
- `sfx` object maps sound names to functions; call `sfx.shoot()`, `sfx.explode(x)`, etc.
- Pan control uses `panFor(x)` to place stereo sound based on screen position

#### Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` – modulates camera offset
- **Flash**: `flashHit()`, `flashBomb()` – class toggles trigger CSS animations
- **Vignette pulse**: Red edge vignette pulses when player health critical
- **Combo/stage popups**: CSS animations for text popups (combo, stage clear messages)
- **Reduced motion**: When enabled, animations shortened/disabled; screen shake intensity reduced

#### Input Handling
- **Keyboard**: Arrow keys / WASD move, Space/X fire/bomb, P/Esc pause, motion/mute toggle
- **Mouse**: Move to crosshair position (when not on touch device), click to fire
- **Touch**: Joystick in left 55% of screen (drag to move), FIRE/BOMB buttons in right 45%

### Performance Optimization

- **Adaptive quality**: `quality` variable adjusted based on frame time smoothing
  - Triggers particle/effect density reduction on slow frames
  - Recovers to full quality when performance improves
- **Object pooling concepts**: Entities are created/destroyed dynamically; no large pools
- **Canvas resize**: Maintains fixed game canvas (480×720) scaled to fit window

### Visual Style & Themes

- `THEMES`: Array of 6 themes with different sky gradients, cloud colors, sun tones
- Selected randomly or cycled per stage for variety
- Color names are semantic (e.g., "Arctic Wastes", "Volcanic Ridge")

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (define sprite colors, HP, speed, score value)
2. Add to spawn logic (usually in `updateEnemies()` function or spawn check)
3. Implement render function: usually a rectangle with a darker stripe/center for visual interest
4. Add explosion sound if unique (add to `EXPLODE_SFX` map)

### Modifying Boss Behavior

1. Change `BOSSES` array for cosmetics (colors, names)
2. Locate boss update/render logic in `updateBoss()` / boss render block
3. Adjust `bossShootPattern()` for different bullet formations
4. Tweak `hpMul` for health scaling across difficulty levels

### Adjusting Difficulty Balance

- **Spawn rate**: Edit `DIFFICULTIES.*.spawnBase` and `spawnMin`
- **Enemy toughness**: Modify `ENEMY_TYPES.*.speedMul` and `hp`
- **Player health**: `DIFFICULTIES.*.lives` sets starting lives
- **Boss HP**: `DIFFICULTIES.*.bossHpMul` scales boss health

### Fixing Visual Glitches or Performance

- Check `quality` level in console: `quality` should adapt downward if FPS drops
- For rendering issues, inspect canvas drawing order (background → entities → HUD)
- Profile with DevTools Performance tab (record a game frame, check `updateGameState` and render calls)

### Adding Accessibility Features

- Use `announce()` to send screen reader messages (already done for some events)
- Test with reduced motion enabled: verify animations disable or drastically shorten
- Ensure keyboard controls work for all game actions (currently: arrows, space, X for bomb, P for pause)

## Important Implementation Details

### Storage Keys

Game persists to `localStorage`:
- `skyfighter.bestScore` – high score
- `skyfighter.muted` – audio mute state
- `skyfighter.difficulty` – selected difficulty
- `skyfighter.reducedMotion` – reduced motion preference

### Performance Considerations

- Canvas size fixed at 480×720 internally; DOM scaling handles window resize
- Particle/effect density controlled by `quality` variable
- Entity collision is bounding-box (AABB), not pixel-perfect
- Audio context only initialized on first user interaction (browser policy)

### Browser Compatibility

- Targets modern browsers (Chrome, Firefox, Safari, Edge)
- Touch support optional; falls back to keyboard/mouse gracefully
- WebAudio API required for sound; game plays silently without it
- LocalStorage required for persistence; game still works without it (no save)

## Common Patterns in Code

- **Entity lifecycle**: Create with `new` or push to `entities`, update in loop, remove when `dead` is set
- **Collision checks**: Iterate `entities` array, test bounding boxes with condition like `x < oX + oW && x + w > oX`
- **Rendering**: Push `ctx.save()`, modify transform/style, draw, then `ctx.restore()`
- **Audio**: Call `ensureAudio()` before any sound to initialize context; check `muted` flag before playing
- **State persistence**: Use `localStorage.setItem()` and `getItem()` with a consistent `STORAGE_*` key prefix
