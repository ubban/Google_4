# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas game. Players control a fighter plane, collect weapon power-ups, drop smart bombs, and defeat stage bosses across multiple difficulty levels.

- **Engine**: Pure HTML5 Canvas + WebAudio (no external assets, fully self-contained)
- **Lines of Code**: ~1,670 in `index.html`
- **Architecture**: Single-file (CSS + HTML + JavaScript inline)

## Running & Development

Since this is a single HTML file game, no build step is required:

```bash
# Open the game in a browser
# Option 1: Open directly
open index.html

# Option 2: Serve locally (needed for development/testing)
python3 -m http.server 8000
# Then visit http://localhost:8000

# Option 3: With live reload (using npx http-server or similar)
npx http-server -c-1
```

## Code Architecture

### Single-File Structure
All code is in `index.html` organized in sections:

1. **HTML (lines 362-420)**: Game container, canvas, HUD, buttons, overlay menu
2. **CSS (lines 11-359)**: Complete game styling including animations, responsive layout, UI components
3. **JavaScript (lines 422-1669)**: Game logic in this order:
   - Game state initialization
   - Difficulty definitions
   - Enemy/boss/theme definitions
   - Audio system (WebAudio synthesis)
   - Collision detection utilities
   - Game update logic
   - Rendering functions
   - Input handling (keyboard, mouse, touch)
   - Main game loop

### Key Data Structures

**Game State:**
- `player` - Position (x, y), size (w, h), weapon (vulcan/laser), level, bomb count
- `enemies[]` - Array of active enemy ships with type, HP, movement pattern, AI state
- `boss` - Current boss state, health, attack pattern index, telegraph timing
- `bullets[]`, `enemyBullets[]` - Player and enemy projectiles
- `particles[]` - Explosion/effect particles
- `powerUps[]` - Collectible items (weapon, bomb, life)
- `score`, `multiplier`, `chain` - Scoring and combo state
- `stage`, `lives`, `bombCount` - Game progression

**Configuration Objects:**
- `DIFFICULTIES` - Spawn rates, enemy speed, lives, bombs per difficulty level
- `ENEMY_TYPES` - Grunt, Interceptor, Tank with stats (HP, speed, score value)
- `BOSSES` - 6 boss designs with visual colors and HP multiplier
- `BOSS_PATTERNS` - Attack patterns for each boss (6 patterns per boss)
- `THEMES` - 6 visual themes with sky gradient colors

### Game Loop & Update Cycle

Main loop in `loop()` function (line 1640):
```
requestAnimationFrame → loop(dt)
  ├─ update(dt) - Game state updates
  │  ├─ Player movement/firing
  │  ├─ Bullet movement & collision
  │  ├─ Enemy spawning & AI
  │  ├─ Boss updates
  │  ├─ Power-up collection
  │  └─ Stage progression
  └─ render() - Canvas drawing
     ├─ Background (gradient)
     ├─ Boss, enemies, player
     ├─ Bullets (glow effects at quality 0)
     ├─ HUD updates
     └─ Visual effects (particles, flash, vignette)
```

### Critical Subsystems

**Enemy AI:**
- Spawns from top randomly using `spawnEnemy()` (stage-aware type selection)
- Two patterns: "aimed" (toward player) and "spread" (radial)
- Weaving movement (interceptor type)
- Tank enemies have multi-hit HP with health bar

**Boss System:**
- One boss per stage; uses `BOSSES[(stage-1) % 6]` to cycle through designs
- `updateBoss()` handles movement and attack phases
- Attack patterns use dedicated fire functions:
  - `fireRadial()` - Circular bullet spreads
  - `fireAimedSpread()` - Aimed with spread angle
  - `fireColumns()`, `fireSpiralBurst()`, `fireCross()` - Geometric patterns
  - `fireHomingSeeded()` - Homing bullets
  - `fireDoubleRing()` - Double circular rings
- Telegraph warning (visual + audio) 22 frames before attack

**Collision & Scoring:**
- `rectsOverlap()` - AABB collision for bullets, enemies, power-ups
- Combo system: `chain` counter increments per hit; `multiplier = 1 + floor(chain/5)`; caps at 10x
- Chain breaks if no kills for 2.2 seconds (`chainTimer`)

**Audio (No External Files):**
- `beep(freq, dur, type, vol, opts)` - Oscillator-based sound with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)` - Noise-based sound effects
- Procedural sfx: powerup, fire, boss telegraph, stage clear, damage, etc.
- All routed through master gain with dynamics compression

**Input Handling:**
- **Keyboard**: Arrow keys + WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move to steer (cursor follows mouse), click to fire
- **Touch**: Drag left side to move (visual joystick), tap FIRE/BOMB buttons
- `mouseActive` flag tracks mouse usage

**Performance Tuning:**
- Adaptive quality (0=full, 1=medium, 2=low) based on 45-frame rolling average
- Quality affects particle count, shadow blur, glow effects
- Frame time tracked in `frameTimes[]`

**Difficulty Levels:**
- **Easy**: Spawn base 75, min 40; enemy speed 1.0; shoot chance 0.45; 4 lives
- **Normal**: Spawn base 58, min 26; enemy speed 1.4; shoot chance 0.7; 3 lives
- **Hard**: Spawn base 40, min 18; enemy speed 1.9; shoot chance 0.95; 3 lives

### Rendering & Effects

**Visual Themes:** 6 themes cycle per stage with unique sky gradients and cloud colors
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

**Screen Effects:**
- `screenShake()` - Shakes camera for impacts/boss death
- `vignette` - Red pulsing edge effect when damaged
- `flash` - White/color flash overlay on hits/bombs
- `comboPop` - Animated combo multiplier text
- `stageBanner` - "WARNING — BOSS APPROACHING" on stage start
- All disabled via `reduced-motion` class for accessibility

**Weapon Rendering:**
- Player bullets: Yellow squares (Vulcan) with glow, or cyan rectangles (Laser) with glow
- Enemy bullets: Red circles, no glow at quality 2
- Weapon level increments 1–3 per power-up; after level 3, toggles between Vulcan/Laser

### Persistence & State

Uses `localStorage` with keys:
- `skyfighter.bestScore` - All-time high score
- `skyfighter.muted` - Sound mute state
- `skyfighter.difficulty` - Last selected difficulty (default: normal)
- `skyfighter.reducedMotion` - Accessibility preference override

### Accessibility Features

- `aria-live` announcements for game events
- Reduced motion mode respects `prefers-reduced-motion` media query
- Screen reader text layer (`sr-only` class, `#srLive`)
- Touch controls and icon buttons with titles
- Semantic button labels

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with stats (w, h, hp, speedMul, color, dark, score, minStage)
2. Update `spawnEnemy()` random weight checks if needed
3. Add rendering logic in `render()` if special shape needed (tanks show HP bars)

### Adding a New Boss
1. Add to `BOSSES` array with design (colors, hpMul)
2. Add corresponding attack patterns array to `BOSS_PATTERNS` (same index)
3. Boss health formula: `baseHp = 70 + stage * 35; hp *= difficulty.bossHpMul * design.hpMul`

### Adding a New Boss Attack Pattern
Create a function in the BOSS_PATTERNS section like:
```javascript
b => fireRadial(b, count, speed)
```
Functions receive boss object `b` and push to `enemyBullets[]`.

### Adjusting Difficulty
Edit `DIFFICULTIES` object keys: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

### Adding Visual Effects
- Use `screenShake(magnitude, duration)` for impact
- Use `spawnExplosion(x, y, color, count)` to create particles
- Modify CSS animations (search `@keyframes`) for UI feedback

### Audio SFX
- Call `beep()` or `noiseBurst()` with parameters
- Both return immediately if `muted` or `audioCtx` not ready
- Ensure `ensureAudio()` called first (user must have interacted with page)

## Testing

**Manual Testing Checklist:**
1. Start game, verify HUD shows Score, Best, Weapon, Lives (hearts), Bombs
2. Keyboard controls: Arrow keys move, Space fires, X/Shift drops bomb
3. Mouse: Drag to move, click to fire
4. Touch: Swipe left to move, tap fire/bomb buttons
5. Pause (P or Esc button), resume
6. Difficulty buttons select Easy/Normal/Hard
7. Kill 18 enemies per stage to trigger boss
8. Boss enters, uses attack patterns, takes damage, dies → stage clears
9. Collect power-ups: weapon upgrade, extra life, extra bomb
10. Combo multiplier builds on consecutive kills, breaks after 2.2s no kills
11. Sound on/off mute button; reduced motion toggle
12. Score persists after game over (localStorage)
13. Quality adapts if frame rate drops below 40 FPS

**Performance Profiling:**
- Check frame rate in browser DevTools Performance tab
- Watch for quality downgrade message in console if desired
- Explosive moments (boss death) should trigger 0–2 screen shakes

## Common Tasks

**Adjust weapon balance:**
- Edit Vulcan fire rate in `update()` where `fireTimer` checks
- Edit Laser fire rate or damage (collision detection is same for all bullets)

**Change score values:**
- Enemy kills: `ENEMY_TYPES[type].score`
- Boss kill: `addScore(500)` in `killBoss()`
- Multiplier formula: `multiplier = 1 + Math.min(9, Math.floor(chain / 5))`

**Modify spawn rate:**
- `spawnTimer` logic in `update()` decreases per frame
- Base spawn interval: `DIFFICULTIES[difficulty].spawnBase - stage` (decreases per stage)
- Minimum cap: `DIFFICULTIES[difficulty].spawnMin`

**Change stage progression:**
- Current: 18 kills per stage (`KILLS_PER_STAGE = 18`)
- Boss uses stage to pick design: `BOSSES[(stage - 1) % BOSSES.length]`
- Max displayed stages: cycles after 6 bosses

**Tweak visual themes:**
- Edit `THEMES` array (sky gradients, cloud/sun colors)
- Theme picked per stage: `THEMES[(stage - 1) % THEMES.length]`

## Debugging Tips

- Add `console.log(player.x, player.y, stage, enemies.length)` in update loop for state
- Use `ctx.fillText()` overlay to debug values on canvas
- Check `quality` variable to see if perf degradation kicked in
- Inspect `localStorage` in DevTools to see saved settings
- Listen for audio context state via `audioCtx.state` (suspended, running, closed)
