# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game logic, rendering, audio synthesis, and UI are contained in `index.html` with no external dependencies—only browser APIs.

## Architecture

### Single-File Structure
The game is self-contained in `index.html`:
- **HTML (lines 1-420)**: DOM structure with canvas and UI overlays for score, HUD, boss health bar, pause menu, and touch controls
- **CSS (lines 10-358)**: Responsive design using `clamp()` for fluid scaling, dark theme, animations for UI feedback (card animations, combo pop, stage banners)
- **JavaScript (lines 422-1666)**: Complete game loop with input handling, state management, physics, rendering, and audio

### Key Game Systems

**Game Loop & Timing** (lines 1640-1653):
- `requestAnimationFrame` drives the loop; delta time is capped at 50ms to prevent large jumps
- Input polling happens on every frame via the `keys` object
- `update(dt)` handles all state changes; `draw()` renders the frame

**Input Handling** (lines 767-861):
- Keyboard input: arrows/WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- Mouse: steering follows pointer, click to fire; takes priority over keyboard when active
- Touch: joystick-style movement on left half, fire/bomb buttons on right half; `moveZone` and `actionZone` define touch areas

**State Management** (lines 863-911):
- Game state: `running`, `paused`, `gameOver`, `stage`, `score`, `lives`, `chain` (combo counter)
- Player object stores position, weapon type, cooldown timers, invulnerability frames
- Enemies and bullets are pooled in arrays and filtered each frame

**Rendering** (lines 1520-1637):
- Canvas is `BASE_W=480px` by `BASE_H=720px` (portrait) but rendered at a scale determined by viewport (max 1.6x)
- Sky gradient, clouds, planes, bullets, particles, HUD are all drawn via Canvas 2D API
- Adaptive glow effects and shadow blur are disabled at lower quality settings to improve performance

**Performance Optimization** (lines 473-483):
- Adaptive quality scaling: tracks frame times over 45 frames; reduces quality (fewer particles, less glow) if avg frame time exceeds 1/40s
- Enemy spawning difficulty increases gradually with frame count
- Particle count is reduced at lower quality tiers

**Audio** (lines 542-676):
- Web Audio API synthesizer: `beep()` creates tonal sounds via oscillators; `noiseBurst()` creates percussive effects
- `masterGain` and `compressor` manage volume and dynamics
- All SFX (shoot, explode, hit, bomb, boss telegraph, etc.) are procedurally generated
- Muted state and audio context suspension handling for user interactions

### Gameplay Mechanics

**Weapons** (lines 1124-1146):
- Vulcan (default): straight shots, spread increases with level (level 1 = 1 shot, level 2 = 2, level 3 = 3)
- Laser: piercing shots that can hit multiple enemies; lanes increase with level
- Switching happens via power-ups; weapon level resets to 1 when swapping

**Difficulty** (lines 485-489):
- Three modes: Easy, Normal, Hard
- Affects enemy spawn rate, speed, accuracy, lives, bombs, boss HP scaling
- Persisted in localStorage

**Chain/Multiplier System** (lines 1087-1101):
- Each kill extends the chain timer (2.2s)
- Multiplier increases: 1x base, then +1x per 5 kills (max 10x)
- Breaking the chain resets to 1x
- Chain timeout is tracked separately; timer resets on each kill

**Bosses** (lines 508-515):
- Six boss designs with unique color schemes and HP multipliers
- Boss selection cycles through `BOSSES` array by stage modulo
- Each has 3 attack patterns (radial bursts, aimed spreads, spirals, etc.) defined in `BOSS_PATTERNS` (lines 1372-1409)
- Boss telegraphs its attack with audio and visual cue (22-frame wind-up)

**Themes** (lines 493-500):
- Six visual themes (sky gradients, cloud colors, sun glow colors) that rotate per stage
- Dynamically generate gradient and color values based on theme

### UI & Overlay System

**HUD** (fixed at top):
- Score, best score, multiplier, weapon name/level, hearts (lives), bombs
- Responsive text sizing with `clamp()`

**Boss Health Bar** (appears when boss spawns):
- Positioned above HUD
- Fades in/out; updates width as boss takes damage

**Overlays** (fullscreen):
- Title screen with difficulty selection, start button, controls hint
- Game Over screen showing score, best score, stage reached
- Pause menu (triggered by P or Esc)

**Touch Controls** (mobile only):
- `moveZone` (left 55% of screen) for movement; draws joystick visuals
- `actionZone` (right 45% of screen) with fire and bomb buttons
- Enabled automatically if device supports touch

## Development Notes

### Modifying Game Balance
- **Enemy difficulty**: Adjust `DIFFICULTIES` object (spawn rate, speed, shoot chance, lives, bombs)
- **Boss difficulty**: Tweak `bossHpMul` in difficulties or `hpMul` in `BOSSES` array
- **Player speed**: Increase `player.speed` (default 260 pixels/sec)
- **Weapon balance**: Adjust cooldown in `fireDelay` or bullet speeds in `fireWeapon()`
- **Chain multiplier**: Edit the multiplier calculation in `addScore()` (currently `1 + Math.min(9, Math.floor(chain / 5))`)

### Adding New Features
- **New weapon type**: Add entry to weapon switch logic in `applyPowerUp()` and firing code in `fireWeapon()`
- **New enemy type**: Add to `ENEMY_TYPES` object; set `minStage` to control when it appears
- **New boss pattern**: Add function to `BOSS_PATTERNS` following the signature `(boss) => { enemyBullets.push(...) }`
- **New theme**: Push to `THEMES` array with sky gradient colors and cloud/sun RGB values

### Testing & Iteration
- Open `index.html` in a browser to play
- Use browser DevTools to pause game and inspect state (canvas element, game variables)
- Adjust `localStorage` keys (`skyfighter.bestScore`, `skyfighter.difficulty`, etc.) to test different conditions
- Toggle reduced motion in-game to see full particle/shake effects

### Persistence
- Best score, muted state, difficulty, and reduced motion setting are stored in `localStorage`
- Keys are `skyfighter.*` prefixed
- Clear with `localStorage.clear()` in DevTools to reset to defaults

## Performance Considerations

- The adaptive quality system is essential; it prevents FPS drops on low-end devices by reducing particles and glow
- Enemy and projectile arrays are actively filtered to remove off-screen objects
- Canvas context transformations are saved/restored to avoid state leaks
- Screen shake only applies if reduced motion is disabled
- Shadow blur is only applied at full quality (`quality === 0`)
