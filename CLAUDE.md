# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application using Canvas and Web Audio API. The game features:
- Dynamic difficulty levels (Easy, Normal, Hard)
- Progressive stage system with boss encounters
- Weapon upgrade system (VULCAN, LASER, MISSILE types)
- Smart bomb mechanic
- Adaptive performance quality scaling
- Keyboard, mouse, and touch controls
- Procedurally generated themes with dynamic color schemes
- Full WebAudio synthesis for all sound effects (no external audio files)
- Reduced-motion support for accessibility

## Development & Running

**No build process required.** The game runs directly in any modern browser.

### Running the game locally
```bash
# Serve the file via any HTTP server (or open directly in browser for local testing)
python3 -m http.server 8000
# Then visit: http://localhost:8000
```

### Key development files
- `index.html` — single-file implementation (~1669 lines)
  - Embedded CSS (lines ~10-360)
  - Embedded JavaScript game logic (lines ~422-1669)
  - HTML structure & touch/keyboard control bindings

### Testing & validation
- **Manual testing**: Open in browser, test all three difficulty levels, verify touch/keyboard controls, check audio (click to unmute)
- **Accessibility**: Test with screen reader (HUD updates announced via `srLive` element); verify reduced-motion respects `prefers-reduced-motion`
- **Performance**: Game includes adaptive quality scaling that downgrades rendering at frame rates < 40 FPS

## Architecture & Code Organization

The entire game logic runs within an IIFE (Immediately Invoked Function Expression) starting at line 423. Key subsystems:

### Game State & Variables
- **Persistent storage**: Best score, mute state, difficulty, reduced-motion preference (via `localStorage`)
- **Difficulty constants** (`DIFFICULTIES`): Spawn rates, enemy speed, lives, bombs, boss HP multiplier
- **Themes** (`THEMES`): 6 visual themes with sky gradient, cloud, sun colors; rotated by stage
- **Enemy types** (`ENEMY_TYPES`): grunt, interceptor, tank with distinct stats (HP, speed, score value)
- **Bosses** (`BOSSES`): 6 unique boss variants (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with visual & HP characteristics

### Audio System (Synth-based, no external files)
- **Initialization**: `ensureAudio()` lazily creates WebAudio context on first interaction (respects autoplay policy)
- **Effects**:
  - `beep(freq, dur, type, vol, opts)` — oscillator with optional frequency sweep, filter, stereo pan
  - `noiseBurst(dur, vol, opts)` — white noise burst for impacts
  - `sfx` object maps game events to audio parameters
- **Master gain & compressor**: Applied to all audio for consistent loudness
- **Pan calculation**: `panFor(x)` maps screen X coordinate to stereo pan value

### Entity System
Three main entity types: **Player**, **Enemies** (bullets), **Explosions**
- Each entity is a simple object with position, velocity, hit tests
- Projectiles include weapon type (affects damage & sound)
- Enemies spawn continuously per stage; stop at `KILLS_PER_STAGE` (18 by default)
- Boss spawns on stage completion; kills boss to advance

### Weapon System
- **Three weapon types**: VULCAN (default), LASER (high spread), MISSILE (homing)
- **Upgrade mechanic**: Collecting weapon pickups increases current weapon level (max 4)
- **Weapon switch**: Click/tap FIRE button cycles weapons
- **Visual feedback**: HUD shows current weapon name & level

### Game Loop
- `gameLoop(t)` — runs at screen refresh rate via `requestAnimationFrame`
  - Input handling (keyboard, mouse, touch joystick)
  - Update phase: player, enemies, projectiles, bombs, screen shake
  - Render phase: canvas clear, background with dynamic theme, all entities
  - Performance tracking for adaptive quality

### Performance Adaptation (`quality` flag)
- Measures frame time over last 45 frames
- Downgrades rendering quality (disables certain effects) if avg FPS < 40
- Upgrades back when performance improves above 56 FPS
- Tracks in `frameTimes` array

### HUD & UI Updates
- Score, best score, multiplier, weapon info updated each frame
- Hearts display (❤️) for lives; bombs display (💣) for smart bombs
- Boss health bar appears when boss is active
- Stage banner, combo pop notifications
- Reduced-motion mode disables screen shake, flash, vignette pulse

### Control Systems
- **Keyboard**: Arrows/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move to steer, click to fire, right-click to bomb
- **Touch**: Joystick base on left, FIRE/BOMB buttons on right (full layout in `.touch-layout` CSS)

### Accessibility Features
- Screen reader announcements via `srLive` (aria-live region)
- Reduced motion support (disables animations & shake effects)
- Keyboard-navigable UI buttons
- High contrast design

## Important Constants & Values

| Constant | Purpose |
|----------|---------|
| `KILLS_PER_STAGE` | Enemies killed to trigger boss (default: 18) |
| `DIFFICULTIES` | Per-difficulty spawn rates, enemy behavior, starting lives/bombs |
| `ENEMY_TYPES` | Stats for grunt, interceptor, tank enemies |
| `BOSSES` | 6 boss variants with HP multipliers |
| `THEMES` | 6 visual themes (colors, sky gradients) |
| `quality` | Adaptive rendering level (0=full, 1=medium, 2=low) |

## Common Changes & Extensions

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update enemy spawn logic (around line 1200) to select new type based on stage & difficulty
3. Add corresponding explosion SFX variant if desired

### Adjust game difficulty
- Modify `DIFFICULTIES.normal` (or easy/hard) values: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

### Change visual theme rotation
- Edit `THEMES` array or modify theme selection logic (currently stage-based modulo themes count)

### Customize audio
- All sound effects generated via `beep()` and `noiseBurst()`; edit `sfx` object to tweak frequencies, durations, volumes

### Add new weapon type
1. Define weapon stats in weapon selection UI (around line 800)
2. Add projectile spawn logic with unique visual/behavior
3. Register in `sfx` object for unique firing sound

## Styling & Responsive Design

- **Canvas scaling**: Responsive via CSS `max-width/max-height`, centered in viewport
- **HUD layout**: Flexbox, responsive text sizing (`clamp()` for font scaling)
- **Touch controls**: Hidden on desktop, visible on mobile via media query
- **Dark theme**: Uses `:root { color-scheme: dark }`, dark blue background (#05122b)
- **Performance**: CSS animations only for non-reduced-motion mode

## Storage Keys

The game persists user preferences in `localStorage`:
- `skyfighter.bestScore` — highest score achieved
- `skyfighter.muted` — audio mute state (1 or 0)
- `skyfighter.difficulty` — last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` — reduced motion preference (1 or 0)
