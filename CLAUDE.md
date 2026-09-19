# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in a single HTML file with embedded CSS and JavaScript. It runs in any modern web browser with no external dependencies or build process.

## Running the Game

- **Direct browser**: Open `index.html` in any modern web browser
- **Local server**: For best results with some browser features, serve via HTTP:
  ```bash
  python3 -m http.server 8000
  ```
  Then navigate to `http://localhost:8000/index.html`
- **Testing**: No test suite exists; validate changes by playing the game and checking:
  - Game starts and displays correctly
  - Controls work (keyboard on desktop, touch on mobile)
  - Enemies spawn and move
  - Shooting and bomb mechanics function
  - Score tracking and best score persistence work
  - Audio plays (if not muted)
  - All three difficulty levels are selectable and change gameplay

## Code Architecture

The `index.html` file contains ~1,670 lines organized into distinct sections:

### Structure (in order):

1. **HTML Markup** (lines 1-420): Game container, canvas, HUD elements, overlay UI, touch controls
2. **CSS Styles** (lines 10-359): Responsive design, animations, light/dark mode support, touch UI
3. **JavaScript Game Engine** (lines 422+):
   - **Configuration Objects**: Difficulty settings, enemy types, boss definitions, visual themes
   - **Game State**: Current score, lives, bombs, weapons, stage progression
   - **Audio System**: WebAudio synthesizer (no external audio files)
   - **Input Handling**: Keyboard, mouse, and touch input systems
   - **Rendering Pipeline**: Canvas-based sprite drawing and effects
   - **Game Loop**: Frame timing and adaptive quality scaling
   - **Entity Systems**: Player, enemies, bullets, bosses, effects
   - **Game Flow**: Menu overlay, stage progression, boss encounters, game over

### Key Game Systems

**Difficulty System**: Three presets (easy/normal/hard) control:
- Enemy spawn rates and minimum spawn interval
- Enemy movement speed and shooting frequency
- Player starting lives and bombs
- Boss health multiplier

**Weapon System**: 
- Player starts with Vulcan weapon (level 1)
- Weapon drops from killed enemies increment weapon level
- Weapon level displayed in HUD
- Affects projectile behavior and damage

**Scoring & Progression**:
- Base score from killing enemies (grunt=10, interceptor=15, tank=35)
- Combo multiplier increases for consecutive kills without being hit
- Best score persists in localStorage
- 18 kills per stage to progress; stages 1-6 with boss encounters

**Audio**:
- All sound via WebAudio API (no external files)
- Sounds for: gunfire, bomb detonation, boss defeated, hit, combo announcements
- Frequency sweep effects for dramatic moments
- Master gain with dynamic compression

**Visual Effects**:
- Screen shake on player hit (can be disabled via reduced motion setting)
- Flash overlay for hit/bomb effects
- Vignette (red pulsing) during critical health state
- Combo pop-up text
- Stage banner announcements
- Boss health bar

**Input Systems**:
- **Desktop**: Mouse aim (move to steer), keyboard (arrows/WASD), click/space to shoot, X/shift for bomb, P/esc to pause
- **Mobile**: Touch drag (left 55% of screen to fly), tap FIRE/BOMB buttons
- Adaptive joystick display on touch devices

**Storage Persistence** (localStorage keys):
- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion toggle (0/1)

**Visual Themes**: Six different sky/background themes cycling through stages:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

### Key Variables to Know

- `canvas` / `ctx`: Main game rendering canvas and 2D context
- `state`: Current game state object (score, lives, stage, enemies array, etc.)
- `quality`: Adaptive performance mode (0=full, 1=medium, 2=low quality) — auto-adjusts if framerate dips
- `difficulty`: Current difficulty name ("easy", "normal", "hard")
- `DIFFICULTIES`: Presets object with spawn/enemy/life parameters per difficulty
- `THEMES`: Array of background theme configurations
- `ENEMY_TYPES`: Object defining grunt, interceptor, tank properties
- `BOSSES`: Array of 6 boss definitions with HP multipliers and colors

## Making Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update spawn logic around line 1100 to include new type based on stage
3. Add drawing logic for the new type in the rendering section

### Modifying Weapon Behavior
1. Locate projectile firing logic (~line 1200)
2. Adjust projectile speed, spread, or damage based on `player.weaponLevel`
3. Update visual appearance (color, size) in the rendering section

### Changing Difficulty Balance
1. Edit `DIFFICULTIES` object constants (lines 485-489)
2. Adjust: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
3. Test by playing each difficulty level

### Adding a New Boss
1. Add to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul`
2. Update stage progression logic to introduce new boss at appropriate stage
3. Adjust `KILLS_PER_STAGE` if needed for pacing

### Adjusting Audio
1. Modify `beep()` parameters: frequency, duration, type ("sine", "square", "sawtooth", "triangle")
2. Use options like `sweepTo` for frequency ramps, `filterFreq` for lowpass, `pan` for stereo positioning
3. Adjust `masterGain.gain.value` to change overall volume

### Accessibility & Reduced Motion
- Reduced motion setting disables all CSS animations and screen shake effects
- Changes are controlled via `document.body.classList.toggle('reduced-motion', reducedMotion)`
- Screen reader announcements use `announce()` function
- Always test with reduced motion ON by clicking the 🚫 button

## Performance Considerations

- **Adaptive Quality**: Frame time is tracked; if average FPS drops below ~40, quality mode increases (reducing particle effects, enemy counts, etc.)
- **Canvas Optimization**: All drawing done to single canvas; no layering overhead
- **Enemy Culling**: Enemies off-screen are still processed; optimization opportunity here
- **Audio Context**: Lazily initialized on first sound (`ensureAudio()`) to avoid startup overhead

## Common Dev Tasks

**Test a specific difficulty**: Click difficulty button on start screen, then play
**Disable audio**: Click mute button (🔊) or check localStorage for `skyfighter.muted`
**Test on mobile**: Use browser DevTools device emulation (Ctrl+Shift+M in Chrome), or open on real device via local server
**Check for performance issues**: Open DevTools performance tab, record a gameplay segment, look for frame drops
**Test accessibility**: Enable reduced motion, use keyboard-only controls, check that screen reader announcements work
**Clear saved data**: In browser console: `localStorage.clear()` then reload
**Check current state**: In console: `console.log(state)` to inspect all game variables mid-game

## Browser Compatibility

- Requires modern browser with: Canvas 2D, Web Audio API, localStorage, ES6 syntax
- Tested in Chrome, Firefox, Safari (desktop and iOS)
- Mobile touch support: iOS Safari, Chrome Android

## Project Philosophy

This is a complete, self-contained game with no external dependencies. The single-file approach keeps the entire game logic visible and editable in one place. When making changes, consider:
- Keep animations performant (60fps target)
- Maintain responsive touch controls on all screen sizes
- Preserve accessibility features (reduced motion, screen reader support)
- Test game balance after difficulty or enemy changes
- Persist player preferences in localStorage for continuity
