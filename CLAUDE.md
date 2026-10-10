# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built in vanilla HTML/JavaScript. It's a single-file game that runs entirely in the browser with procedurally generated audio via WebAudio API (no external assets required).

**Key Features:**
- Three difficulty levels (Easy, Normal, Hard) with different enemy spawn rates, speeds, and boss HP multipliers
- Weapon system: Vulcan (spread shots) and Laser (piercing shots), each with 3 upgrade levels
- Smart bomb mechanic with limited charges
- Boss encounters with distinct attack patterns (6 unique boss types across stages)
- Adaptive performance: automatically reduces visual quality (particle count, shadows) on slower devices
- Responsive design: supports desktop (mouse/keyboard), mobile (touch), and gamepad-style input
- Local storage for best scores and user preferences
- Accessibility: screen reader support, reduced-motion preference detection

## Development & Running

### Running the Game
Simply open `index.html` in a web browser. No build process, server, or dependencies required.

To test locally with live reload during development, use any simple HTTP server:
```bash
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Project Structure
- **Single file**: `index.html` contains all game logic, rendering, and styling
- **No build step**: Vanilla JavaScript with no bundler
- **No external dependencies**: Uses only browser APIs (Canvas, WebAudio, localStorage)

## Code Architecture

### High-Level Game Loop

The game runs on `requestAnimationFrame` with a fixed timestep pattern:

```
loop() → update(dt) → draw() → requestAnimationFrame(loop)
```

**Key flow:**
1. Player input (keyboard, mouse, touch) updates movement vectors
2. `update(dt)` advances game state (positions, collisions, spawning, scoring)
3. `draw()` renders to canvas with layered backgrounds, entities, and particles
4. Performance tracking adjusts rendering quality to maintain 60 FPS

### Core Game Systems

#### 1. **Input Handling**
- **Keyboard**: Arrow keys, WASD for movement; Space/Click to fire; X/Shift/P for bomb/pause
- **Mouse**: Glides player toward cursor while hovering (passive when keyboard/joystick active)
- **Touch**: Left zone for movement joystick, right zone for action buttons (Fire/Bomb)
- **Gamepad-like**: Joystick and button support for mobile

**File location**: Lines 767–835 (input event listeners and joystick tracking)

#### 2. **Player & Weapons**
- Player object tracks: position, health, cooldown, weapon type, upgrade level, invulnerability timer
- **Vulcan**: Spread-shot weapon (1, 2, or 3 spread angles by level)
- **Laser**: Piercing shots (can hit multiple enemies, penetrates by level)
- Weapon cooldown: 0.15s for Vulcan, 0.1s for Laser

**File location**: Lines 871–1146 (player state, weapon firing logic)

#### 3. **Enemy Spawning & Behavior**
- Three enemy types: Grunt (weak), Interceptor (fast/weaving), Tank (tanky)
- Enemies spawn off-screen with random patterns: "spread" (radial fire) or "aimed" (target player)
- Interceptors weave side-to-side using sine wave on frame count
- Enemy spawn rate increases dynamically as the stage progresses (difficulty ramps up)
- After 18 kills in a stage, boss warning triggers and a boss spawns

**File location**: Lines 485–506 (enemy types), 1022–1041 (spawn logic)

#### 4. **Boss System**
- Bosses appear at stage boundaries (stages 1–6 cycle through 6 unique designs)
- Each boss has: custom colors, health scaling, 3-pattern attack sequence
- Patterns include radial bursts, aimed spreads, homing missiles, cross-fire, columns, spirals
- Boss telegraphs before each attack (visual core glow + sfx)
- Defeating a boss advances the stage and resets enemy spawning

**File location**: Lines 508–515 (boss designs), 1372–1409 (attack patterns), 1411–1454 (boss update/kill)

#### 5. **Collision Detection & Scoring**
- Simple AABB (axis-aligned bounding box) overlap check: `rectsOverlap(a, b)` at line 1069
- Scoring: base points per enemy type × combo multiplier (1–10x)
- Combo chain: resets on player hit, builds on successive kills within 2.2 seconds
- Every 10 kills displays an on-screen combo popup

**File location**: Lines 1069, 1087–1101 (collision & scoring logic)

#### 6. **Audio Synthesis**
- **No audio files**: all sounds generated via WebAudio API (tone/noise oscillators)
- Master gain routed through dynamics compressor for polished sound
- Sound effects: sfx object with methods for shoot, laser, explosions (enemy/boss), power-ups, boss warnings, stage clears, game over
- Muting: toggled by button, persisted in localStorage
- Spatial audio: panning left/right based on entity screen position

**File location**: Lines 542–676 (audio system and SFX definitions)

#### 7. **Visual Effects & Rendering**
- **Screen shake**: camera jitter on player hit/bomb detonation (intensity & duration)
- **Particle explosions**: spawned on enemy/boss death, scaled by quality setting
- **Vignette effect**: red pulse when player health is critical (1 life remaining)
- **Flash overlays**: brief white flash on hit, longer on bomb
- **Combo popups**: animated text scaling up then fading on kill milestones
- **Boss bar**: health indicator at top center when boss active

**File location**: Lines 698–730 (visual effect triggers), 1520–1638 (draw function)

#### 8. **Difficulty & Performance**
- **Three difficulty presets** (Easy/Normal/Hard): spawn rate, enemy speed, shoot frequency, HP scaling
- **Adaptive quality**: tracks frame times over 45 frames; reduces particle count, disables shadows on slower devices
- **Reduced motion**: respects `prefers-reduced-motion` OS setting; strips animations/shakes on user toggle

**File location**: Lines 474–483 (performance tracking), 485–489 (difficulty presets)

#### 9. **Game State & Lifecycle**
- **resetGame()**: initializes player, clears bullets/enemies, resets score/stage (called on "Start Game" or "Play Again")
- **endGame()**: runs on death, saves best score to localStorage, shows game-over overlay
- **togglePause()**: pauses on P or Esc key, shows overlay
- **Persistent state**: best score, mute toggle, difficulty, reduced-motion preference saved to localStorage

**File location**: Lines 863–911 (state initialization), 1469–1504 (game end/pause)

### Canvas & Rendering
- **Base resolution**: 480×720 (4:6 aspect ratio)
- **Scaling**: responsive; scales up to fit window while maintaining aspect ratio (max 1.6× on desktop)
- **Layer order**: sky gradient → sun glow → parallax clouds → player/enemies/bullets → UI/boss bar → particles
- **Theme rotation**: 6 sky themes (day, sunset, night, storm, arctic, volcanic) that cycle per stage

**File location**: Lines 753–765 (canvas sizing), 1529–1545 (background/theme rendering)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502): define width, height, HP, speed multiplier, color, score value
2. Update `spawnEnemy()` logic (line 1022) to include new type in random selection
3. Add explosion SFX to `EXPLODE_SFX` (line 678) if a custom sound is desired
4. Test spawning by playing a stage

### Adding a New Boss Attack Pattern
1. Create a new pattern function (see `fireRadial`, `fireAimedSpread`, etc. at lines 1324–1370)
2. Add pattern function to the appropriate boss's pattern array in `BOSS_PATTERNS` (line 1372)
3. Boss telegraph duration (22 frames) allows time for visual/audio cue before pattern fires

### Tuning Difficulty
- Edit `DIFFICULTIES` object (line 485): spawn base, enemy speed, shoot chance, boss HP multiplier
- Increasing frame count → longer wait between spawns = easier
- Decreasing spawn rates or increasing enemy speed makes game harder

### Adding a Sound Effect
1. Create a function in the `sfx` object (line 624) using `beep()` or `noiseBurst()` primitives
2. Call `ensureAudio()` to initialize WebAudio context on first user interaction
3. Use `panFor(x)` to add spatial panning based on screen position

### Testing/Validation
- **No automated tests**: game is validated by play-testing
- Open in browser, test all three difficulty levels, confirm bosses spawn correctly, verify audio plays
- Check console for any errors: responsiveness, input lag, or visual glitches
- Verify localStorage persistence: reload page and confirm best score/mute/difficulty settings persist

## Key Code Patterns

### DeltaTime Updates
Movement and physics use frame-independent deltaTime (`dt` in seconds):
```javascript
function update(dt) {
  player.x += mvx * player.speed * dt;  // position += velocity × deltaTime
}
```

### Quality-Based Rendering
Performance-sensitive code branches on the `quality` variable (0=full, 1=medium, 2=low):
```javascript
const glow = quality === 0;  // Only enable shadows/glow on fast devices
```

### Event Deduplication
Animation trigger functions remove and re-add classes to reset CSS animations:
```javascript
function flashHit() {
  flashEl.classList.remove('hit');
  void flashEl.offsetWidth;  // Force reflow to trigger animation restart
  flashEl.classList.add('hit');
}
```

## Important Notes

- **Single file design**: Keep all code in `index.html`. No external modules or splits.
- **No dependencies**: Avoid adding npm packages; use browser APIs only.
- **Canvas rendering**: All visuals drawn to canvas; no DOM elements except buttons/overlay.
- **LocalStorage keys**: Prefixed with `skyfighter.` to avoid conflicts (STORAGE_BEST, STORAGE_MUTE, etc.)
- **Mobile-first**: Test on phone (touch input, safe area insets for notches)
- **Accessibility**: Maintain screen-reader announcements, keyboard shortcuts, and reduced-motion support
