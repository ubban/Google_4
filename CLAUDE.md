# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features procedural enemy waves, boss encounters, multiple weapon types, smart bombs, difficulty modes, touch/mobile support, and synthesized sound effects (no external audio files).

## Running & Development

### View the Game
Simply open `index.html` in a web browser. The game is fully self-contained with no build process or dependencies.

```bash
# Quick preview (from project root)
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Testing Changes
1. Make edits to the `<script>` block in `index.html`
2. Reload the browser to see changes immediately
3. Open browser DevTools (F12) to debug or check the console for errors

## Architecture Overview

The entire game logic is contained in a single `<script>` block within `index.html`, organized into logical sections:

### Core Systems

**1. Canvas Rendering & Game Loop**
- Canvas-based 2D rendering at 480×720 base resolution (scales responsively)
- Game loop driven by `requestAnimationFrame` with frame-time tracking
- Adaptive quality system that reduces particle effects if FPS drops

**2. Entity Systems**
- **Player**: Single ship at screen bottom, steered via mouse/keyboard/touch, tracks health, bombs, weapon state
- **Enemies**: Three types (grunt, interceptor, tank) spawn in waves based on difficulty, shoot projectiles, drop weapon power-ups
- **Bosses**: 6 boss types (Vanguard, Behemoth, Specter, etc.) with unique visual designs, appear after 18 enemy kills per stage
- **Projectiles**: Player bullets, enemy fire, bomb radius effects
- **Particles**: Explosions, hits, visual feedback (screenshake, vignette, flash)

**3. Game State**
- `running`: Whether game is currently active
- `paused`: Pause state
- `lives`, `bombs`: Consumables that persist across stages
- `score`, `mult`: Score and combo multiplier
- `stage`: Current stage number (increases with each boss defeat)
- `weapon`, `weaponLvl`: Current weapon type and upgrade level

**4. Difficulty System**
Three presets (Easy, Normal, Hard) control:
- Enemy spawn rates and minimum timings
- Enemy movement speed and fire rate
- Player starting lives, bombs, and boss HP multiplier
- Difficulty choice is saved to `localStorage` and applied on next game

**5. Audio System**
All sounds are synthesized via WebAudio API (no external files):
- `beep()`: Tonal sounds with optional frequency sweep, filter, stereo pan
- `noiseBurst()`: Percussive/explosive sounds with noise generation
- Individual SFX functions for each game event (shoot, explode, hit, bomb, boss telegraph, etc.)
- Mute state saved to `localStorage`

**6. Input & Touch Controls**
- **Keyboard**: Arrow keys or WASD to move, Space/click to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to aim/steer, click to fire
- **Touch**: Left 55% of screen is movement zone, right 45% has fire/bomb buttons with virtual joystick
- Touch joystick appears when dragging left side, normalizes to [-1,1] movement vector

**7. Visual Effects & HUD**
- Screen space rendering: health hearts, bombs, score/best, weapon name/level, multiplier
- Boss health bar (appears only during boss encounters)
- Combo popup: Shows damage multiples (large yellow text when combo activates)
- Stage banner: Displays stage number on progression
- Vignette: Red pulse effect when health critical (1 heart left)
- Flash effect: White screen flash on hit or bomb

**8. Accessibility**
- Reduced motion support: Disables animations/shake/flash if preferred
- Screen reader live region: Announces key events (new best, stage clear, etc.)
- Respects system `prefers-reduced-motion` setting by default

## Key Implementation Details

### Game Loop Flow
1. **Input processing**: Read keyboard, mouse, touch states
2. **Update phase**: Move entities, handle collisions, update state
3. **Spawn phase**: Generate new enemies based on difficulty timers
4. **Render phase**: Clear canvas, draw background gradient, entities, HUD, effects

### Enemy Spawning
Controlled by `spawnCounter` that decrements. When it reaches 0:
- Randomly select enemy type weighted by stage/difficulty
- Spawn at random x position at top of screen
- Reset counter to random value between `spawnMin` and `spawnBase`
- Increases difficulty over time as `spawnMin` decreases

### Boss Encounters
After `KILLS_PER_STAGE` (18) enemy kills:
- Stage banner displays, stage clear SFX plays
- Next screen scrolls in with boss health bar
- Boss spawns with HP = `baseHp * bossHpMul[difficulty]`
- Boss firing pattern and movement scripted; destruction gives bonus score

### Weapon System
Three weapons available:
- **Vulcan**: Standard rapid fire (can be in `index.html` around line 1400+)
- **Laser**: Higher damage, slower fire rate
- **Missile**: Guided projectiles with tracking
- Weapon switches when player picks up power-up drops from defeated enemies
- Weapon level increases with same-weapon pickups

### Combo System
- Multiplier starts at 1×, increases by 0.05 for each enemy killed within 1.2 seconds
- Caps at 5×
- Resets if player takes damage
- Applied to all score gains

### Scoring
Base scores vary by enemy type. Each kill multiplied by current combo. Boss defeats grant bonus flat scores.

## Common Changes

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` object (line ~502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add rendering code in the enemy rendering section (look for `ctx.fillStyle = enemy.color`)
3. Optionally add explosion SFX to `EXPLODE_SFX` mapping

### Add a New Boss
1. Add to `BOSSES` array (line ~508) with `name`, `hullTop`, `hullBottom`, `wing`, `core` (colors), `hpMul`
2. Add boss AI logic in update loop where boss movement/firing is handled
3. Boss rendering already handles hull/wing/core shapes generically

### Adjust Difficulty Balance
Edit `DIFFICULTIES` object (line ~485):
- `spawnBase/spawnMin`: Lower = faster enemy spawns
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemy fires each frame
- `lives/bombs`: Starting resources
- `bossHpMul`: Multiplier on boss base HP

### Change Visual Theme
Edit `THEMES` array (line ~493). Each theme has:
- `sky`: Array of 3 gradient colors for background
- `cloud/sun`: RGB values for particle colors
- `name`: Display name for internal reference

A new stage randomly picks one of these themes for background gradient and particle effects.

### Add New Sound Effect
Create a function in the `sfx` object (line ~624) using `beep()` or `noiseBurst()`:
```javascript
newEffect: () => {
  beep(440, 0.1, 'sine', 0.05);  // freq, duration, type, volume
}
```
Call it with `sfx.newEffect()` where needed. Audio context is lazily initialized on first user input.

## Performance Notes

- Particles are culled when off-screen to reduce draw calls
- Enemy count is capped (see `MAX_ENEMIES` constant if present)
- Canvas is redrawn every frame; no dirty-rect optimization
- Quality setting (line ~474) auto-adjusts particle density if frame time exceeds 40ms
- Touch inputs batch-process multiple touches per frame to avoid jank

## Browser Support

Works in all modern browsers with:
- HTML5 Canvas 2D
- WebAudio API (for synth sounds)
- LocalStorage (for high scores, difficulty, mute state)
- Touch Events (for mobile)

Gracefully degrades if audio unavailable (game still playable, just silent).
