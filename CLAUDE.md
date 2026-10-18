# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 game. The entire game logic (rendering, game state, input handling, audio synthesis) is embedded in `index.html` as vanilla JavaScript with no external dependencies or assets.

## Architecture

### Single-File Structure
The game is self-contained in `index.html` (~1500 lines):
- **HTML** (lines 1-420): DOM structure, canvas, UI overlays, HUD elements, touch controls
- **CSS** (lines 10-358): Responsive layout, animations, dark theme styling
- **JavaScript** (lines 422+): Complete game engine

### Key Game Systems

#### 1. **Canvas & Rendering**
- Canvas size: 480×720px (BASE_W, BASE_H at top of script)
- Responsive scaling to fit window (max 1.6x scale factor)
- Uses requestAnimationFrame for 60fps game loop
- Adaptive quality system (`quality` variable) that reduces particle/effect rendering under frame-rate pressure

#### 2. **Game State & Flow**
- Main state variables: `running`, `paused`, `gameOver`
- Game progresses through stages: enemies spawn, each stage requires KILLS_PER_STAGE (18) kills to advance
- Boss fight triggers after stage completes
- Loss condition: player loses all lives; win tracked via best score in localStorage

#### 3. **Audio System** (Lines 542-676)
- **WebAudio API** synthesizer with no external audio files
- Master gain + dynamics compressor chain for polished sound
- Procedural SFX: `beep()` for tones, `noiseBurst()` for percussion
- Mute toggle persisted to localStorage (STORAGE_MUTE)
- Each game action has a unique sound function in `sfx` object (shoot, explode, bomb, powerup, bossTelegraph, etc.)

#### 4. **Input System** (Lines 767-861)
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer, click to fire (steering uses smooth glide toward pointer)
- **Touch**: Left zone dragging for movement (virtual joystick), right zone buttons for FIRE/BOMB
- Multi-touch support with touch ID tracking to prevent cross-contamination

#### 5. **Entity System**
- **Player**: Position, weapon type (vulcan/laser), weapon level, invuln timer, cooldown
- **Enemies**: Type (grunt/interceptor/tank), position, HP, movement vector
- **Bullets**: Fast projectiles from player (fire-and-forget, cleaned when off-canvas)
- **Enemy Bullets**: Slower projectiles from enemies
- **Particles**: Visual effects spawned on hits/explosions (cleaned automatically)
- **Power-ups**: Weapon upgrades spawned from dead enemies
- **Boss**: Special entity with large HP pool, phase-based attacks, visual telegraph

#### 6. **Game Config** (Lines 485-515)
- **DIFFICULTIES**: Easy/Normal/Hard with spawn rates, enemy behavior, lives, bombs, boss HP multiplier
- **THEMES**: 6 visual themes with sky gradient, cloud, and sun colors (applies to each stage sequentially)
- **ENEMY_TYPES**: `grunt`, `interceptor`, `tank` with different speeds, HP, and scores
- **BOSSES**: 6 bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique colors and HP multipliers

#### 7. **Score & Progression**
- Base score: fixed points per enemy type
- **Combo multiplier**: Increases (1→2→3→etc.) as you chain kills without breaks; resets on pause or when timer expires
- **Chain timer** (`chainTimer`): Counts down; kill extends it
- Best score saved to localStorage (STORAGE_BEST)

#### 8. **Collision Detection**
- Simple AABB (axis-aligned bounding box) checks between player/bullets and enemies
- Player invuln frame tracking post-hit to prevent instant re-damage

#### 9. **Accessibility**
- Screen reader support via `srLive` (aria-live="polite")
- Reduced motion preference detection and enforcement (STORAGE_REDUCED)
- Theme color, viewport fit for mobile

## Making Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~503) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add a case in the enemy spawn logic to select this type based on stage and difficulty
3. Add explosion SFX to `EXPLODE_SFX` map if custom sound desired (e.g., `tank: 'explodeTank'`)
4. Adjust `KILLS_PER_STAGE` if balance changes significantly

### Adding a New Boss
1. Add entry to `BOSSES` array (line ~508) with: `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul`
2. Boss rendering and attack pattern are procedurally generated; no per-boss logic branch needed
3. Adjust `hpMul` to tune difficulty (higher = tougher)

### Changing Difficulty or Balance
- Tweak spawn rates, enemy speeds, lives in `DIFFICULTIES[difficulty]`
- Adjust `KILLS_PER_STAGE` to change pace
- Modify weapon cooldown/damage in the fire logic (search for `player.cooldown`)

### Adding SFX
1. Write a new function in the `sfx` object using `beep()` and/or `noiseBurst()`
2. Call via `sfx.yourSoundName()` where needed
3. Use `opts.pan` for stereo positioning and `opts.delay` for sequencing multiple tones

### Visual Effects & UI
- Use `flashHit()` / `flashBomb()` for screen flash (respects `reducedMotion`)
- Use `screenShake(magnitude, duration)` for rumble
- Use `showComboPop()` or `showStageBanner()` for text overlays

## Testing & Verification

Since this is a game with no build step or tests:
1. Open `index.html` directly in a browser (file:// protocol works)
2. Test all three difficulty levels and stage progression
3. Verify boss phases trigger and enemies spawn correctly
4. Test all input methods: keyboard, mouse, touch (use browser DevTools device emulation for mobile)
5. Check audio mutes correctly and reduced-motion disables animations
6. Verify high score saves by opening DevTools → Application → LocalStorage

## Performance Notes

- Particle count and effect rendering scales with `quality` variable to maintain 60fps
- Collision checks are O(n²) but acceptable given typical enemy counts (<20 per frame)
- Canvas resize is debounced by browser resize event (safe for mobile orientationchange)
- WebAudio context resumes automatically on first interaction (user gesture required by spec)

## Responsive Design

- Canvas scales to fit viewport (max 1.6× original size) while maintaining 480×720 aspect ratio
- HUD uses CSS clamp() for font sizes that adapt to viewport
- Safe-area insets respected for notched devices
- Touch controls only enabled on touch devices (`isTouchDevice` check)

## Browser Compatibility

- Requires: ES6+ JavaScript, Canvas 2D, WebAudio API, localStorage
- Tested on: Modern Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari, Chrome Android (viewport-fit=cover for notch support)
