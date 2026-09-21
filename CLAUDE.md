# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file. The entire game runs client-side in the browser with procedurally generated audio (WebAudio synth) and no external dependencies.

- **Game**: 480×720 canvas-based vertical shmup with 6 bosses, 3 enemy types, 3 difficulty modes, and weapon progression
- **Tech**: Pure vanilla JavaScript (1669 lines) with embedded CSS and no build step
- **Platform**: Browser-based, mobile-friendly with touch controls

## Running the Game

1. **Local Development**: Open `index.html` directly in a browser
   - No build tools required
   - No server needed (works with `file://` protocol)
   - Responsive design: scales to fill window up to 1.6× base size (480×720)

2. **Testing**:
   - Test controls: keyboard (arrows/WASD + Space/X for bomb), mouse (move to steer, click to fire), touch (drag left to move, tap FIRE/BOMB buttons)
   - Test all three difficulty modes: Easy, Normal, Hard (buttons available before game starts)
   - Verify screen shake, flash effects, and audio work (toggle via icons in top-right)
   - Test reduced motion mode (🎆 icon) which disables animations for accessibility

3. **Build/Deploy**: Simply upload `index.html` as-is to a web server

## Architecture

### Core Game Loop
- `loop(now)` (line 1640): RequestAnimationFrame loop that calls `update(dt)` then `draw()`
- `update(dt)`: Physics, collision detection, enemy spawning, boss logic
- `draw()`: Canvas rendering with parallax clouds, entities, particles

### Game State
- **Player**: Position, velocity, weapon type/level, cooldown, invulnerability timer
- **Entities**: `bullets[]`, `enemies[]`, `enemyBullets[]`, `particles[]`, `powerUps[]`, `boss`
- **Progression**: `score`, `lives`, `stage`, `kills`, `stageKills` (18 kills per stage triggers boss)
- **Difficulty**: Stored in localStorage; affects spawn rates, enemy speed, lives, bombs, boss HP

### Key Game Systems

**Player Movement** (line 1152-1172):
- Keyboard/joystick take priority over mouse
- Mouse steering: player glides toward pointer when hovering canvas
- Constrained to canvas bounds

**Weapon System**:
- **Vulcan** (line 1124-1134): Spread shots (level 1: 1 bullet, 2: 2 bullets, 3: 3 bullets)
- **Laser** (line 1135-1145): Fewer shots with pierce ability (level 1: pierce 1, level 3: pierce 2)
- Power-ups cycle: weapon level up first, then switch weapon at level 3

**Combat & Collision** (line 1232-1302):
- Simple AABB overlap detection (line 1069-1070)
- Separate collision loops for: enemies vs player bullets, boss vs player bullets, enemy bullets vs player, boss vs player, power-ups vs player
- Laser bullets can pierce multiple enemies; vulcan bullets one-shot

**Enemy System** (line 502-1041):
- **Types**: Grunt (1 HP), Interceptor (1 HP, fast, weaves), Tank (4 HP)
- **Patterns**: "spread" (shoots 3-way) or "aimed" (targets player)
- Stage-based availability (tanks appear stage 2+)
- Weaving pattern: sine wave based on frame count to avoid monotony

**Boss System** (line 508-1435):
- **6 Boss Designs**: Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
- **Attack Telegraph**: Boss glows white 22 frames before attack (visual/audio cue)
- **6 Pattern Sets**: Each boss has 3 attack patterns (radial burst, aimed spread, teleport + fire, etc.) that cycle
- Boss HP multipliers: 0.8× to 1.7× based on design (Titan is hardest)
- Entering animation: boss slides down from top (line 1413-1416)

**Audio System** (line 542-676):
- WebAudio synth: no external sound files
- Oscillator types: square (shoot), sawtooth (explode), sine (powerup)
- Dynamic lowpass/highpass filtering for variety
- Spatial panning based on entity X position
- Gain envelope: attack-sustain-release for natural decay
- Disable with mute button (stored in localStorage)

**Visual Effects**:
- **Screen Shake** (line 726-729): Random pixel offset, scaled down in reduced-motion mode
- **Flash Effects** (line 698-709): Full-screen white flash on hit or bomb (removed for reduced motion)
- **Particles** (line 1073-1085): Spawn on death/explosion, fade out over time (count scales with performance)
- **Vignette** (CSS): Red pulse when 1 life remaining
- **Parallax Clouds**: Two layers at different speeds, looped (line 1507-1544)

**Difficulty Scaling**:
- Spawn rate decreases over time (line 1201)
- Easy: 4 lives, 3 bombs, weaker bosses
- Normal: 3 lives, 2 bombs, baseline bosses
- Hard: 3 lives, 2 bombs, 1.3× boss HP, tougher enemies
- Enemy aggression increases with difficulty (shoot chance, speed)

**Combo & Multiplier System** (line 1087-1101):
- Chain counter increments on each kill
- Chain timer resets if player doesn't kill for 2.2 seconds
- Multiplier = 1 + floor(chain / 5), caps at 10×
- Breaks if player loses life

### Performance Adaptation (line 473-483)
Tracks frame time and dynamically switches quality:
- Quality 0 (full): Glow effects, many particles, high-res shadows
- Quality 1 (medium): ~60% particles, reduced glow
- Quality 2 (low): ~35% particles, no glow
- Threshold: switches down if avg frame time > 25ms, up if < ~18ms

### Input Handling
- **Keyboard** (line 769-776): Arrows/WASD move, Space fires, X/Shift bombs, P/Esc pause
- **Mouse** (line 838-861): Move to steer, click to fire; activates full-screen crosshair
- **Touch** (line 814-834): Left 55% of screen = movement joystick, right 45% = FIRE/BOMB buttons

### Browser Storage
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio on/off toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility setting (also respects system `prefers-reduced-motion`)

## Code Structure

| Section | Lines | Purpose |
|---------|-------|---------|
| Imports & DOM | 426-461 | Get references to all HTML elements |
| Constants | 463-520 | Storage keys, difficulties, enemy types, bosses, themes |
| Audio Setup | 542-676 | WebAudio context, synth functions, SFX library |
| UI Updates | 680-750 | Render hearts, bombs, mute/motion buttons |
| Canvas Sizing | 752-765 | Responsive scaling based on window size |
| Input Handlers | 767-861 | Keyboard, mouse, touch events |
| Game State Init | 863-911 | `resetGame()` initializes all entities |
| Drawing Functions | 913-1637 | `drawPlane()`, `drawBossShip()`, `draw()` (main render) |
| Update Logic | 1148-1302 | `update(dt)` - physics, spawning, collision |
| Boss Patterns | 1324-1435 | Bullet pattern functions and `updateBoss()` |
| Main Loop | 1640-1665 | `loop(now)` and event listeners for start button |

## Notable Implementation Details

### Sprite Rendering
All entities are drawn procedurally (no sprites):
- **Player**: Triangle-based fighter jet shape with flame when moving forward
- **Enemies**: Same jet shape with different colors; tanks have HP bar
- **Bullets**: Rectangles (vulcan) or rounded (laser/enemy)
- **Boss**: Multi-part design with hull, wings, glowing core; patterns repeat

### Scrolling & Movement
- No level scrolling (player confined to screen)
- Enemies spawn above screen, fall downward at varying speeds
- Clouds loop seamlessly for illusion of scrolling sky

### Canvas 2D Performance Tips
- Use `ctx.save()`/`restore()` to manage transform/alpha/shadow state
- Shadow blur only applied at quality 0 (full) to save fillRect calls
- Particles scaled down based on performance (line 1074)
- Clear rect once per frame (line 1521)

### Balancing Considerations
- Boss HP multiplier system allows tweaking difficulty without changing base HP pool
- Enemy spawn timer ramps difficulty over time (frame counter at line 1201)
- Power-up drop rates tuned to feel rewarding without trivializing (line 1058-1066)
- Chain multiplier cap at 10× prevents runaway high scores
- Invulnerability window (1.5s after hit) prevents instant death chains

## Future Development Notes

- **Themes**: 6 themes defined (Daylight, Sunset, Night, Storm, Arctic, Volcanic) selectable by stage
- **Combo Pop**: Pops up every 10 kills for visual/audio feedback
- **Boss Patterns**: Each boss has 3 patterns in `BOSS_PATTERNS[]`; extend for more variety
- **Mobile Optimization**: Touch controls already adapted (big hit targets), but viewport meta tags are set for mobile
- **Accessibility**: Reduced motion mode disables all animations and screen shake; audio can be muted; screen reader live region for announcements

## Testing Checklist

- [ ] Canvas scales properly on desktop (4:3 and 16:9) and mobile (portrait)
- [ ] All three difficulty modes spawn correctly and feel progressively harder
- [ ] Boss telegraphs render, audio plays, patterns fire at expected times
- [ ] Weapon upgrade cycle works (vulcan 1→3, then switch to laser 1→3)
- [ ] Power-up drop rates and types feel balanced
- [ ] Screen shake and flash toggle with reduced motion setting
- [ ] Touch controls are responsive (20ms+ latency acceptable)
- [ ] Audio context resumes on user interaction (iOS requirement)
- [ ] Score multiplier resets on pause but not on player death alone
- [ ] Best score persists across sessions
