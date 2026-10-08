# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single HTML file (`index.html`) with embedded CSS styling and vanilla JavaScript game logic. No build step is required — simply open the HTML file in a modern browser to play.

## Running the Game

- **Direct browser**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- **Local server** (recommended for testing): 
  ```bash
  python3 -m http.server 8000
  # Then visit http://localhost:8000/index.html
  ```
- **No dependencies**: Vanilla HTML5, CSS3, Web Audio API, and Canvas 2D — no npm packages or build tools required

## Code Architecture

The entire game logic lives in a single IIFE (Immediately Invoked Function Expression) within the `<script>` tag. Key architectural layers:

### 1. **Game State** (lines 863-869)
- Player object: position, dimensions, speed, weapon type, invulnerability timer, cooldown
- Entity collections: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game mode flags: `running`, `paused`, `gameOver`
- Score tracking: `score`, `chain`, `multiplier`, `kills`, `stage`, `stageKills`, `lives`

### 2. **Input Handling** (lines 767-861)
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire (auto-aims toward cursor)
- **Touch**: Left side for joystick, right side has Fire/Bomb buttons
- Priority: Keyboard/joystick input overrides mouse steering to avoid conflicting inputs

### 3. **Game Loop** (lines 1640-1665)
- `loop(now)` driven by `requestAnimationFrame` at ~60 FPS
- Per-frame: calculate delta time, update game state, draw canvas
- Handles frame skipping if a frame takes >50ms (clamped with `Math.min(dt, 0.05)`)

### 4. **Update/Physics** (lines 1148-1302)
- **Player movement**: Input-driven steering with boundary clamping
- **Bullet lifecycle**: Upward movement (player) and spreading patterns (enemies)
- **Enemy spawning**: Difficulty-scaled timers that accelerate as game progresses
- **Collision detection**: Rectangle overlap for all entity interactions
- **Boss spawning**: Triggered after 18 kills per stage; spawns at stage 1+ and loops through boss designs

### 5. **Boss System** (lines 1372-1439)
- **Boss patterns** (line 1372): 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Attack patterns**: Each boss has 3 unique attack patterns (radial, spread, aimed, spiral, columns, homing, etc.)
- **Telegraph mechanic**: Boss glows white 22 frames before firing to give player reaction time
- **Damage model**: Bullets deal 3 HP, bombs deal 24 HP to boss (separate from enemy damage)

### 6. **Weapon System** (lines 1124-1146)
- **Vulcan** (default): Spray weapon with increased spread at levels 2-3; fires every 0.15s
- **Laser**: Piercing weapon with wider fire lanes at level 2-3; fires every 0.1s
- **Progression**: Levels 1-3 upgrade spread/pierce; level 4 switches weapons
- **Power-ups**: Weapon drops have 34% spawn chance, promote to next tier or swap/reset on cycle

### 7. **Difficulty & Scaling** (lines 485-489, 1200-1202)
- **Difficulties**: Easy (relaxed), Normal (balanced), Hard (intense) — stored in localStorage
- **Dynamic spawning**: Spawn rate accelerates every 5 seconds (`difficultyRamp`)
- **Boss HP**: Scales by difficulty multiplier and boss design multiplier (0.8–1.7×)

### 8. **Audio System** (lines 542-676)
- **Synth engine**: `beep()` generates tones with frequency sweep, filter, and panning
- **Noise generation**: `noiseBurst()` for impact and bass effects
- **No external assets**: All 11 sound effects are procedurally generated (shoot, laser, explosions, bomb, powerup, boss telegraph/defeat, stage clear, UI click, game over)
- **Master compressor**: Prevents clipping on simultaneous sounds
- **Mute toggle**: Persisted in localStorage

### 9. **Visual System** (lines 1507-1638)
- **Canvas drawing**: 480×720 base resolution, scales to fit screen up to 1.6× (line 757)
- **Theme rotation**: 6 themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) cycle per stage
- **Effects**: Screen shake, flash on hit/bomb, vignette pulse when low health, combo text pop
- **Reduced motion**: Respects `prefers-reduced-motion` media query; can be toggled in UI
- **Parallax clouds**: Two cloud layers scroll at different speeds for depth

### 10. **UI & Overlays** (lines 200-358)
- **HUD**: Score, best score, multiplier, weapon level, hearts, bomb count (lines 368-374)
- **Boss bar**: Health bar + name appear when boss spawns (lines 376-379)
- **Overlay card**: Shows at start, pause, and game over with difficulty selector, stats, etc.
- **Accessibility**: Screen-reader live region for announcements (`#srLive`)

### 11. **Persistence** (lines 463-466)
- **localStorage keys**: Best score, mute state, selected difficulty, reduced motion preference
- All user settings persist across sessions

## Key Implementation Details

### Collision & Damage
- `rectsOverlap()` (line 1069): AABB collision for all entity types
- Enemy bullets → player: Loses a life, triggers invulnerability (1.5s), visual flash
- Player bullets → enemies: Deal 1 HP, piercing bullets (laser) can pass through (pierce counter)
- Bomb: Kills all enemies on-screen, deals fixed 24 HP to boss, triggers flash and screen shake

### Chain Multiplier
- Increments on each kill; multiplier = 1 + Math.floor(chain / 5)
- Window: 2.2 seconds between kills to maintain; breaks on taking damage or timeout
- Visual feedback: Counter text pops at 10, 20, 30, etc. kills

### Enemy Spawning
- Pool drawn from types available at current stage: Grunt (stage 1+), Interceptor (stage 1+, 40% rare), Tank (stage 2+, 15% rare)
- Interceptor and Tank have modifiers (1.7×/0.55× speed) and higher HP/score
- Weaving pattern: Interceptors oscillate left-right using sine-wave offset

### Performance Adaptation
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Tracks frame times over 45 frames; adjusts if average >25ms (60 FPS) or <17.8ms (56 FPS)
- Affects particle count on explosions and shadow/glow rendering

## File Structure

```
Google_4/
├── index.html          # Single-file game (entire codebase)
├── README.md           # Minimal project info
└── CLAUDE.md          # This file
```

## Testing & Debugging

- **No automated tests**: Game logic is straightforward; manual testing in browser is primary method
- **Browser DevTools**: Check Console for any errors; Network tab shows no external requests (zero dependencies)
- **Performance profiling**: Open DevTools > Performance tab, record a gameplay session
- **Touch debugging**: Emulate mobile in DevTools (F12 > toggle device toolbar) to test touch controls

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update the spawn logic in `spawnEnemy()` to weight the new type
3. Add explosion SFX mapping in `EXPLODE_SFX` if needed

### Adding a New Boss Pattern
1. Define the attack function in `BOSS_PATTERNS`, e.g., `b => fireRadial(b, 12, 160)`
2. Assign to the boss design's pattern array (index in `BOSS_PATTERNS` = boss index in `BOSSES`)
3. Test telegraph timing (22 frames) and bullet speed/density for balance

### Balancing Difficulty
- Adjust `DIFFICULTIES` object: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Tweak bullet speeds in attack patterns (e.g., 150 → 180 in `fireRadial`)
- Modify stage progression: Change `KILLS_PER_STAGE` to alter pacing

### Visual Polish
- Modify theme colors in `THEMES` array (RGB triplets for sky gradient, cloud, sun)
- Adjust particle count and lifetime in `spawnExplosion()` multiplier
- Tweak animation durations in CSS keyframes (e.g., `@keyframes heartBeat`)

### Audio Tuning
- Edit frequencies and durations in `sfx` object (e.g., `beep(760, 0.07, ...)`)
- Adjust master gain (line 556) or individual volumes in beep/noiseBurst calls

## Browser Compatibility

- **Required**: ES6 (template literals, arrow functions, let/const)
- **Requires APIs**: Canvas 2D, Web Audio API, localStorage, requestAnimationFrame
- **Tested**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile**: iOS Safari, Chrome on Android (touch controls enabled on detection)

## Notes for Future Maintainers

- The codebase is self-contained with no external dependencies — keep it that way for portability
- Frame-rate logic is locked to 60 FPS logic; avoid tying timers to `frame` counter alone (use `dt` for physics)
- Boss telegraph visual (white glow) and audio cue are paired — don't change one without the other
- Reduced-motion preference is critical for accessibility; test that animations respect it before shipping visual changes
- localStorage keys are prefixed with `skyfighter.` to avoid collisions if hosted alongside other games
