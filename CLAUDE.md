# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a self-contained HTML file. The game features:
- Dynamic difficulty levels (Easy, Normal, Hard)
- Multiple enemy types with varying behavior patterns
- Boss encounters with distinct attack patterns
- Weapon progression system (Vulcan gun → Laser)
- Procedurally themed stages (6 visual themes)
- Web Audio API synthesis for all sound effects (no audio files)
- Full touch/mobile and keyboard/mouse support
- Responsive canvas rendering with adaptive quality

## Architecture

All code is contained in a single HTML file (`index.html`, ~1670 lines) organized as an IIFE (Immediately Invoked Function Expression). The architecture follows these layers:

### Game State & Systems
- **Input System**: Handles keyboard, mouse steering, touch joystick, and button inputs. Mouse input takes priority when active; keyboard/joystick always overrides mouse. Touch controls provide a virtual joystick and button interface for mobile.
- **Audio System**: Web Audio API synthesizer creating procedural sound effects via oscillators, noise bursts, and filtered envelope modulation. Master gain and compressor provide dynamic range management. Muted state is persisted in localStorage.
- **Collision & Physics**: Rectangular AABB collision detection, velocity-based movement, and simple physics for particles/enemies.
- **Entity System**: Structured as plain objects (player, enemies, bullets, boss, particles, power-ups) with properties updated each frame.
- **Rendering Pipeline**: 
  1. Clear canvas and apply screen shake
  2. Draw sky gradient + sun + clouds (themed per stage)
  3. Draw all game entities (player, enemies, boss, bullets, power-ups, particles)
  4. Mouse pointer crosshair (when mouse active)
- **Game Loop**: `requestAnimationFrame` at 60fps. Timestep-based updates (delta time clamped to 50ms). Adaptive quality system scales particle counts and glow effects based on frame time.

### Data Structures
- **DIFFICULTIES**: 3 presets (Easy/Normal/Hard) with spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP multipliers
- **ENEMY_TYPES**: 3 types (Grunt/Interceptor/Tank) with size, HP, speed multiplier, color, score value, and stage unlock threshold
- **BOSSES**: 6 boss designs with unique color schemes and HP multipliers
- **THEMES**: 6 stage themes with sky gradients, cloud/sun colors, and names
- **BOSS_PATTERNS**: 6 attack pattern arrays, one per boss (18 total patterns with named functions like `fireRadial`, `fireColumns`, `fireHomingSeeded`)

### Score & Progression
- Kill count (`kills`) and stage advancement: spawn boss when `stageKills >= KILLS_PER_STAGE` (18)
- Combo system: kills within 2.2 seconds build chain, capping at multiplier 10x (1 + floor(chain/5))
- New best score tracked in localStorage, shown on game over screen with trophy icon

## Common Development Tasks

### Running the Game
- **Web**: Open `index.html` directly in a browser (no build/server needed)
- **Development**: Edit the file and refresh browser; changes take effect immediately

### Testing the Game
- Test on desktop with keyboard (WASD/Arrows) and mouse steering
- Test on mobile: touch joystick and fire/bomb buttons
- Test difficulty progression: confirm stage bosses spawn at correct kill count (18)
- Test collision: enemies, bullets, power-ups should overlap smoothly
- Test audio: mute button, Web Audio context resume on first interaction

### Canvas Size & Responsiveness
- Canvas is positioned at the center of the viewport with `max-width: 100%` and `max-height: 100%`
- Responsive sizing via `clamp()` for HUD text (e.g., `clamp(12px, 2.8vw, 16px)`)
- Safe-area insets for notched/folded displays: `env(safe-area-inset-top)`, etc.
- Touch actions disabled globally (`touch-action: none` on canvas and body)

### Adding Enemy Behavior
1. Add new type to `ENEMY_TYPES` with size, HP, speed multiplier, and color
2. Enemies spawn in `spawnEnemy()` with random patterns: `'spread'` (bullets spread outward) or `'aimed'` (bullets track player)
3. Enemy bullet logic: shoot chance per difficulty, aimed bullets calculate direction to player each frame
4. Update `spawnEnemy()` logic to select new type by stage unlock (`minStage`)

### Adding Boss Patterns
1. Add new pattern function: `function fireXyz(boss, args) { enemyBullets.push(...) }`
2. Pattern functions manipulate `boss` coordinates and push entries to global `enemyBullets` array
3. Add function name to corresponding element of `BOSS_PATTERNS` array (indexed by boss design)
4. Boss telegraph (white flash before attack) is triggered by `boss.telegraph = 22`

### Modifying Game Difficulty
- Edit `DIFFICULTIES` object properties: `spawnBase` (enemy spawn interval base), `spawnMin` (fastest spawn rate), `enemySpeed` (enemy velocity multiplier), `enemyShootChance` (probability enemy fires), `lives` (starting hearts), `bombs` (starting smart bombs), `bossHpMul` (boss health multiplier)
- Difficulty is persisted in localStorage and recalled on page load

### Adjusting Audio
- `beep(freq, dur, type, vol, opts)`: Synthesis oscillator with optional frequency sweep, filter, pan, and delay
- `noiseBurst(dur, vol, opts)`: White noise with optional filter
- `sfx` object holds pre-configured sound effects (e.g., `sfx.shoot()`, `sfx.bossTelegraph()`)
- Mute state gates audio output and is persisted in localStorage

### Screen Shake & Visual Feedback
- `screenShake(magnitude, duration)`: Applies random translation to canvas each frame for specified time
- `flashHit()` / `flashBomb()`: Triggers white screen flash animation via CSS keyframes
- Vignette (red pulsing ring) appears when player has 1 life
- All motion effects can be disabled via "reduced motion" button (respects OS preference + localStorage)

### Scoring & Combo System
- `addScore(base)`: Increments kill chain and multiplier (caps at 10x). Multiplier persists 2.2s between kills.
- `breakChain()`: Resets combo when player takes damage
- Score color: yellow (#ffd54f); multiplier color: orange (#ff8a65)

### Accessibility
- Screen reader announcements via `#srLive` (ARIA live region)
- Keyboard & mouse control equivalents for touch buttons
- Color contrast tested for clarity
- Motion effects can be reduced/disabled
- Icons paired with text labels

## Storage Keys (localStorage)
- `skyfighter.bestScore`: High score as integer
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Motion effects toggle ('0' or '1')

## Performance Considerations
- **Adaptive Quality**: Frame time tracking (`frameTimes` array) auto-adjusts quality level 0-2. Higher quality = more particles, stronger glow effects, shadow blur.
- **Particle Pooling**: Particle counts scale with quality (`quality === 0 ? 1 : quality === 1 ? 0.6 : 0.35`)
- **Collision Checks**: Simple AABB (`rectsOverlap`) is fast enough for typical enemy/bullet counts
- **Canvas Transforms**: Batch transformations where possible (screen shake applied once per frame via `ctx.translate`)
- **localStorage**: Reads on initialization, writes only on state changes (mute, difficulty, best score)

## Browser APIs Used
- **Canvas 2D Context**: Drawing, gradients, transformations, text rendering
- **Web Audio API**: Oscillators, gains, compressor, filters, panners for synthesized sound
- **localStorage**: Persistent game state
- **requestAnimationFrame**: Game loop timing
- **matchMedia**: OS preference for reduced motion detection
- **Touch Events**: Mobile joystick and button controls
- **Mouse/Keyboard Events**: Input handling

## Key Variables & Constants
- `player`: Object with x, y, width, height, speed, weapon, level, cooldown, rapid-fire, invulnerability timer
- `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`: Arrays of entity objects
- `boss`: Current boss object (null when no boss active)
- `stage`, `kills`, `stageKills`: Progression tracking
- `score`, `multiplier`, `chain`, `chainTimer`: Scoring system
- `running`, `paused`, `gameOver`: Game state flags
- `quality`: Adaptive quality level (0 = high, 1 = medium, 2 = low)
- `shakeTime`, `shakeMag`: Screen shake parameters
- `frame`: Global frame counter (increments each update)

## Debugging Tips
- Open DevTools Console to set `quality = 0` and reduce spawning with `KILLS_PER_STAGE = 99999` for slower progression
- Log entity counts: `console.log(enemies.length, bullets.length, enemyBullets.length)`
- Check collision manually: `rectsOverlap(player, enemy)` in console
- Audio context state: `audioCtx.state` (should be 'running' after first click)
- Performance: Monitor frame time with `trackPerf(dt)` and check `quality` variable
