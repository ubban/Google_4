# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file. The game features a player aircraft, procedurally spawned enemies, boss encounters, a weapon progression system, and power-ups. All gameplay, audio synthesis, and rendering happen in the browser with no external dependencies or build step.

The entire game (HTML, CSS, JavaScript) is in `index.html` (~1,670 lines).

## Running the Game

Simply open `index.html` in a web browser. No build process, server, or dependencies required.

For development/testing:
- Open the file directly in a browser
- Or use a local HTTP server: `python3 -m http.server 8000` (then visit http://localhost:8000/index.html)

## Code Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) containing:

### 1. **UI & DOM Setup** (~lines 426–461)
- Canvas and overlay element references
- HUD elements (score, health, weapons, bombs)
- Input controls (keyboard, mouse, touch)

### 2. **Configuration Constants** (~lines 463–515)
- `STORAGE_*`: LocalStorage keys for persistence (best score, muted state, difficulty, reduced motion)
- `DIFFICULTIES`: Three difficulty presets (easy/normal/hard) with spawn rates, enemy behavior, lives, bombs
- `KILLS_PER_STAGE`: Threshold (18 kills) to trigger boss encounter
- `THEMES`: Six visual themes with sky gradients and cloud/sun colors
- `ENEMY_TYPES`: Three enemy classes (grunt, interceptor, tank) with HP, speed, score value
- `BOSSES`: Six boss designs with unique visual styling and HP multipliers
- `BOSS_PATTERNS`: Attack patterns for each boss (6 patterns per boss, ~10 different attack functions)

### 3. **Audio System** (~lines 542–676)
- Web Audio API synth with no external audio files
- `beep()`: Generates pitched tones with optional frequency sweep, filtering, panning
- `noiseBurst()`: Generates filtered noise for explosion and bomb sounds
- `sfx` object: Named sound effects (shoot, laser, explode, hit, bomb, powerup, bossTelegraph, etc.)
- Dynamically adjusts loudness via ADSR envelope (attack-decay-sustain-release patterns)

### 4. **State Management** (~lines 863–870)
- **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Stats**: `score`, `lives`, `maxLives`, `bombCount`, `maxBombs`, `stage`, `stageKills`, `chain`, `multiplier`
- **Input state**: `keys`, `joyVec`, `mouseTarget`, `firing`, `mouseActive`
- **Rendering state**: `frame`, `shakeTime`, `shakeMag`

### 5. **Input Handling** (~lines 767–861)
- **Keyboard**: Arrow keys / WASD for movement; Space to fire; X/Shift/P/Esc for bomb/pause
- **Touch**: Left 55% of screen (joystick) for movement; right 45% has FIRE and BOMB buttons
- **Mouse**: Hover position steers player; click to fire
- Input priority: keyboard/joystick > mouse steering (prevents fighting)

### 6. **Game Loop** (~lines 1640–1665, 1148–1302)
- `loop()`: Called via `requestAnimationFrame`, 60 FPS target
- Each frame: update game state, then draw
- `update(dt)`: Advances all objects (player, enemies, bullets, boss)
- `draw()`: Renders scene with sky gradient, clouds, all sprites, particles

### 7. **Player Mechanics** (~lines 877, 1124–1146)
- Position-clamped to canvas bounds
- Two weapons: "vulcan" (spread bullets, 3 levels) and "laser" (pierce bullets, 3 levels)
- Weapon switching and level progression via power-ups
- Invulnerability frames (1.5s) after taking damage

### 8. **Enemy System** (~lines 502–506, 1022–1041, 1210–1228)
- Three types defined in `ENEMY_TYPES`
- `spawnEnemy()`: Randomly selects type based on stage, assigns behavior pattern (spread/aimed fire)
- Spawn rate decreases over time (difficulty ramps up)
- Enemies fire bullets at player or in spread pattern; different types have different speeds/HP
- Tank enemies show health bar above them

### 9. **Boss System** (~lines 1043–1056, 1372–1439, 1441–1454)
- One boss per stage, cycles through `BOSSES` array
- Boss design determines visual style and attack pattern pool
- Attack phases: telegraph (visual warning) → fire pattern
- `BOSS_PATTERNS`: 6 attack functions per boss (radial, aimed spreads, columns, spirals, homing, crosses, double rings, etc.)
- Boss HP scales with stage and difficulty; pattern rotates through available attacks

### 10. **Collision Detection** (~lines 1069–1071, 1232–1302)
- `rectsOverlap()`: Simple AABB (axis-aligned bounding box) check
- Bullets vs enemies → damage/knockback/particles/score
- Bullets vs boss → HP reduction/particles
- Enemies/boss vs player → damage/invulnerability
- Enemy bullets vs player → damage
- Power-ups vs player → stat changes

### 11. **Effects & Polish** (~lines 697–730, 1073–1085, 1507–1518)
- **Screen shake**: Triggered on hits/bombs, respects reduced-motion preference
- **Flash effects**: Hit flash (white) and bomb flash (longer duration)
- **Particles**: Colorful burst from explosions (count scales with performance quality)
- **Clouds**: Parallax scrolling background
- **Combo pop-ups**: Show kill counts at landmarks (10, 20, 30...)
- **Stage banners**: Announce stage transitions and boss arrivals
- **Vignette**: Red pulsing edge effect when at 1 HP (critical state)

### 12. **Performance Adaptation** (~lines 473–483)
- `quality` variable (0=full, 1=medium, 2=low) adjusts dynamically
- Tracks frame time; if avg < 1/56 FPS, reduce quality (fewer particles, less glow, lower shadow blur)
- If avg > 1/40 FPS, increase quality
- Glowing effects, shadows, and particle counts scale with quality level

### 13. **UI & Overlays** (~lines 752–765, 1491–1504, 1469–1489)
- Canvas auto-scales to fit window (max 1.6× base size)
- Pause screen (P/Esc key)
- Game Over overlay with score, stage, best score
- Start screen with difficulty selection (Easy/Normal/Hard)
- HUD shows: score, best, multiplier, weapon name/level, health hearts, bomb count, boss health bar

### 14. **Persistence** (~lines 517–527)
- LocalStorage keys: best score, muted state, difficulty, reduced motion preference
- Best score and user preferences survive page reloads

## Key Design Patterns

1. **Framerate-independent movement**: All movement multiplied by `dt` (delta time)
2. **Spawn rate progression**: Enemy spawn interval decreases gradually over time (line 1201)
3. **Kill chain multiplier**: Combo timer resets on 2.2s inactivity; multiplier caps at 10× (lines 1087–1101)
4. **Difficulty presets**: Single object with all tuning numbers (spawn, enemy speed, shoot chance, lives, bombs, boss HP)
5. **Damage multiplier architecture**: Boss patterns use different bullet counts and speeds rather than hardcoded damage values
6. **Theme cycling**: Visual themes rotate per stage; chosen from THEMES array by `(stage - 1) % THEMES.length`
7. **Graceful degradation**: Reduced-motion mode disables all animations when OS requests it

## Modifying Gameplay

### Add/tweak difficulty
Edit `DIFFICULTIES` object (lines 485–489). Each preset controls: spawn rate, enemy speed, shoot chance, starting lives, bombs, boss HP multiplier.

### Add/modify enemy types
Edit `ENEMY_TYPES` (lines 502–506) or add a new key. Must define: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`.

### Add/modify boss designs
Add to `BOSSES` array (lines 508–515) with new color scheme and HP multiplier. Corresponding attack pattern array will use the same index.

### Add/modify boss attack patterns
Edit `BOSS_PATTERNS` (lines 1372–1409). Each boss has an array of 3 attack functions. Add new functions (e.g., `fireChain`, `fireWave`) to the helper functions and reference in the pattern array.

### Adjust visual themes
Edit `THEMES` array (lines 493–500). Each theme defines: sky gradient colors, cloud/sun colors, and a display name.

### Add/modify power-ups
Edit `maybeSpawnPowerUp()` (lines 1058–1067) for spawn chance, and `applyPowerUp()` (lines 1304–1322) for effects.

### Tweak audio
Edit individual functions in the `sfx` object (lines 624–676). Parameters: frequency, duration, type (oscillator or noise), volume, options (sweep target, filter, pan).

## Important Implementation Notes

- **Radial/rotating bullets**: Boss bullets use `dx`/`dy` velocity per frame (not x/y position). Player bullets move vertically with optional `vx` spread (line 1188).
- **Weapon pierce**: Laser bullets have pierce count (decrementing on hit); vulcan bullets are consumed on hit.
- **Boss entering state**: Boss teleports in from top (y < 120), skips collision/attacks until `entering: false`.
- **Canvas coordinates**: Origin (0,0) is top-left. Y increases downward. Player starts near bottom.
- **Touch joystick**: Uses 50px radius constraint; clamped to [-1, 1] range per axis.
- **Reduced motion**: Applies to animations (keyframes) via CSS class and also disables screen shake (line 728).

## File Structure

```
index.html  – Complete game (HTML + CSS + JavaScript)
README.md   – Project description
CLAUDE.md   – This file
```

## Quick Reference: Key Functions

| Function | Purpose |
|----------|---------|
| `resetGame()` | Initialize new game state |
| `update(dt)` | Advance game logic each frame |
| `draw()` | Render all sprites and UI |
| `spawnEnemy()` | Add a random enemy |
| `spawnBoss()` | Trigger boss encounter |
| `fireWeapon()` | Player shoots based on weapon type |
| `useBomb()` | Smart bomb (clears enemies/bullets, damages boss) |
| `addScore(base)` | Add points with multiplier and chain logic |
| `killBoss()` | Handle boss defeat, advance stage |
| `loseLife()` | Handle player damage |
| `endGame()` | Game over state |
| `rectsOverlap()` | Simple collision check |
| `spawnExplosion()` | Particle burst effect |

## Testing Tips

- **Change difficulty mid-game**: Edit `difficulty` variable in dev console or comment out difficulty button prevention (line 747)
- **Spawn a boss**: Set `stageKills = KILLS_PER_STAGE` in console
- **Test boss patterns**: Modify `attackTimer` or `telegraph` in the boss object
- **Performance profiling**: Check `quality` variable; monitor `frameTimes` array for avg frame duration
- **Audio debugging**: Set `muted = false` in console if it's stuck on true
