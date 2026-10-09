# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-em-up game built with HTML5 Canvas and vanilla JavaScript. The entire game logic, rendering, and audio synthesis runs in a single `index.html` file (~1670 lines) with no external dependencies.

## Development Setup

Since this is a single-file HTML game:

- **Opening the game**: Open `index.html` in a browser (no build step required)
- **No dependencies**: Everything is vanilla HTML, CSS, and JavaScript
- **No build tools**: Edit `index.html` directly and reload the browser

## Architecture & Code Organization

The game is organized in a single IIFE with the following major sections:

### 1. **Game State & Constants** (lines 423-525)
- `DIFFICULTIES`: Configuration for Easy/Normal/Hard modes (spawn rates, enemy speed, lives)
- `THEMES`: Six visual themes with sky gradients and cloud colors, cycling by stage
- `ENEMY_TYPES`: Grunt, Interceptor, Tank with stats (HP, speed, score, visual properties)
- `BOSSES`: Six boss designs with unique names and color schemes
- `BOSS_PATTERNS`: Attack pattern arrays for each boss
- localStorage keys and initial value loading

### 2. **Audio System** (lines 542-676)
- **WebAudio Synthesis**: Uses oscillators, noise generators, and filters to create all sounds procedurally
- `beep()`: Creates tonal sounds with optional sweeps and filters
- `noiseBurst()`: Creates noise-based explosion sounds
- `sfx` object: Named sound effects (shoot, laser, explode variants, bomb, boss telegraph, etc.)
- **Key insight**: All audio is synthesized, no external audio files

### 3. **Input Handling** (lines 767-861)
- **Keyboard**: Arrow keys + WASD for movement, Space/Click to fire, X/Shift/P/Esc for special actions
- **Mouse**: Move pointer to steer, click to fire (plane glides toward cursor)
- **Touch**: Left side is movement joystick, right side has FIRE and BOMB buttons
- `joyVec` tracks joystick input; keyboard/joystick input takes priority over mouse

### 4. **Game Loop & Rendering** (lines 1148-1666)
- `update(dt)`: Updates game state (player movement, bullets, enemies, collision detection)
- `draw()`: Renders scene using canvas (theme-based sky, clouds, ships, bullets, effects)
- `loop()`: requestAnimationFrame-based main loop with delta-time
- **Performance adaptation**: `quality` variable scales down particle effects if FPS drops below 40

### 5. **Game Objects & Mechanics**
- **Player** (line 877): Position, speed, weapon system (vulcan/laser), invulnerability timer, level
- **Enemies** (line 879): Position, movement pattern (weaving), shooting behavior, type-based properties
- **Bullets** (line 878): Player bullets with pierce for laser, enemy bullets (both straight and radial)
- **Bosses** (line 883): HP tracking, telegraph/attack pattern system, movement
- **Power-ups**: Life, Bomb, Weapon (increments level or switches weapon type)
- **Particles**: Explosion particles with velocity fade

### 6. **Stage & Boss Progression**
- **Stages**: Auto-progress after defeating KILLS_PER_STAGE (18 kills) enemies
- **Boss Battle**: Warning banner → boss enters with health bar → defeat triggers next stage
- **Difficulty**: Affects spawn rate, enemy speed, lives, bomb count, boss HP multiplier

## Key Gameplay Systems

### Scoring & Multiplier Chain
- `chain`: Increments for each kill; breaks if no kills within 2.2 seconds
- `multiplier`: Calculated as `1 + Math.min(9, Math.floor(chain / 5))`
- Max 10x multiplier at 50+ consecutive kills
- Bomb kills also extend chain

### Weapon System
- **Vulcan**: Spread shot that increases shot count at level 2/3
- **Laser**: Piercing shots with higher damage; switch to laser after level 3
- Power-ups increment level up to 3, then toggle weapon type

### Boss Battle Mechanics
- **Telegraph Phase** (22 frames): Boss core flashes white, warning sound plays
- **Attack Phase**: Boss executes pattern from its pattern array
- **Patterns vary by boss** (6 types × 3 patterns each):
  - VANGUARD: Radial burst, aimed fan, V-spread
  - BEHEMOTH: Heavy columns, homing spread, dense radial
  - SPECTER: Teleport + dash, tight spiral, double aimed
  - DREADNOUGHT: 8-way cross, homing missiles, radial wall
  - WRAITH: Teleport barrage, expanding rings, aimed cross-spread
  - TITAN: Heavy columns, full ring, relentless homing

### Visual & Audio Feedback
- **Screen shake**: Magnitude and duration based on event (hit: 10, bomb: 14, boss death: 18)
- **Flash overlay**: 0.25s white flash on hit, 0.5s on bomb
- **Vignette pulse**: Red vignette animates when player at 1 HP (critical state)
- **Combo pop text**: Shows "X KILLS!" every 10 consecutive kills
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables/dampens animations

## Performance Considerations

1. **Adaptive Quality**: Tracks frame times over 45 frames; reduces particle count if avg FPS < 40
2. **Canvas sizing**: Scales canvas to fit viewport (max 1.6x for mobile, respects safe areas)
3. **Object pooling**: Not used; particles/bullets are simply filtered out when dead
4. **Shadow rendering**: Only applied at quality level 0 (full quality); disabled on lower tiers

## Accessibility & Usability

- **Screen reader support**: aria-live status announcements for game events
- **Reduced motion toggle**: Button in top-right; persists to localStorage
- **Mute toggle**: Audio can be disabled; persists to localStorage
- **High contrast**: Colors chosen for visibility (gold scores, green weapons, red health)
- **Touch-friendly**: Mobile controls with 78px buttons; safe-area-inset support for notched devices

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (around line 502)
2. Update `spawnEnemy()` (line 1022) to choose it based on stage/difficulty
3. Add corresponding explosion sound in `EXPLODE_SFX` (line 678) if desired

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508)
2. Create attack patterns in `BOSS_PATTERNS` array (line 1372)
3. Boss is auto-selected by stage modulo: `(stage - 1) % BOSSES.length`

### Adjusting Difficulty
- Modify `DIFFICULTIES` object (line 485) to change spawn rates, lives, bomb count
- `bossHpMul`: Multiplier applied to base boss HP

### Tweaking Game Feel
- `KILLS_PER_STAGE` (line 491): How many kills trigger boss spawn
- Player speed (line 877): `player.speed = 260`
- Weapon fire rates (lines 1177–1178): `fireDelay`
- Particle count reduction formula (line 1074)

## Testing Notes

- **No automated tests**: This is a game; test by playing
- **Mobile testing**: Use browser DevTools device emulation or deploy to test touch controls
- **Performance**: Monitor FPS via DevTools; quality will auto-downgrade if needed
- **Audio**: Test with both audio on and muted; synth sounds should be crisp with no pops

## Common Editing Tasks

### Adjust visual theme
Edit `THEMES` array (line 493) to modify sky gradients and cloud colors for new stages

### Change game difficulty
Modify values in `DIFFICULTIES` object (line 485)

### Update player or enemy visuals
`drawPlane()` (line 913) draws all ships; modify path commands or colors there

### Add a new sound effect
Use `beep()` or `noiseBurst()` in the `sfx` object (line 624); see existing examples for parameter patterns

### Fix game balance
Adjust `spawnEnemy()` thresholds, boss pattern timings, or damage values (boss hit: line 1259, bomb damage: line 1118)

## Key Variables Reference

- `running`: Game is active
- `paused`: Game is paused (overlay shown)
- `gameOver`: Game ended (player lost all lives)
- `stage`: Current stage (1+)
- `score`: Current game score
- `multiplier`: Current kill chain multiplier (1–10x)
- `difficulty`: 'easy' | 'normal' | 'hard'
- `quality`: 0 (full) | 1 (medium, fewer particles) | 2 (low, minimal particles)
