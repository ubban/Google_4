# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game implemented in pure HTML5/Canvas/JavaScript. There are no build tools, no dependencies, and no external assets — the entire game is contained in `index.html` as a self-contained playable web page.

## Running and Testing

**No build step is needed.** Simply open `index.html` in a browser to play the game.

- Desktop: Use mouse to steer, click to fire, Space to fire, X/Shift to bomb, P/Esc to pause
- Mobile: Touch-driven controls with joystick on left, fire/bomb buttons on right
- The game saves best score, difficulty preference, and motion/sound settings to localStorage

## High-Level Architecture

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) inside a `<script>` tag. The architecture layers are:

### 1. **Game State & Configuration** (lines 463–527)
- Persistent settings stored in `localStorage`: best score, mute state, difficulty, reduced motion preference
- Difficulty presets defining enemy spawn rates, speeds, shoot chances, lives, bombs, and boss HP multipliers
- Constants for: 6 stage themes, 3 enemy types, 6 boss types, 18 enemy types → kills to trigger boss
- Global game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, stage, chain/multiplier, etc.

### 2. **Audio Engine** (lines 542–676)
- **Pure WebAudio synthesis** — no external audio files
- `ensureAudio()` initializes the AudioContext on first user interaction (required for browser autoplay policies)
- `beep()` generates frequency sweeps (used for weapon fire, boss hits, powerups)
- `noiseBurst()` generates filtered noise (used for explosions)
- Sound effects are precomputed functions in the `sfx` object (shoot, laser, explosions for each enemy type, hit, bomb, powerup, boss telegraph, stage clear, UI click, game over)
- All sounds use panning based on screen position for stereo directionality
- Global `muted` flag and master gain node control playback

### 3. **Input Handling** (lines 768–861)
- **Keyboard**: arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Touch**: left side is joystick movement, right side has FIRE and BOMB buttons
- **Mouse**: player glides toward pointer while hovering canvas; left-click to fire
- Mouse input is deprioritized when keyboard or joystick is active (no conflicts)
- Joystick tracking with touch ID to handle multi-touch correctly

### 4. **Rendering Pipeline** (lines 1520–1638)
- **Sky background**: gradient with sun glow (changes per stage theme)
- **Clouds**: two parallax layers with different speeds and opacity
- **Player**: drawn via `drawPlane()` with flame effect when firing
- **Enemies**: drawn via `drawPlane()` with type-specific colors; tanks show health bars
- **Boss**: custom `drawBossShip()` with design colors, animated core glow during attack telegraph
- **Bullets**: player bullets are rectangles (yellow vulcan or cyan laser); enemy bullets are circles (red)
- **Particle effects**: spawn on hit/explosion, fade out with gravity/velocity
- **Screen shake**: applied via canvas context translation when player takes damage or uses bombs
- **Flash effect**: white overlay flashes on player hit (bomb flash is longer)
- Rendering respects `reducedMotion` setting for accessibility (disables shake, glow shadows, animations)

### 5. **Game Update Loop** (lines 1148–1302)
Runs at ~60fps via `requestAnimationFrame()`. Each frame:
1. **Player movement**: integrate keyboard/joystick/mouse input, clamp to canvas bounds
2. **Weapon fire**: handle cooldown and fire rate (0.15s for vulcan, 0.1s for laser)
3. **Bullet physics**: simple linear movement (player bullets up, enemy bullets down)
4. **Enemy spawning**: difficulty-based spawning that ramps faster over time; spawn boss when stage kill target is reached
5. **Enemy AI**: 
   - Movement: weaving interceptors use sine wave; other types fall straight
   - Shooting: random shoot timer and pattern (spread or aimed at player)
6. **Collision detection**: 
   - Player bullets vs enemies (pierce mechanics for laser)
   - Enemy bullets vs player (causes life loss)
   - Enemies vs player (causes life loss)
   - Boss vs player (causes life loss)
   - Power-ups vs player (apply effects)
7. **Boss behavior**: separate `updateBoss()` function handles movement, attack telegraph, pattern firing
8. **Score system**: chain multiplier grows per kill and resets on player damage
9. **Stage progression**: kill threshold triggers boss spawn; boss defeat advances stage and resets enemies

### 6. **Boss AI & Attack Patterns** (lines 1324–1435)
Each boss type has 3 unique attack patterns in `BOSS_PATTERNS`:
- **VANGUARD**: radial burst, aimed fan, wide V-fan
- **BEHEMOTH**: heavy columns, slow homing spread, dense radial wall
- **SPECTER**: teleport + radial burst, tight spiral, double aimed shots
- **DREADNOUGHT**: 8-way cross, homing missiles, dense radial wall
- **WRAITH**: teleport + cross fire, expanding double ring, aimed cross-spread
- **TITAN**: heavy columns, full ring, relentless homing volley

Bosses telegraph their attack for ~20 frames (visual: core flashes white, sound effect plays). After attack fires, next attack is queued after ~85 frames.

### 7. **Power-Up & Weapon System** (lines 1304–1322)
- **Weapon types**: Vulcan (spread cannon) or Laser (single piercing beam)
- **Weapon levels**: 1, 2, 3 — each level increases bullet count or spread; level 4 = switch to other weapon
- **Power-up types**: Life (restore 1 HP up to max), Bomb (restore 1 bomb up to max), Weapon (level up or switch weapon)
- Spawn rate tuned so weapon upgrades are most common (34%), bombs (8%), lives (6%)

### 8. **UI & HUD** (HTML structure; updated in JS)
- Top HUD: Score, Best, Multiplier, Weapon name/level, Hearts (lives), Bombs
- Top right: Mute button, Reduced Motion toggle, Pause button
- Overlay: Intro menu with difficulty buttons, Game Over screen with stats
- Boss health bar: Shows name and HP fill when boss enters
- Stage banner: Animates on stage start and boss entry
- Combo pop-up: Displays on every 10 kills
- Screen vignette: Red pulse when HP is critical (1 heart)

## Key Design Decisions

1. **Single HTML file**: No tooling required; drag-and-drop deployment
2. **No external assets**: All graphics drawn with Canvas 2D, all audio synthesized with WebAudio
3. **Adaptive performance**: Quality setting (0/1/2) adjusts particle count and glow shadow effects based on frame time
4. **Touch-first controls**: Joystick + buttons work well on phones; mouse + keyboard support desktop play
5. **Accessibility**: Reduced motion preference respected, audio context only initialized on first interaction, screen reader announcements for key events
6. **Difficulty progression**: Stage increases spawn rate and enemy speed; bosses get HP multipliers
7. **Weapon diversity**: Two weapon types play very differently (spray vs precision), each with visual/audio feedback

## Code Organization Tips

- **Modify game constants** at the top: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
- **Adjust balance** by tweaking spawn timers, enemy speeds, shoot chances, and boss HP multipliers in `DIFFICULTIES`
- **Add new enemy types** by adding entries to `ENEMY_TYPES` and wiring up rendering in `enemies.forEach()`
- **Add new bosses** by extending `BOSSES` and `BOSS_PATTERNS` arrays (patterns are functions that fire enemy bullets)
- **Change visuals** (colors, particle effects, glow): search by color hex or `drawPlane()`/`drawBossShip()` function names
- **Audio tweaking** (SFX pitch/duration): modify `sfx` object function calls and `beep()`/`noiseBurst()` parameters
- **Canvas sizing** is responsive; change `BASE_W` / `BASE_H` to alter play area (currently 480×720)

## Testing & QA Checklist

When making changes, verify:
- Game starts without errors (check browser console)
- All three difficulty levels spawn/play correctly
- Player can move, fire, and bomb consistently
- Enemies spawn and enemies and bullets collide correctly
- Boss appears after stage kill target, follows attack pattern, dies on HP = 0
- Stage advances and theme changes after boss defeat
- Chain multiplier builds correctly and resets on damage
- Power-ups spawn and apply effects correctly
- Audio plays (use mute button to verify without sound)
- Reduced motion setting works (glow/shake/animations stop)
- Game saves/loads best score and settings from localStorage
- Touch controls work on mobile (if available)
