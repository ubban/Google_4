# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML/CSS/JavaScript with no external dependencies. The entire game is contained in a single `index.html` file (~1700 lines) combining markup, styles, and game logic.

## Running the Game

The game runs directly in any modern browser—no build tools, dev server, or dependencies required:

```bash
# Option 1: Open the file directly in a browser
open index.html

# Option 2: Use a local HTTP server (for any browser compatibility edge cases)
python3 -m http.server 8000
# then visit http://localhost:8000
```

Since this is a pure client-side game with no backend, development is entirely local. Changes are immediate upon refresh.

## Code Architecture

The `index.html` file is organized in three layers:

### 1. HTML & DOM Elements (Lines 361–420)
- Canvas element (`#gameCanvas`) for rendering
- HUD elements: score, multiplier, weapon info, hearts/bombs
- Boss health bar (`#bossBarWrap`)
- UI overlays: title screen, pause menu, game-over screen
- Touch controls (mobile joystick + fire/bomb buttons)
- Screen effects: vignette, flash, combo popups, stage banners

### 2. CSS Styles (Lines 10–359)
- Dark theme (`color-scheme: dark`)
- Responsive canvas scaling via `clamp()` (works on mobile and desktop)
- HUD styling: glassmorphic stat pills, glow effects
- Animations: heartbeat, combo pulse, stage banner, card entrance
- Screen-shake and flash effects
- Reduced-motion support via `prefers-reduced-motion` media query
- Mobile touch controls layout (left side = movement, right side = actions)

### 3. JavaScript Game Engine (Lines 422–1667)

#### State & Initialization (Lines 863–911)
- Global game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Counters: `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`
- Settings persisted to localStorage: best score, mute state, difficulty, reduced motion preference
- `resetGame()` initializes a fresh game with difficulty-dependent settings

#### Input Handling (Lines 768–861)
- **Keyboard**: Arrow keys/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move cursor to steer (smooth gliding), click to fire; keyboard input takes priority
- **Touch**: 55% left screen = movement joystick, 45% right screen = fire/bomb buttons
- Audio context initialized on first user interaction (`ensureAudio()`)

#### Game Loop & Rendering (Lines 1640–1665, 1520–1637)
- `requestAnimationFrame` loop at 60 FPS
- `update(dt)` handles physics, collision, and state changes
- `draw()` renders canvas
- **Adaptive performance**: Monitors frame times and reduces quality (glow effects, particle count) on slower devices
- Screen shake applied to canvas translate when damage taken or bomb detonated

#### Core Game Systems

**Weapon System (Lines 1124–1146, 871–873)**
- Two weapons: VULCAN (spreads, faster fire) and LASER (piercing, glowing, slower fire)
- Three upgrade levels per weapon
- Power-up at level 3 switches weapon and resets to level 1
- Fire rate: VULCAN 0.15s, LASER 0.1s

**Difficulty & Progression (Lines 485–491)**
- Three presets: Easy, Normal, Hard
- Difficulty affects: enemy spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- Dynamic spawning: base rate improves with frame count (gets harder over time)
- Stages triggered at 18 kills; boss spawns after warning delay

**Boss System (Lines 1043–1056, 1411–1439)**
- 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycling by stage
- Each boss has unique attack patterns (radial bursts, aimed spreads, homing fire, 8-way crosses, etc.)
- Boss enters from top, bounces horizontally, attacks with telegraph phase
- Bombs deal 24 HP; bullets deal 3 HP to bosses, 1 HP to regular enemies

**Scoring & Combos (Lines 1087–1101)**
- Base score: grunts 10 pts, interceptors 15 pts, tanks 35 pts, bosses 500 pts
- Multiplier scales: 1–10x based on kill chain (5 kills per multiplier level)
- Chain resets if player misses for 2.2 seconds
- Every 10 kills shows combo popup

**Visual Themes (Lines 493–500)**
- 6 rotating sky themes (Daylight Front, Sunset, Night, Storm, Arctic, Volcanic)
- Each theme has gradient sky, cloud color, and sun glow
- Clouds scroll at two parallax speeds

#### Audio (Lines 542–676)
- WebAudio API synthesis (no pre-recorded files)
- Sound effects for: shoot, laser, explosions (3 types), hit, bomb, powerup, boss telegraph, boss death, stage clear, UI click, game over
- Dynamic panning based on event X position
- Mutable via button; preference persisted to localStorage
- All audio respects `muted` flag—safe to call sfx functions regardless

#### Collision & Physics (Lines 1069–1294)
- Axis-aligned bounding box collisions (`rectsOverlap()`)
- Player: bounds-checked to screen edges, speed 260 units/sec
- Enemies: spawn at top, fall downward, weaving interceptors follow sine curve
- Bullets: upward (player), downward (enemies), pierce mechanic
- Enemy bullets: radial or aimed patterns
- Power-ups: fall downward at 90 units/sec

#### Enemy Types (Lines 502–506)
- **Grunt**: 34×34, 1 HP, red, basic fire, score 10
- **Interceptor**: 26×26, 1 HP, orange, fast, weaving, score 15 (spawns stage 1+)
- **Tank**: 46×46, 4 HP, purple, slow, tank-like, score 35 (spawns stage 2+, has visible health bar)

#### Boss Patterns (Lines 1372–1409)
Each boss has 3 unique attack patterns that cycle. Examples:
- VANGUARD: radial 14-bullet burst, aimed 5-bullet fan, wide V-fan spreads
- BEHEMOTH: heavy columns + slow aimed spread + dense radial wall
- TITAN: columns + full ring + relentless homing volleys

Pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

#### UI & Persistence (Lines 517–751, 923–1090)
- Pause overlay with resume button
- Game-over screen showing final score, stage reached, best score, and "NEW BEST" badge
- Difficulty selection persisted per session
- Reduced-motion mode: disables screen shake, flash, and animations
- Hearts display (❤️) with critical heartbeat at 1 life
- Bomb counter (💣) shows available smart bombs
- Screen-reader announcements for accessibility via `announce()`

## Extending the Game

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` (line 502) with width, height, HP, speed multiplier, color, and score
2. Update `spawnEnemy()` (line 1026) to include the new type in rotation logic based on stage
3. Add an explosion sound effect to `EXPLODE_SFX` mapping if needed (line 678)
4. Add a render case in the `enemies.forEach()` draw loop if custom visuals needed (line 1603)

### Adding a New Boss
1. Add to `BOSSES` array (line 508) with name and color scheme
2. Create 3 attack pattern functions (or reuse existing ones) and add an array to `BOSS_PATTERNS` (line 1372)
3. Patterns are indexed by BOSSES array position—ensure counts match

### Modifying Difficulty
1. Adjust `DIFFICULTIES` object (line 485): spawn rates, speeds, shoot chances, HP multipliers
2. Dynamic difficulty ramps based on frame count in `spawnEnemy()` (line 1201)

### Adding Visual Polish
- Increase particle count or lifetime in `spawnExplosion()` for more dramatic effects
- Adjust glow settings based on `quality` variable (adaptive performance system)
- Add new animations via keyframes in the `<style>` section

### Performance Tuning
The game includes **adaptive performance** (lines 474–482):
- Tracks last 45 frame times
- If average > 1/40 FPS (>25ms) and not already reduced, degrade quality
- If average < 1/56 FPS (<17.8ms) and not at max detail, upgrade quality
- Quality levels: 0 = full (glow, full particles), 1 = medium (reduced particle count), 2 = low (minimal effects)

Adjust thresholds or quality logic in `trackPerf()` if targeting specific hardware.

## Key Game Constants

| Constant | Value | Purpose |
|----------|-------|---------|
| `BASE_W`, `BASE_H` | 480, 720 | Canvas resolution (scales responsively) |
| `KILLS_PER_STAGE` | 18 | Enemies to defeat before boss spawns |
| `chainTimer` | 2.2 | Seconds to maintain kill chain |
| Player speed | 260 | Units/second movement |
| VULCAN fire rate | 0.15 | Seconds between shots |
| LASER fire rate | 0.1 | Seconds between shots |
| Boss base HP | 70 + stage × 35 | Multiplied by difficulty and design factors |
| Bomb damage | 24 | HP dealt to boss; kills all regular enemies |
| Enemy bullet speed | 130–260 | Varies by pattern and difficulty |

## Testing & Debugging

- Open browser DevTools Console (`F12`)
- Check for any JS errors during gameplay
- Test on multiple device sizes and orientations (canvas scales responsively)
- Verify touch controls on mobile (joystick base appears on first touch)
- Test audio on different browsers (some may require user interaction first)
- Reduced motion mode should disable all animations when `prefers-reduced-motion` is set in OS accessibility settings

## Browser Compatibility

Works on all modern browsers with:
- Canvas 2D context
- AudioContext / webkitAudioContext
- localStorage
- CSS Grid & Flexbox
- CSS `clamp()` and safe-area-inset (mobile notch support)
- Touch Events API (optional, falls back to keyboard)

No transpilation, polyfills, or bundling needed.

## File Size & Performance

- Single file: ~58 KB (HTML + CSS + JS)
- No external assets or dependencies
- Gzipped: ~12–14 KB
- Typical load time on 3G: <1 second
- Adaptive quality system keeps 60 FPS on most devices

## Common Tasks

**Change title or description**: Edit the `<title>` tag and meta description (lines 6–8).

**Adjust game difficulty curve**: Modify `spawnBase` / `spawnMin` in `DIFFICULTIES` or the ramp formula in `spawnEnemy()`.

**Add a new sound effect**: Create a function in the `sfx` object (line 624), using `beep()` or `noiseBurst()` helpers with WebAudio parameters.

**Extend the game length**: Increase `KILLS_PER_STAGE` or add more boss designs / patterns.

**Customize colors**: Use find-replace on color hex codes, or adjust `THEMES` array for different color schemes.
