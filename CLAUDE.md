# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 canvas application. No build system, no dependencies—just one `index.html` file containing the complete game (HTML, CSS, and JavaScript).

### Running the Game

Simply open `index.html` in a web browser. The game is fully playable on desktop (keyboard/mouse) and mobile (touch joystick). No server or build step required.

## Codebase Structure

The entire game lives in `index.html` (~1669 lines), structured as a series of interconnected systems:

### 1. **Game Configuration & Constants** (lines ~470–520)
- `DIFFICULTIES`: Defines easy/normal/hard modes (enemy count, speed, lives, bombs)
- `ENEMY_TYPES`: Defines grunt, interceptor, tank archetypes with HP, speed, scoring
- `BOSSES`: 6 boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with color schemes and health multipliers
- `THEMES`: 6 stage backdrop themes (sky gradient colors, cloud colors, sun colors)
- `BOSS_PATTERNS`: Attack pattern factories for each boss (radial, aimed, homing, column, spiral, V-spreads)

### 2. **Audio System** (lines ~542–676)
Synthesized WebAudio (no external sound files). Key functions:
- `ensureAudio()`: Initializes Web Audio context and master gain/compressor
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillator with optional pitch sweeps, filters, and stereo panning
- `noiseBurst(dur, vol, opts)`: White noise filtered and enveloped for explosions
- `sfx` object: Pre-configured sounds (shoot, laser, explosions by enemy type, hit, bomb, power-up, boss telegraph/death, stage clear, game over)
- Panning is driven by on-screen X position for spatial depth; compressor prevents clipping on overlapping sounds

### 3. **Input System** (lines ~767–862)
Supports keyboard, touch joystick, and mouse steering—each can be active, but keyboard/joystick takes priority.
- **Keyboard**: Arrow keys or WASD for movement; Space to fire; P/Escape to pause; X/Shift for bombs
- **Touch**: Left zone spawns a joystick; right buttons fire and bomb
- **Mouse**: Pointer over canvas glides plane toward it (capped at 1.7× normal speed); mouse button fires; touchpad is drawn as a crosshair

### 4. **Game State & Flow** (lines ~863–912)
Central state variables:
- `player`: Position, speed, cooldown, weapon type/level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`: Lists of active game objects
- `score`, `lives`, `stage`, `kills`, `stageKills`: Progress tracking
- `chain`, `chainTimer`, `multiplier`: Kill-chain meter (resets after 2 sec of no kills)

`resetGame()` initializes all state and renders HUD elements.

### 5. **Rendering System** (lines ~680–1577)
- `drawPlane()`: Player/enemy plane with fuselage, wings, tail, and optional flame
- `drawBossShip()`: Multi-part boss with hull gradient, wings, glowing core, and telegraph flash
- `draw()`: Main frame renderer—clears canvas, applies screen shake, draws sky/gradient/clouds/parallax, renders all game objects (player, bullets, enemies, boss, particles, power-ups), draws HUD overlays

Canvas is 480×720 base, responsive scaled via CSS to fit the window (max 1.6× scale).

### 6. **Collision & Damage** (lines ~1100–1200, inlined in update loop)
- Player bullets vs. enemies: Halve enemy HP; respawn if dead, add score
- Enemy bullets vs. player: Lose one life (if invulnerable timer expired)
- Player vs. power-ups: Upgrade weapon level, add bombs, or gain extra life
- Boss collision: Take 1 HP per player bullet

### 7. **Enemy & Boss Systems**
**Enemy update & spawn** (lines ~1300–1350):
- Waves spawn based on stage and kill count (grunts + interceptors early, tanks later)
- Each enemy moves toward bottom, wraps horizontally, shoots back every N frames

**Boss system** (lines ~1050–1440):
- Spawns after clearing stage wave (`stageKills >= STAGE_KILL_TARGET`)
- Enters from top, then patrols left–right
- Uses telegraph (22 frames white flash) before firing; attack patterns cycle through the boss's pattern set
- Boss health based on `hpMul × (100 + stage × 10)`

**Boss patterns** are small functions that fire bullets into `enemyBullets`:
- `fireRadial()`: Circular spread
- `fireColumns()`: Vertical streams
- `fireAimedSpread()`: Toward player with spread
- `fireDoubleRing()`: Two concentric circles
- `fireHomingSeeded()`: Bullets that curve toward player

### 8. **Score & Combo System** (lines ~1240–1270)
- `chain` increments each kill; `chainTimer` counts down 2 seconds
- `multiplier = floor(1 + chain / 3)`, capped at 5×
- Scores: Basic kill score × multiplier; boss = 500 fixed
- `breakChain()` on player damage or pause

### 9. **Visual Effects & Polish**
- **Screen shake**: `screenShake(mag, dur)` offsets canvas via transform; scaled down if reduced motion enabled
- **Flash**: Hit/bomb flash animations via CSS keyframes
- **Vignette**: Red pulsing overlay when health ≤ 1 (accessibility: dims if reduced motion on)
- **Combo popups**: Floating multiplier text on-screen
- **Parallax clouds**: Two scrolling cloud layers at different speeds

### 10. **Accessibility & Performance** (lines ~520–540, scattered throughout)
- **Reduced motion**: `reducedMotion` boolean (defaults to OS `prefers-reduced-motion`), toggleable via UI button; disables shake/flash/pulse animations
- **Screen reader**: `announce()` function posts to an aria-live region for life loss, boss defeats, game over
- **Performance**: `quality` tiers (0 = glow+shadow, 1 = glow only, 2 = none) based on rolling frame-time monitor; auto-downgrade if FPS drops
- **Mute & difficulty**: Persisted to `localStorage` with `STORAGE_*` keys

## Development Workflow

### Making Changes to Game Mechanics
1. **Edit `index.html`** directly—no build step
2. **Refresh the browser** to see changes immediately
3. **Test in multiple input modes**: keyboard, mobile touch, mouse

### Common Edits
- **Adjust enemy or boss stats**: Edit `ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`, or `BOSS_PATTERNS`
- **Change colors or visuals**: Edit the color strings in `THEMES`, drawing functions, or CSS rules (e.g., `.stat`, `#gameCanvas`)
- **Add or modify sound effects**: Edit `sfx` object; use `beep()` and `noiseBurst()` with different frequencies, durations, and filters
- **Tweak gameplay balance**: Adjust `STAGE_KILL_TARGET`, weapon cooldown, enemy spawn rate, boss health, or score values
- **Mobile/accessibility**: Test with `--disable-extensions` to avoid ad blockers; check both `window.matchMedia('prefers-reduced-motion')` mode and `prefers-color-scheme: dark`

### Debugging
The game has no console output by default, but:
- Open DevTools (F12) → Console tab to see any errors
- Use `console.log()` to trace game state in the update loop
- The canvas is 480×720; player position is relative to this (not window) coordinate space
- Collision tests are bounding-box (x ± w/2, y ± h/2)

## Key Design Decisions

1. **Single-file architecture**: No build, easy to distribute (just email or host the HTML)
2. **WebAudio synth**: No asset loading; full audio control with pitch/filter sweeps
3. **Canvas + CSS**: Canvas for game, CSS for HUD/overlay/pause menu
4. **Accessibility-first polish**: Reduced motion and screen reader support baked in from the start
5. **Adaptive quality**: Frame-time monitor automatically scales glow/shadow effects to keep mobile devices smooth
6. **Persistent state**: `localStorage` tracks best score, mute, difficulty, and accessibility preferences

## Performance Notes

- The game loop runs at `requestAnimationFrame` (~60 FPS on most devices)
- Performance tier downgrade kicks in if frame time exceeds ~20ms (quality tier 1) or ~30ms (tier 2)
- Heavy CPU cost: collision checks (bullets vs. enemies), particle spawning, audio synthesis on explosion—all run every frame
- If adding features: profile in DevTools (Performance tab) before and after to ensure mobile smoothness

## Testing Checklist for Changes

Before pushing a change, verify:
- [ ] Game starts without console errors
- [ ] Player can move and fire (keyboard + mouse + touch)
- [ ] Enemies spawn and die on hit
- [ ] Boss appears after stage wave and follows attack patterns
- [ ] Screen shake and flash work (or are muted if reduced motion is on)
- [ ] Sound plays (or is muted if audio is off)
- [ ] Score and combo updates display
- [ ] Game ends when lives hit 0
- [ ] Pause works (P key, pause button)
- [ ] Best score persists across page refresh
- [ ] No console errors in DevTools
