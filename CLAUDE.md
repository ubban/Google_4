# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. It features procedurally-generated sound effects via Web Audio API, responsive canvas rendering, touch and keyboard controls, and adaptive performance optimization.

## Quick Start

- **Open the game**: Simply open `index.html` in a web browser
- **Test locally**: Use a simple HTTP server: `python3 -m http.server 8000` then visit `http://localhost:8000`
- **No build step required** — the game runs directly as a static HTML file

## Architecture

The entire game lives in a single `index.html` file (58KB) split into three sections:

### 1. **DOM & Styling (HTML + CSS)**
- Game canvas and HUD elements (score, health, bombs, weapon level, boss bar)
- Overlay system for title screen and game over states
- Touch control zones (joystick, fire button, bomb button)
- Status indicators (hearts for lives, bombs, combo popups, stage banners)
- Accessibility features (screen reader live region, reduced motion preference support)
- Responsive layout using `clamp()` for font scaling across devices

### 2. **Core Game Logic (JavaScript IIFE)**
The script is wrapped in a self-executing function starting at line 422. Key systems:

#### **Game State Management**
- `player`: x, y, width, height, weapon type, health, invulnerability
- `enemies` / `enemyBullets` / `bullets`: Array-based entity management
- `particles`: Visual effects (explosions, dust)
- `powerUps`: Weapon and health pickups
- `boss`: Current boss encounter
- Score, lives, bomb count, stage tracking
- `stage`, `stageKills`, `multiplier`, `chain` (combo system)

#### **Core Gameplay Systems**

**Player Control**
- Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- Mouse: Movement steers player toward cursor, click to fire
- Touch: Left zone for joystick movement, right zone for fire/bomb buttons
- Priority system: Keyboard/joystick input overrides mouse to prevent conflicts

**Difficulty System** (line 485-489)
- Three presets: Easy (75 spawn base, 0.8× boss HP), Normal (58 spawn base, 1.0×), Hard (40 spawn base, 1.3×)
- Each adjusts enemy spawn rate, enemy speed, enemy fire chance, lives, bombs, and boss health
- Persisted to localStorage

**Weapon System**
- Two weapons: Vulcan (rapid-fire, 60ms cooldown), Laser (slower, longer range)
- Weapons have levels (1-6): level determines projectile speed and damage
- Weapon pickups randomly spawn from destroyed enemies
- Displayed in HUD with weapon name and current level

**Enemy Types** (line 502-506)
- **Grunt** (red): 1 HP, basic speed, scores 10 points
- **Interceptor** (orange): 1 HP, 1.7× speed, weaving behavior, scores 15 points
- **Tank** (purple): 4 HP, 0.55× speed, tanky, scores 35 points
- Spawn rates and difficulty increase per stage

**Boss Encounters**
- Triggered after 18 kills per stage (`KILLS_PER_STAGE`)
- Six boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique visual styles and HP multipliers
- Boss HP scales with difficulty and stage progression
- Boss bar shown at top when fighting
- Boss telegraph sounds before attacks

**Score Multiplier System**
- Base: 1× multiplier
- Chain counter: Increases by 1 for each enemy destroyed without taking damage
- Resets to 1 when player takes damage
- Multiplier = min(chain, 8) — caps at 8×
- Displayed in HUD; shows "x" symbol

**Bomb Mechanics**
- Limited quantity (2-3 based on difficulty)
- Triggered by X, Shift, or bomb button
- Clears all enemy bullets and damages nearby enemies
- Uses procedural "bomb" sound effect

#### **Rendering Pipeline**
- Canvas setup: 480×720 base resolution, scaled to fit window (line 753-765)
- RAF loop with delta-time calculation
- Adaptive performance: Tracks frame times (line 476-483), automatically reduces quality if avg > 25ms
- Screen shake applied via canvas translate based on collision force
- All entities (player, enemies, bullets, particles, powerups) drawn each frame

#### **Audio System** (line 542-676)
- Web Audio API with master gain + compressor
- Lazy initialization: Audio context created on first user interaction (touch, click, key)
- Procedural SFX library (`sfx` object): 
  - `shoot`, `laser`, `explodeGrunt`, `explodeInterceptor`, `explodeTank`
  - `hit`, `bomb`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`
- Pan and sweep parameters for positional audio
- No external audio files — all generated via oscillators and noise bursts
- Mutable via icon button; mute state persisted to localStorage

#### **Persistence** (lines 463-466)
- localStorage keys:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.difficulty`: Selected difficulty
  - `skyfighter.reducedMotion`: Reduced motion preference

#### **Visual Effects**
- Vignette flash (red pulse) when health critical
- Screen flash on hit or bomb detonation
- Combo popup animations (yellow text, scale pulse)
- Stage banner entrance animation (1s slide)
- Particle system for bullet impacts and explosions

#### **Accessibility**
- Keyboard-only playable (no mouse required)
- Touch controls fully accessible
- Reduced motion mode respects system preference (`prefers-reduced-motion`)
- Screen reader announcements via `srLive` (live region)
- Semantic HTML: buttons, aria-live region, title attributes

#### **Themes**
Six visual themes (line 493-500) with different sky gradients, cloud colors, and sun colors:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Theme selects automatically based on stage progression (cycles through themes)

## Key Code Sections to Modify

When making changes, understand these regions:

1. **Game Constants** (lines 485-515): Difficulty settings, enemy types, boss data, theme definitions
2. **Game State** (line 864): Central state variables — modify here to add new mechanics
3. **Render Loop** (search for `RAF`): Main `requestAnimationFrame` callback — where all drawing happens
4. **Player Update** (search for `player.x`): Player movement and collision logic
5. **Enemy Spawning** (search for `spawnTimer`): Enemy generation and wave difficulty
6. **Collision Detection** (search for `Math.hypot` or distance checks): Hit detection between bullets, enemies, bombs
7. **Audio Functions** (lines 565-676): Sound effect generation — modify frequencies, durations, sweeps
8. **Input Handlers** (lines 768-861): Keyboard, mouse, touch event listeners

## Common Development Tasks

**Add a new weapon type:**
1. Add to weapon type constants (e.g., define speed, rate of fire)
2. Add rendering logic in the draw section for the new weapon projectiles
3. Add collision/damage logic in enemy hit detection
4. Add SFX for the new weapon

**Add a new enemy type:**
1. Add to `ENEMY_TYPES` constant with visual and behavioral properties
2. Increase `minStage` so it doesn't spawn too early
3. Add spawn logic in the enemy wave generator
4. Add rendering code in the canvas draw loop
5. Add explosion sound to `EXPLODE_SFX` mapping if needed

**Add a new boss:**
1. Add to `BOSSES` array with name, hull colors, core color, and HP multiplier
2. Add movement and attack pattern logic in boss update
3. Add rendering code (boss is drawn as a composite of shapes)
4. Add boss-specific telegraph/death sounds if needed

**Adjust difficulty balance:**
1. Modify `DIFFICULTIES` presets: spawn rates, enemy speed, fire chance, lives, bombs, HP multiplier
2. Adjust `KILLS_PER_STAGE` to change when bosses appear
3. Test on all three difficulties to ensure progressive challenge

**Change game resolution:**
1. Modify `BASE_W` and `BASE_H` constants (line 753)
2. Update any hardcoded coordinate logic in rendering/collision
3. Resize function recalculates scaling automatically

## Testing

The game has no automated test suite. Manual testing checklist:

- [ ] Game starts and reaches title screen
- [ ] All three difficulty levels selectable and persist across sessions
- [ ] Player movement responsive (keyboard, mouse, touch)
- [ ] Weapons fire and cycle correctly
- [ ] Enemy spawning accelerates through stages
- [ ] Collisions detect hits (player ↔ enemy, bullet ↔ enemy)
- [ ] Bombs clear screen and prevent stacks
- [ ] Boss encounters trigger after 18 kills
- [ ] Score and multiplier update visually
- [ ] Audio mute button toggles sound (no errors when clicking)
- [ ] Reduced motion mode disables animations
- [ ] Game over screen shows final score
- [ ] Best score persists between sessions
- [ ] High score updates correctly
- [ ] Responsive on mobile: touch controls visible, canvas scales
- [ ] No console errors

## Performance Considerations

- **Adaptive rendering**: Quality decreases if avg frame time exceeds 25ms (line 476-483)
- **Particle pooling**: Particles array reused rather than recreated
- **Collision broadphase**: Uses distance checks; only pixel-perfect on collision
- **Canvas clearing**: Full canvas redraw every frame (no dirty regions)
- **WebAudio synthesis**: On-demand generation; no audio buffer caching

## Browser Compatibility

- Canvas 2D context required
- Web Audio API (with webkit prefix fallback for older Safari)
- localStorage for persistence
- ES6 syntax: let/const, arrow functions, template literals, for-of loops
- Touch Events API for mobile controls
- CSS: clamp(), backdrop-filter, CSS Grid/Flex, CSS animations, CSS variables via :root

## Future Enhancement Ideas

- Multiple weapons at once (dual-wield)
- Shield mechanic (temporary invulnerability with visual indicator)
- Enemy AI patterns (sweep attacks, formations)
- Stage-specific hazards (moving obstacles, walls)
- Power-up variants (rapid fire boost, health restore)
- Leaderboard system (cloud-based or local IndexedDB)
- Replay system (record and playback input)
- Level editor or procedural level generation
- New boss attack patterns (rotating projectiles, homing fire)
