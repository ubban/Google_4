# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The entire game (HTML markup, CSS styling, and JavaScript logic) is contained in `index.html` (~1670 lines). It requires no build process or external dependencies—open the file directly in a browser to play.

### Key Characteristics
- **Canvas-based 2D rendering** using the HTML5 Canvas API
- **Procedurally generated audio** via WebAudio API (all sounds are synthesized in-game, no external audio files)
- **Responsive design** that scales to any screen size and adapts to mobile/touch input
- **Accessibility features** including screen reader announcements, reduced-motion preferences, and color-safe visuals
- **Persistent state** using browser localStorage for high scores and user settings

## Architecture

The game is built around several interconnected systems:

### Game Loop & Timing
The main game loop drives rendering and updates at ~60 FPS. Frame timing is tracked for adaptive performance—if the game can't maintain 60 FPS, visual quality settings (`quality` variable) are automatically reduced (enemy particle counts, shadow effects).

### Entity Systems
All game objects share a common pattern with `x`, `y`, `w` (width), `h` (height), and type-specific properties:

- **Player**: Single entity with position, velocity, weapon state, cooldown, invulnerability frames. Controlled by keyboard, mouse, or touch joystick.
- **Enemies**: Three types (grunt, interceptor, tank) with varying HP, speed, and behavior. Spawned in waves based on difficulty and stage.
- **Bullets**: Both player bullets and enemy bullets. Player bullets are fired in patterns based on the current weapon (vulcan spread or laser), enemy bullets rain down individually.
- **Particles**: Visual feedback for explosions, impacts, and effects. Pooled for performance.
- **Power-ups**: Drop from defeated enemies, grant weapon upgrades or rapid-fire bonuses.
- **Boss**: Special entity spawned after enough stage kills. Unique patterns and health pool per boss design.

### State Management
Game state is split into two layers:

1. **UI State** (overlay visible/hidden, difficulty selection, pause state)
2. **Play State** (running, paused, gameOver, stage/kills/score counters)

The `running` flag gates whether the game loop processes updates; `paused` gates rendering. Difficulty is stored in localStorage and applied at game reset.

### Input Handling
Three input methods coexist with priority-based precedence:

1. **Keyboard & Mouse** (always available): WASD or Arrow Keys to move, Space/click to fire, X/Shift/P to bomb/pause
2. **Touch Joystick** (on touch devices): Left side for movement, right-side fire button, bomb button
3. **Mouse Steering** (fallback): While hovering the canvas, the player glides toward the pointer

Keyboard/joystick input takes priority over mouse, so players don't fight the two modes.

### Rendering Pipeline
1. Clear canvas and apply screen shake offset
2. Render background gradient (theme-based)
3. Render all entities in order: enemies, player, bullets, particles, power-ups, effects
4. Render HUD overlays (hearts, score, multiplier, weapon info, boss bar)
5. Apply fullscreen effects (vignette when critical health, flash on hit/bomb)

### Audio System
All sound effects are synthesized using WebAudio oscillators, noise buffers, and filters:

- **Beeps**: Frequency sweep, optional filter, optional stereo pan
- **Noise bursts**: Filtered white noise with ADSR envelope
- Each effect (shoot, explode, hit, bomb, power-up, etc.) is a composition of beeps and/or noise

The `muted` flag gates all audio; the `masterGain` provides global volume control. Panning is calculated per-screen-x-position for stereo localization.

### Difficulty & Progression
Three difficulties (easy/normal/hard) adjust:
- Enemy spawn rates and minimum intervals
- Enemy speed and shoot accuracy
- Starting lives, bomb count, boss HP multiplier

Progression is wave-based: after killing `KILLS_PER_STAGE` (18) enemies, the stage advances, a new boss theme plays, and a new boss spawns. Stages 2+ introduce new enemy types.

### Scoring & Multiplier
- Base score per kill depends on enemy type
- A combo chain (`chain` counter) multiplies all subsequent kills
- The multiplier (`multiplier` variable) caps at 10x
- Combo breaks if too long passes without a kill; resets on new stage

## How to Modify

### Adding a New Enemy Type
1. Add an entry to `ENEMY_TYPES` with properties: `w, h, hp, speedMul, color, dark, score, minStage, weave` (optional)
2. Define spawn logic in the enemy-spawning section (search for `spawnTimer`)
3. Add collision/damage logic if the new type has special behavior
4. If needed, add a custom explosion sound in the `sfx` object and map it in `EXPLODE_SFX`

### Adding a New Boss
1. Add an entry to `BOSSES` with `name, hullTop, hullBottom, wing, core, hpMul`
2. Adjust the `drawBossShip()` function if the new boss needs a unique visual
3. If adding attack patterns, modify the boss update logic (search for `if (boss)` in the update section)

### Tweaking Difficulty
Edit the `DIFFICULTIES` object. Changes to `spawnBase`, `spawnMin`, `enemySpeed`, etc. immediately affect the next game started.

### Adding a New Sound Effect
1. Create a function in the `sfx` object using `beep()` and/or `noiseBurst()`
2. Call `ensureAudio()` before playing to initialize the WebAudio context
3. Respect the `muted` flag and `reducedMotion` preference

### Changing Visual Themes
Add entries to the `THEMES` array. Each theme specifies the sky gradient colors, cloud color, sun color, and a display name. The theme is chosen per-stage to cycle through all themes.

## Development Notes

### Single-File Constraint
The entire game is in one HTML file for portability and simplicity. This means:
- No module imports or bundling
- All code runs in a single global scope (wrapped in an IIFE to avoid polluting `window`)
- Refactoring is possible but keep the file structure intact; reading it end-to-end is necessary to understand dependencies

### Performance Considerations
- The adaptive quality system reduces rendering burden (fewer particles, disabled shadows) on slower devices
- Particle and entity pooling are minimal but feasible—consider adding object pools if frame rates drop
- Canvas shadow effects (on boss core) are expensive; `quality` setting controls them
- The game uses `requestAnimationFrame` for the main loop, so rendering is tied to the browser's refresh rate

### Browser APIs Used
- **Canvas 2D**: All graphics
- **WebAudio API**: All sounds (requires user gesture to initialize)
- **localStorage**: Persistent state (best score, difficulty, mute, reduced motion)
- **Touch Events**: Mobile input
- **Keyboard/Mouse Events**: Desktop input
- **Responsive**: Uses `window.innerWidth/Height`, media queries, `safe-area-inset` for notches

### Accessibility
- **Screen reader support**: HUD updates are announced via a live region (`#srLive`)
- **Reduced motion**: Honors `prefers-reduced-motion` and allows user override; disables animations, reduces vignette pulsing
- **Color contrast**: UI text uses high-contrast colors on dark backgrounds

## Testing the Game

Since there's no build step, simply open `index.html` in a web browser. On first load:
1. Click to enable audio
2. Select a difficulty
3. Click "Start Game" to begin

### Quick Play Testing
- **Keyboard**: Arrow keys to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move pointer over canvas to steer, click to fire
- **Touch**: Use on-screen joystick and buttons

### Debugging Tips
- Open DevTools (F12) and check Console for any errors
- Check Network tab to confirm no external files are requested
- Frame rate counter can be added by logging `1000 / dt` in the frame loop
- Inspect `player`, `enemies`, `bullets` in the console during gameplay to inspect state

## Common Edits

| Task | Location |
|------|----------|
| Adjust player speed | Line ~877, `player.speed` |
| Change enemy spawn rate | Line ~486–488, `DIFFICULTIES` |
| Modify weapon cooldown | Search for `player.cooldown` increment logic |
| Tweak combo timeout | Search for `chainTimer` updates |
| Adjust canvas size | Lines ~753 `BASE_W`, `BASE_H` |
| Add a new keyboard key | Lines ~769–776, add to `keydown` handler |
