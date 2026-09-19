# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky Fighter is a complete Raiden-style vertical shoot-'em-up game built as a single HTML5 file. The game features canvas-based 2D rendering, procedurally generated sound effects via WebAudio API, and full game mechanics including boss battles, difficulty levels, and touch/keyboard controls.

**Key constraint**: This is a self-contained single-file application (index.html) with no build step, dependencies, or external assets. All CSS and JavaScript are embedded.

## Running the Game

Simply open `index.html` in a modern web browser. The game runs entirely client-side with no server required.

For development, you can:
- Edit the HTML file directly and refresh the browser
- Use a local HTTP server: `python -m http.server 8000` (or similar) and visit `http://localhost:8000/index.html`

## Architecture Overview

### High-Level Structure

The game uses a classic game loop pattern:
1. **Input handling** (keyboard, mouse, touch) → movement vectors and firing state
2. **Update logic** (dt-based) → physics, collision detection, game state changes
3. **Rendering** (canvas 2D) → draw all game objects to screen

All code runs within a single IIFE (Immediately Invoked Function Expression) to maintain encapsulation. Global state includes:
- Player object (position, health, weapon, cooldown)
- Arrays: bullets, enemies, enemy bullets, particles, power-ups
- Game state: score, lives, stage, difficulty, running/paused flags

### Key Game Systems

#### 1. Player and Movement
- Located at: Lines 875-877 (player object creation)
- Movement input: keyboard (arrows/WASD), mouse steering, joystick/touch
- **Important**: Keyboard/joystick always takes priority over mouse to avoid conflicts
- Collision boundary checking keeps player within canvas

#### 2. Weapon System
- Two weapon types: Vulcan (spread bullets) and Laser (piercing)
- Weapon upgrades increase level (1-3) or switch weapon type
- Level affects bullet spread/lane count
- Fire rate is weapon-specific (vulcan: 0.15s, laser: 0.1s)
- Line 1124-1146: `fireWeapon()` function handles weapon firing logic

#### 3. Enemy Spawning and Types
- Three enemy types in `ENEMY_TYPES` (lines 502-506):
  - **Grunt**: Fast, weak, score 10
  - **Interceptor**: Very fast with weaving, score 15
  - **Tank**: Slow, 4 HP, heavy armor visual, score 35
- Enemies spawn based on difficulty and stage progression
- Spawn rate increases over time via `difficultyRamp` (line 1201)
- Stage progression: 18 kills per stage trigger boss spawn (line 491)

#### 4. Boss System
- Six boss designs with unique visual appearance and attack patterns
- Each boss has a pattern pool with 3 different attacks
- Boss patterns (lines 1372-1409): radial bursts, aimed spreads, homing projectiles, etc.
- Boss telegraph (warning) system: 22-frame animation before attack fires
- HP scales with difficulty multiplier and boss-specific multiplier

#### 5. Collision and Damage
- `rectsOverlap()` (line 1069) handles all collision detection using AABB (axis-aligned bounding box)
- Player bullets vs enemies: damage check in lines 1232-1253
- Player bullets vs boss: damage check in lines 1255-1266
- Enemy/boss collision vs player: triggers life loss (lines 1268-1287)
- Power-up collection: lines 1289-1294

#### 6. Audio System
- **No external audio files** - all sounds are synthesized using WebAudio API
- `beep()` function: creates tones with optional frequency sweep and filters
- `noiseBurst()` function: creates noise-based sounds with filtering
- Sound effects defined in `sfx` object (lines 624-676)
- Stereo panning: based on screen position (`panFor()`, line 562)
- Mute toggle persists in localStorage

#### 7. Particle System
- Simple particle spawning for explosions via `spawnExplosion()` (line 1073)
- Particles have position, velocity, and lifetime
- Rendered as circles with alpha fade
- Quality-adaptive: spawn count scales with performance

#### 8. Difficulty System
- Three difficulties (easy/normal/hard) with distinct parameters (lines 485-489):
  - Spawn rate and minimum spawn interval
  - Enemy speed and shooting frequency
  - Boss HP multiplier
  - Starting lives and bombs
- Difficulty persists in localStorage

#### 9. Score and Progression
- Chain system: consecutive kills increase multiplier (1+ per 5 kills)
- Chain timer resets if player takes damage (line 1096)
- Boss kills award flat 500 points
- Multiplier displays in HUD and affects all score gains

#### 10. Visual Effects
- **Screen shake**: magnitude and duration (line 727-729)
- **Flash overlay**: hit flash (white) and bomb flash (white brighter)
- **Vignette**: red pulse when player health is critical (1 life remaining)
- **Reduced motion support**: respects `prefers-reduced-motion` media query and provides toggle (lines 520-540)

#### 11. Rendering System
- Canvas size: 480×720 (lines 753-764)
- Responsive scaling: canvas scales up to 1.6× based on window size
- Themes (6 color schemes): sky gradients, cloud colors, sun color (lines 493-500)
- Theme cycles per stage: `THEMES[(stage - 1) % THEMES.length]`
- Drawing order: background → player → enemies → bullets → particles → UI

#### 12. Input and Control
- **Keyboard**: Arrows/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire (takes precedence only when moving, keyboard has priority)
- **Touch**: Left zone for movement (joystick), right zone for fire/bomb buttons
- Touch joystick has max distance (50px) and shows visual base

#### 13. Storage and Persistence
- Best score in localStorage
- Difficulty preference
- Mute state
- Reduced motion preference (with system preference as fallback)

## Common Modifications

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508-515) with name and color scheme
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Boss index is determined by `(stage - 1) % BOSSES.length`, so it cycles

### Tweaking Difficulty
- Modify `DIFFICULTIES` object (lines 485-489)
- Key parameters: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

### Adding a Sound Effect
1. Create function in `sfx` object that calls `beep()` or `noiseBurst()`
2. Parameters for `beep()`: frequency (Hz), duration (s), wave type, volume, options object
3. Use `panFor(x)` to auto-calculate stereo pan from screen position
4. Call via `ensureAudio()` first to initialize WebAudio context

### Changing Game Balance
- Enemy spawn rate: adjust `spawnBase` and `spawnMin` in difficulty
- Enemy HP: modify `hp` in `ENEMY_TYPES`
- Weapon damage: change bullet pierce count or adjust enemy HP
- Boss HP: modify base value in `spawnBoss()` (line 1046) or difficulty multiplier

### Visual Theme Changes
- Sky colors: modify `sky` array in `THEMES` (RGB stops for gradient)
- Cloud/sun colors: RGB strings in theme objects
- Enemy/boss colors: change `color` and `dark` values in type definitions

## Performance Notes

- Adaptive quality system tracks frame time and scales particle count and shadow effects
- Quality levels: 0 (full), 1 (medium), 2 (low) - set by `quality` variable
- Shadow blur and glow effects disabled at lower quality tiers
- Particle count scales: `Math.max(3, Math.round((count || 14) * quality_multiplier))`

## Testing and Debugging

### Quick Testing Approaches
- Open DevTools console and modify variables like `score`, `lives`, `stage`
- Trigger boss spawn: set `stageKills = 18` and let spawn timer elapse
- Check localStorage: `localStorage.getItem('skyfighter.bestScore')`
- Performance: watch DevTools frame rate graph while gameplay happens

### Accessibility Features
- ARIA live region (`#srLive`) announces game events via `announce()` function
- Reduced motion toggle respects system preferences but allows manual override
- Screen reader friendly control hints
- High contrast colors for all UI elements

## File Structure

- `index.html` (1669 lines) - The entire game
  - Lines 1-361: HTML structure + embedded CSS styles
  - Lines 422-1666: Embedded JavaScript game engine

## Known Limitations & Constraints

1. **Single file architecture**: Any modification must stay within index.html
2. **Canvas size fixed**: 480×720 logical size with responsive scaling
3. **No external dependencies**: WebAudio and Canvas 2D only
4. **Mobile considerations**: Touch controls only on devices with touch support detection
5. **Browser support**: Requires ES6 JavaScript, Canvas 2D, and WebAudio API

## Future Enhancement Areas

- Additional boss patterns via procedural generation
- Leaderboard system (would require backend)
- Weapon customization between levels
- Mini-boss encounters before stage bosses
- Power-up combinations for combo effects
- Custom difficulty presets (player-defined)
