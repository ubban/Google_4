# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built in vanilla HTML5 Canvas. It's a single-file application (`index.html`) featuring player-controlled plane combat, difficulty selection, procedural boss battles, and Web Audio synth-based sound effects.

## Architecture

The entire game lives in `index.html` as one self-contained file (~1,670 lines). Structure:

### HTML (lines 1-420)
- Game canvas container and UI overlays
- HUD elements: score, lives, bombs, weapon status, boss health bar
- Touch controls for mobile (joystick + fire/bomb buttons)
- Overlay for menu, pause, and game over screens

### CSS (lines 10-358)
- Responsive design (scales canvas 1x to 1.6x based on viewport)
- Glassmorphism UI with backdrop blur
- Animation keyframes: heart beat, combo pop, stage banner, screen flash, vignette pulse
- Reduced-motion media query support for accessibility
- Dark theme with color-coded UI elements (health=red, bombs=yellow, weapon=lime, multiplier=orange)

### JavaScript (lines 422-1666)

**Core systems:**

1. **Game loop** (line 1640)
   - `requestAnimationFrame(loop)` runs update → draw cycle
   - `dt` capped at 50ms to prevent lag skips
   - Performance tracking adapts quality (full → medium → low)

2. **Input handling** (lines 768-861)
   - Keyboard: arrows/WASD move, space/mouse fire, X/Shift bomb, P/Esc pause
   - Mouse: ship glides toward pointer (overridden by keyboard/joystick)
   - Touch: left zone = joystick move, right zone = fire/bomb buttons
   - Adaptive priority: keyboard > joystick > mouse (so inputs never fight)

3. **Game state** (lines 863-911)
   - `player`: position, weapon, cooldown, invuln, level
   - `enemies`, `enemyBullets`, `bullets`, `powerUps`, `particles`
   - `score`, `lives`, `stage`, `chain`, `multiplier`, `boss`
   - Persisted: best score, mute, difficulty, reduced-motion preference

4. **Enemy spawning & AI** (lines 1022-1041, 1210-1227)
   - 3 types: grunt (weak), interceptor (fast/weaves), tank (armored)
   - Per-difficulty spawn rate ramps as frame count increases
   - Two shoot patterns: spread or aimed at player
   - Weaving enemies use sin wave offset from base spawn X

5. **Boss system** (lines 1043-1434, 1372-1409)
   - 6 boss designs, each with unique attack pattern sets
   - Boss HP scales by difficulty and stage progression
   - Telegraph animation (core glows) before firing
   - Pattern types: radial bursts, aimed spreads, columns, double rings, homing missiles, teleport barrages, cross-fire

6. **Collision detection** (line 1069, used throughout)
   - Simple rect overlap: `Math.abs(a.x - b.x) < (a.w + b.w) / 2`
   - Checked: player↔enemies, player↔enemy bullets, bullets↔enemies, bullets↔boss

7. **Player weapons** (lines 1124-1145)
   - **Vulcan**: fast spread shots, 1–3 bullets based on level
   - **Laser**: piercing shots with damage falloff, up to 4 bullets at level 3
   - Power-ups toggle weapon or increase level; at level 3, next pickup switches weapon

8. **Score & chain multiplier** (lines 1087-1101)
   - Kill resets 2.2s timer; chain breaks if timer expires
   - Multiplier = 1 + floor(chain / 5), capped at 10x
   - Boss kill worth 500 base points

9. **Audio** (lines 542-676)
   - Web Audio API synth: oscillators + filters + panning
   - SFX map: shoot, laser, explosions (per enemy type), powerup, boss telegraph, stage clear, game over
   - Dynamic panning based on on-screen X position
   - Master gain + dynamics compressor for polished output
   - Triggered on user input (ensures audio context resumed on iOS)

10. **Visual systems**
    - **Rendering** (lines 1507-1637): sky gradient, clouds, planes, bullets, particles
    - **Particles** (line 1073-1084): burst on kill/hit, fade out over 0.5s, quality-scaled count
    - **Screen shake** (line 726-729): scaled down 85% in reduced-motion mode
    - **Flash & vignette** (lines 698-709): hit/bomb animations, critical health pulse

11. **Themes** (lines 493-500)
    - 6 color schemes cycle per stage: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
    - Each has sky gradient, cloud color, sun glow

12. **Difficulty presets** (lines 485-489)
    - Easy: slower spawns, weaker enemies, 4 lives, 3 bombs
    - Normal: balanced (default)
    - Hard: fast spawns, aggressive enemies, 1.3x boss HP, 2 bombs

**Stage progression:**
- 18 kills per stage triggers boss spawn
- Boss defeat advances to next stage, resets enemy spawn timer
- Boss type cycles through 6 designs, repeating

## How to Run

Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge). No build step, no server needed.

**Touch/mobile:** Fully playable on phone/tablet with on-screen joystick and buttons.

## Development

**To modify the game:**

1. **Balance:** Adjust `DIFFICULTIES`, `KILLS_PER_STAGE`, `ENEMY_TYPES` HP/speed/score
2. **Spawn rate:** Tweak `spawnBase` / `spawnMin` or the difficulty ramp formula (line 1201)
3. **Boss patterns:** Edit functions in `BOSS_PATTERNS` array (lines 1372–1409)
4. **Player/enemy visuals:** Edit `drawPlane()` (line 913) and `drawBossShip()` (line 970) canvas path commands
5. **Weapon balance:** Change `fireWeapon()` spread/pierce, cooldown, or damage (bullet pierce vs enemy HP)
6. **Themes:** Add new object to `THEMES` array with sky gradient stops and cloud/sun RGB
7. **SFX:** Call new `beep()` or `noiseBurst()` functions in `sfx` object
8. **UI:** Modify HTML overlay structure or CSS keyframes for menus/animations

**Common tasks:**

- **Test a specific difficulty:** Click difficulty button on menu before Start
- **Adjust performance:** Modify `trackPerf()` thresholds (lines 480-482) or quality-dependent drawing (e.g., shadow blur at line 1002)
- **Debug state:** Open DevTools Console, pause game, inspect `player`, `enemies`, `boss`, `score`, `stage`
- **Disable animations:** Set `reducedMotion = true` (line 521) or click the motion button in-game
- **Change stage duration:** Edit `KILLS_PER_STAGE` (line 491)

## Key Constants & Tuning

| Constant | Value | Notes |
|---|---|---|
| `BASE_W`, `BASE_H` | 480, 720 | Canvas resolution (scaled to fit viewport) |
| `KILLS_PER_STAGE` | 18 | Enemies killed before boss |
| `maxDist` (joystick) | 50 | Joystick stick max offset in pixels |
| `chainTimer` | 2.2 | Seconds until chain breaks |
| `player.speed` | 260 | Units/sec movement |
| Vulcan cooldown | 0.15s | Time between shots |
| Laser cooldown | 0.1s | Time between shots (faster) |

## Code Style

- Single global IIFE wrapper for encapsulation
- Variables and functions camelCase
- Canvas context methods chained and restored with `save()`/`restore()`
- No external dependencies; vanilla Canvas 2D + Web Audio
- Particle/entity pools stored in arrays (no garbage collection overhead)

## Mobile Considerations

- Touch controls hidden on desktop, enabled on touch devices (line 779)
- Safe area insets respected for notched devices
- Reduced motion preference honored via media query (line 520)
- Canvas scales responsively; font sizes use `clamp()` for fluid text
- Audio context initialized on first user interaction (line 774)
