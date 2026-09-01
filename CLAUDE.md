# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written as a single-file HTML/CSS/JavaScript application. Players control a fighter jet, collect weapon upgrades, deploy smart bombs, and progress through stages with increasingly difficult enemy waves, culminating in boss encounters.

### Core Mechanics
- **Progressive difficulty**: 18 enemy kills per stage advance; stages introduce new enemy types
- **Dual weapons**: Vulcan (spread fire) and Laser (focused beam)—weapon level affects fire rate
- **Smart bombs**: Limited-use stage-wide damage and brief invulnerability
- **Combo multiplier**: Hit streaks (chain variable) increase score multiplier; breaks after ~3 seconds of no hits
- **Boss encounters**: One unique boss per stage; health scaled by difficulty

### Game Settings
- **Difficulty modes**: Easy (4 lives, 3 bombs) / Normal (3 lives, 2 bombs) / Hard (3 lives, 2 bombs)
- **Visual themes**: 6 dynamic sky/weather themes with gradient backgrounds
- **Accessibility**: Reduced motion support (disables animations/screenshake), touch controls for mobile

## Architecture & Key Sections

The entire game is in `index.html` (1669 lines). The structure follows this pattern:

### 1. **HTML + CSS** (lines 1–359)
- Canvas-based game rendering
- HUD displaying score, weapon level, health, bombs, multiplier
- Boss health bar (appears during boss stages)
- Overlay UI (start menu, game-over screen, controls hint)
- Touch control zones (left for movement, right for fire/bomb)
- Visual feedback: flash effect, vignette (low health), combo pop, stage banner

### 2. **Initialization & State** (lines 423–900)
- Local storage for best score, mute state, difficulty preference, reduced motion
- Global game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Difficulty configs (enemy spawn rates, speeds, shoot chance, lives, bomb count, boss HP multiplier)
- 3 enemy types: grunt, interceptor, tank (with distinct colors, speeds, HP, scores)
- 6 boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with visual variants

### 3. **Audio System** (lines 542–676)
- Web Audio API synth (no external audio files)—all sounds procedurally generated
- Beep function: uses oscillators with optional sweep, filter, panning
- Noise burst: filtered white noise for impacts
- SFX effects: shoot, laser, hit, bomb, powerup, boss telegraph, stage clear, game over
- Adaptive to panning (left/right based on source X position)

### 4. **Input Handling** (lines 767–861)
- Keyboard: arrows/WASD move, Space/click fire, X/Shift/click bomb, P/Esc pause
- Mouse: steering by cursor position (priority when moving over canvas)
- Touch: left zone = joystick, right zone = fire/bomb buttons
- Joystick abstraction: `joyVec` tracks normalized input
- Input priority: keyboard/joystick > mouse (mouse steers only if no key input)

### 5. **Game Loop & Rendering** (look for `requestAnimationFrame` and main update function)
- Frame counter drives spawn timers and animation
- Update order: player movement/cooldown, enemy spawn, enemy movement, collision detection, projectile motion, damage/death
- Canvas drawing: background gradient (theme-based), player, enemies, projectiles, particles, UI overlays
- Adaptive performance: tracks frame time, reduces quality if frame rate drops below 40 fps

### 6. **Combat & Collision**
- Player bullets: 2–8 projectiles per fire (weapon-dependent)
- Enemy bullets: simple downward paths from enemy positions
- Hitboxes: rectangles; player invulnerability after taking damage
- Damage: enemies die in 1–4 hits; bosses need cumulative damage over multiple rounds

### 7. **Boss System**
- Boss spawn: after `KILLS_PER_STAGE` kills on current stage
- Boss telegraph: beep + visual warning ~1 sec before first attack
- Boss attack patterns: stage-dependent (not hard-coded per boss name, but behaviors vary)
- Boss defeat: clears enemies, advances stage, triggers stage clear fanfare

### 8. **Progression**
- Each stage ramps enemy spawn frequency, speed, and shoot chance
- New enemy type introduced at specific stages (tank at stage 2+)
- Weapon upgrades: ground pickups increase weapon level (affects fire rate/spread)
- Combo system: hit enemies consecutively to multiply score (caps at ~5x); resets after 3s inactivity

## Common Development Tasks

### **Run the game**
Open `index.html` in a browser. Desktop: use keyboard/mouse. Mobile: drag left zone to steer, tap fire/bomb buttons. Start with Easy mode to verify baseline behavior.

### **Debug a feature**
- Game state lives in module-scoped variables (`player`, `enemies`, `boss`, `score`, `lives`, `chain`, etc.)
- Use browser DevTools console to inspect state mid-game (e.g., `player`, `boss.hp`, `enemies.length`)
- Set breakpoints in the main update loop or specific handlers (e.g., collision checks, spawn logic)
- Log frames with `console.log(frame, state)` to trace state over time

### **Test difficulty balancing**
DIFFICULTIES config (line 485–489) controls spawn rates, enemy speeds, shoot chances, lives, bombs, boss HP multiplier. Tweak these values and reload to test. Use localStorage keys to save player prefs (lines 463–466).

### **Add or modify sounds**
- All audio is in the `sfx` object (lines 624–676)
- `beep(freq, dur, type, vol, opts)` creates a pitched tone with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)` creates filtered noise (impact sounds)
- Combine multiple beeps/bursts with `delay` to build complex sounds
- Test by muting, then calling sfx functions directly in console

### **Adjust visual effects**
- Screen shake: `screenShake(magnitude, duration)` at any collision/impact point
- Flash feedback: `flashHit()` or `flashBomb()` trigger CSS animations
- Combo pop: `showComboPop(text)` displays animated text (e.g., "5x COMBO")
- Reduced motion support: wrap animations in `if (!reducedMotion)` checks or use CSS media query

### **Modify enemy or boss behavior**
- Enemy spawn: check spawn logic (frame-based timers, random type selection by stage)
- Enemy attack: iterate over `enemies`, set bullet spawn positions/velocities
- Boss attacks: stored in `boss` object; update position, shoot timing, visual state within main loop
- Each boss type is just a color/name variant; behavior is generic (no per-boss attack patterns yet)

### **Optimize for performance**
- Quality levels (line 474) disable particle effects on low FPS
- Canvas size: BASE_W=480, BASE_H=720 (intentionally small for fast rendering)
- Particle cap: if too many particles slow the game, reduce max or cull off-screen ones
- Audio: synth generation is fast, but mute sounds during heavy spawns if needed

## Testing Notes

- **Desktop**: Test keyboard (WASD, Space, X) and mouse steering together
- **Mobile**: Verify touch joystick doesn't interfere with fire/bomb buttons
- **Accessibility**: Toggle reduced motion and check animations/screenshake disable correctly
- **Browser audio**: Some browsers block audio until user interaction; start button triggers audio context resumption
- **Performance**: Open DevTools Performance tab, record 30 seconds of gameplay, check frame rate (aim for 50+ fps)

## Storage & Persistence

- Best score, mute state, difficulty, reduced motion are saved to localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- On load, values restore from storage; defaults are applied if keys don't exist

