# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shooter game built as a single self-contained HTML file. It features:
- Canvas-based 2D rendering with dynamic skies
- Procedurally generated enemy waves with stage progression
- Boss encounters (6 unique boss types)
- Web Audio synthesis for all sound effects (no external audio files)
- Mobile touch controls with adaptive joystick
- Difficulty levels (Easy/Normal/Hard)
- High score tracking and player preferences via localStorage
- Adaptive performance scaling to maintain 60 FPS

## Running the Game

- **Local development**: Simply open `index.html` in a modern web browser
- **Testing**: Works on desktop browsers (Chrome, Firefox, Safari, Edge) and mobile browsers
- **No build step required** — the entire game is self-contained in a single file

## Architecture & Key Systems

### High-Level Structure

The game is organized as a single-file IIFE (Immediately Invoked Function Expression) with these major systems:

```
UI Layer (HTML/CSS)
  ↓
Game Loop (requestAnimationFrame)
  ├→ Input Handling (keyboard, mouse, touch)
  ├→ Game State Updates (player, enemies, projectiles, effects)
  ├→ Canvas Rendering
  └→ Performance Tracking
```

### Game State Architecture

The game maintains several key state objects:

- **player**: Position, health, bomb count, weapon system, fire rate
- **enemies**: Pool of active enemies (grunts, interceptors, tanks)
- **projectiles**: Player bullets and enemy bullets
- **particles**: Explosions and visual effects
- **boss**: Current boss entity (when in boss stage)
- **waves**: Wave spawning configuration and progression

### Key Systems

1. **Difficulty System** (lines 485-489)
   - Three difficulty presets: Easy, Normal, Hard
   - Each preset controls spawn rates, enemy speed, shoot chance, starting lives/bombs, and boss HP multiplier
   - Set via localStorage and buttons in overlay UI

2. **Wave & Stage Progression** (constant KILLS_PER_STAGE = 18)
   - Enemies must be defeated to clear waves
   - Stages end with boss encounters
   - 6 themed sky backdrops cycle through stages
   - Boss difficulty scales with BOSSES array hpMul values

3. **Audio Synthesis** (lines 542-676)
   - **beep()**: Generates pitched tones with frequency sweeping, filters, panning
   - **noiseBurst()**: Generates white/filtered noise for impact sounds
   - **sfx object**: Pre-defined sound effects for all game events (shoot, explode, bomb, powerup, boss telegraph, etc.)
   - All sounds are synthesized in real-time; no external audio files
   - Mute state persists via localStorage

4. **Rendering Pipeline**
   - Background: Gradient sky with parallax clouds (cached gradient, updated per stage)
   - Game layer: Enemies, projectiles, player, particles rendered to canvas
   - UI overlay: HUD with score, lives, bombs, weapon level, multiplier, boss health bar
   - Mobile touch zone: Virtual joystick and fire/bomb buttons
   - Screen effects: Flash on hit/bomb, vignette flash on low health, combo popups, stage banners

5. **Adaptive Performance** (lines 473-483)
   - Tracks frame times over 45 frames
   - Quality levels: 0 (full), 1 (medium), 2 (low)
   - Adjusts particle count, effect detail, and rendering quality to maintain 60 FPS
   - Transparent to gameplay logic

6. **Weapon & Power-up System**
   - Player starts with basic gun
   - Enemies drop weapon power-ups (laser, scatter, rapid)
   - Weapons can level up (track separate level per weapon)
   - Multiplier increases on consecutive hits without being damaged

7. **Input Handling**
   - **Desktop**: Mouse move to aim, click/space to fire, X/Shift for bomb, P/Esc to pause
   - **Mobile**: Drag left zone to move, tap FIRE/BOMB buttons, or use virtual joystick
   - Joystick auto-hides on desktop, always visible on mobile
   - Touch handling includes dead zones and velocity

8. **Boss System** (line 508-515)
   - Boss triggered after KILLS_PER_STAGE enemy defeats
   - 6 unique boss types with distinct visual designs (hull colors, wings, core)
   - Boss health bar shown at top
   - Each boss has hpMul for scaling difficulty
   - Boss telegraph sound before attack patterns
   - Boss defeat triggers stage clear sound and next wave

9. **Visual Polish**
   - CSS animations: heart beat (low health), combo pulse, stage banner slide-in, card entrance
   - Performance-aware: animations respect reducedMotion preference
   - Responsive layout uses CSS clamp() for mobile/desktop scaling
   - Safe area insets for notched devices

## Modifying the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (around line 502) with properties: w, h, hp, speedMul, color, dark, score, minStage, optional weave flag
2. Update `EXPLODE_SFX` map if using custom explosion sound
3. Add draw logic in enemy rendering section
4. Enemy update logic is generic; weave behavior is template-based

### Adding a New Weapon

1. Define weapon properties in weapon definitions (look for weapon setup)
2. Add firing pattern in projectile generation
3. Add weapon power-up drop chance in enemy destruction
4. Update HUD weapon display (weaponNameEl updates)

### Adding Sound Effects

1. Add function to `sfx` object (lines 624-676)
2. Use `beep(freq, duration, type, volume, opts)` or `noiseBurst(duration, volume, opts)`
3. Options include: sweepTo, filterFreq, filterType, pan, delay
4. Call from event handlers; respects muted state and audioCtx initialization
5. Use `panFor(x)` to create stereo positioning based on sprite x-coordinate

### Adding Visual Effects

1. Particles are rendered generically; add particle emission in event handlers
2. Use CSS animations for UI (overlay, HUD, stage banner, combo popup)
3. Respect reducedMotion flag: check before applying animations
4. Use #flash, #vignette elements for screen effects

## Testing

Since this is a frontend-only game with no dependencies:

- **Functionality**: Open index.html and playtest each feature
- **Performance**: Use browser DevTools Performance tab to check frame rates
- **Mobile**: Test on actual devices or use browser device emulation for touch
- **Cross-browser**: Test on Chrome, Firefox, Safari, Edge
- **Difficulty balance**: Check that each difficulty feels appropriately challenging

## Performance Considerations

- The game targets 60 FPS but scales down if needed
- Canvas size is responsive; uses requestAnimationFrame for smooth animation
- Particle pools are recycled (no dynamic allocation during gameplay)
- Audio context is lazy-initialized on first game start or sound trigger
- Large object pools (enemies, projectiles, particles) prevent garbage collection pauses

## Browser Compatibility

- Requires: Canvas 2D context, requestAnimationFrame, localStorage, AudioContext
- Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Mobile: iOS Safari 14+, Chrome Android
- Note: Web Audio context may require user interaction (click) before sounds play (autoplay policy)

## File Structure

```
index.html (1669 lines)
  ├─ <head>: Metadata, title, favicon, CSS
  ├─ <body>: Canvas container, HUD elements, touch controls, overlay UI
  └─ <script>: 
      ├─ IIFE wrapper
      ├─ DOM references and storage keys
      ├─ Performance tracking
      ├─ Difficulty and constants (THEMES, ENEMY_TYPES, BOSSES)
      ├─ Audio system (beep, noiseBurst, sfx definitions)
      ├─ UI state (best score, mute, difficulty, reduced motion)
      ├─ Game loop and rendering
      └─ Event listeners (start, pause, difficulty select, mute, etc.)
```

## localStorage Keys

- `skyfighter.bestScore`: Player's high score
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0 or 1)

## Common Tasks

**Adjust difficulty values**: Edit `DIFFICULTIES` object (line 485-489)

**Add a new boss type**: Add entry to `BOSSES` array (line 508) with name, hull colors, wing color, core color, hpMul

**Change color scheme/theme**: Add new sky gradients to `THEMES` array (line 493-500)

**Modify spawn behavior**: Adjust `spawnBase` and `spawnMin` in `DIFFICULTIES`; higher spawnBase = slower spawns

**Test performance**: Open DevTools → Performance tab → record gameplay → check frame rate and look for long tasks

---

*Last updated: 2026-09-19*
