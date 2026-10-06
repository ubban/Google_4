# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. The entire game (1669 lines) is contained in `index.html` with embedded CSS and JavaScript — no build process or external dependencies required.

### Key Characteristics

- **Single-file architecture**: Complete game, styles, and logic in one HTML file
- **Procedural rendering**: All sprites drawn with canvas 2D API (no image assets)
- **Web Audio synthesis**: All sound effects generated with Web Audio API (no external audio files)
- **Responsive design**: Adapts to mobile (touch) and desktop (keyboard/mouse) controls
- **Adaptive performance**: Automatically reduces graphics quality on low-end devices
- **LocalStorage persistence**: Saves best score, audio mute state, and player preferences

## Game Architecture

### Core Game Loop

The main game runs in a `requestAnimationFrame` loop (line 1665) that:
1. **Input handling**: Polls keyboard/touch/mouse input
2. **Physics update** (`update(dt)`): Updates player position, enemy movement, bullet trajectories
3. **Collision detection**: Checks bullets vs. enemies, enemies vs. player, powerups vs. player
4. **Rendering** (`draw()`): Draws the scene with procedural sprites

### State Management

- **Game state variables** (line 864–869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`
- **Settings/persistence** (line 517–523): Best score, mute state, difficulty, reduced motion preference stored in LocalStorage
- **Difficulty presets** (line 485–489): Three tiers (`easy`, `normal`, `hard`) that adjust spawn rates, enemy speed, and boss HP

### Enemy System

- **Enemy types** (line 502–506): Grunt (basic), Interceptor (fast, evasive), Tank (heavy)
- **Spawning** (line 1022–1041): Random spawn location, type selection based on difficulty and stage progression
- **AI patterns**: Enemies weave horizontally, shoot either in spread or aimed-at-player patterns
- **Stage progression**: After 18 kills (`KILLS_PER_STAGE`), a boss spawns

### Boss System

- **Boss designs** (line 508–515): 6 unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with distinct color schemes
- **Boss patterns** (line 1372–1409): Each boss has 3 unique attack patterns (radial bursts, aimed spreads, columns, teleports, spirals, etc.)
- **Attack telegraph**: Bosses have a 22-frame wind-up period before firing, with visual/audio feedback
- **HP scaling**: Boss HP scales with stage and difficulty multiplier

### Weapon System

- **Vulcan**: Spread shot with 1–3 bullets depending on level
- **Laser**: Piercing shots (up to 3 levels) that can pass through multiple enemies
- **Weapon progression**: Collecting 3 weapon powerups levels up the current weapon; at level 3, collecting another switches to the alternate weapon

### Collision & Damage

- **AABB collision** (line 1069–1071): Simple overlap detection using width/height
- **Player invulnerability**: 1.5 seconds after taking damage
- **Enemy bullets damage player**: Break chain multiplier
- **Bomb powerup** (line 1103–1122): Clears all enemy bullets and kills all non-boss enemies, deals fixed 24 damage to boss

### Audio System

- **Web Audio synthesis** (line 542–676): Custom procedural sounds using oscillators and noise
- **Mutable** (line 732–739): Toggle stored in LocalStorage
- **Spatial audio**: Panning based on bullet/explosion x position
- **Sound effects**: shoot, laser, explode (grunt/interceptor/tank variants), hit, bomb, powerup, boss telegraph, boss death, stage clear, UI click, gameover

### Visual Themes

- **6 themes** (line 493–500): Cycled per stage—Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- **Procedural backgrounds**: Sky gradient, sun glow, parallax clouds
- **Dynamic lighting**: Glow effects on bullets and powerups in high-quality mode
- **Screen shake**: On bomb, boss hit, player damage, game over
- **Vignette pulse**: Activates when player at 1 health

### Performance Optimization

- **Adaptive quality** (line 474–483): Tracks frame times; quality levels 0 (full), 1 (medium), 2 (low)
- **Particle culling**: Particles reduced at lower quality levels
- **Shadow/glow disable**: At low quality, skip shadow blur
- **Reduced motion support** (line 520–540): Respects `prefers-reduced-motion` media query; can be toggled in-game

## Development Notes

### Single-File Structure

The entire game is in `index.html`. When making changes:
- **CSS** spans lines 10–359 (style tag)
- **JavaScript** spans lines 422–1666 (script tag)
- **HTML** spans lines 361–419 (DOM structure)

### Game Constants

- Canvas size: 480×720 (BASE_W, BASE_H at line 753)
- Kills per stage: 18 (line 491)
- Invulnerability duration: 1.5 seconds (line 1459)
- Difficulty multipliers stored in `DIFFICULTIES` object

### Input Handling

- **Keyboard**: Arrow keys or WASD for movement; Space to fire; X or Shift for bomb; P or Esc to pause
- **Mouse**: Move to steer (glide), click to fire
- **Touch**: Left zone (55% width) for joystick movement; right zone (45% width) for FIRE and BOMB buttons
- Input priority: Keyboard/joystick > Mouse (mouse never overrides active keyboard/touch)

### LocalStorage Keys

- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (`easy`, `normal`, `hard`)
- `skyfighter.reducedMotion`: User preference for motion reduction (1 or 0)

### Adding Features

When extending the game:
- **New weapon type**: Add to weapon label logic (line 871), fire rate (line 1177), fireWeapon() function
- **New enemy type**: Add to `ENEMY_TYPES`, update spawn logic in `spawnEnemy()`
- **New boss pattern**: Add function to fire bullets, add to `BOSS_PATTERNS` array
- **New sound effect**: Add to `sfx` object (line 624), use `beep()` or `noiseBurst()` helpers
- **New visual theme**: Add entry to `THEMES` array (line 493)

### Testing/Debugging

- Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- No local server required — works as a file:// URL
- Check browser console for any errors (Ctrl+Shift+J / Cmd+Option+J)
- LocalStorage can be cleared in DevTools > Application > Storage if resetting state
- Frame rate and performance visible in browser DevTools Performance tab

## How to Deploy

The game is ready to deploy as-is:
1. Push `index.html` to any static hosting (GitHub Pages, Netlify, etc.)
2. No build step, no dependencies, no environment variables needed
3. Game works offline once cached by browser

## Notable Implementation Details

- **No external assets**: All graphics drawn procedurally, all sounds synthesized
- **Collision system**: Simple AABB (axis-aligned bounding box) with overlap tolerance based on object sizes
- **Boss telegraph**: Visual feedback (white glow on core) and audio cue before boss attack
- **Chain multiplier**: Resets if no enemy killed within 2.2 seconds; multiplier caps at 10x (1 kill per 5 kills needed)
- **Accessibility**: Screen reader support via aria-live region (line 390), respects prefers-reduced-motion
- **Mobile detection**: `ontouchstart` or maxTouchPoints > 0 determines if touch controls are shown
