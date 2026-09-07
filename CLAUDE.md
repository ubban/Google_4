# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application with no external dependencies. It features:
- Multiple difficulty settings (easy, normal, hard)
- Six unique boss encounters with distinct attack patterns
- Two weapon types (Vulcan and Laser) with upgradeable levels
- Smart bomb power-ups and defensive mechanics
- Procedurally enhanced stages with thematic variations
- WebAudio synthesis for all sound effects (no audio assets)
- Full touch and keyboard input support

## Architecture

The entire application logic lives in `/home/user/Google_4/index.html` within a single IIFE (Immediately Invoked Function Expression) that encapsulates all game state.

### Core Systems

**Game Loop** (line 1640–1652)
- RequestAnimationFrame–based update/render cycle
- Delta time capped at 50ms to prevent large time jumps
- Performance tracking adapts quality (line 473–483) based on frame timing

**State Management**
- Game state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Persistent state: best score, difficulty, mute, reduced motion (via localStorage)
- Input state tracked via `keys` object and derived states (`mouseActive`, `joyActive`, `firing`)

**Game Phases**
1. **Menu**: Overlay shows difficulty selection, controls hint, start button
2. **Playing**: Active game loop with enemy spawning, collision detection, scoring
3. **Paused**: Game frozen, overlay shows pause screen
4. **Game Over**: Shows final stats and high-score indicator

### Key Subsystems

**Difficulty Settings** (line 485–489)
- Scales spawn rates, enemy speed/shoot chance, starting lives/bombs, boss HP
- Settings stored in localStorage for persistence

**Enemy AI** (line 1022–1041, 1210–1227)
- Three enemy types: grunt (fast, weak), interceptor (evasive, weave pattern), tank (slow, tough)
- Each has configurable spawn likelihood based on stage progression
- Enemies use either spread-shot or aimed-shot patterns; weaving interceptors move in sine waves

**Boss Design** (line 1043–1056, 1372–1439)
- Six bosses rotate per stage with unique appearance (gradient hulls, animated cores) and attack patterns
- Attack patterns defined in `BOSS_PATTERNS` array: combines radial bursts, aimed spreads, homing volleys, teleportation
- Boss telegraphs incoming attacks with visual effect and audio cue before firing
- Each boss design has an `hpMul` that scales its total HP

**Weapon System** (line 1124–1146, 1304–1322)
- Vulcan: rapid-fire projectiles with cone spread as level increases
- Laser: high-speed piercing shots; can penetrate multiple enemies
- Power-ups (20% spawn rate on enemy death) switch weapons or level them up (cap: level 3)

**Scoring & Chain Multiplier** (line 1087–1101)
- Each kill extends chain timer (2.2 seconds); multiplier caps at 10x
- Chain breaks on player damage or timer expiration
- Combo milestones (10, 20, 30 kills) trigger on-screen visual feedback

**Visual & Audio**
- Canvas renders in 480×720px (scaled to viewport with aspect-ratio lock)
- Parallax cloud layers per-stage theme (6 themes with unique palettes)
- All sounds synthesized via WebAudio oscillators and noise generators
- Screen shake, flash effects, and vignette pulse tied to damage and bombs
- Reduced motion mode disables animations for accessibility

## Development Commands

Since this is a no-build project, development is straightforward:

```bash
# Open the game in a browser (from the repository root)
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

### Testing

- **Full game flow**: Menu → all three difficulty modes → play through multiple stages
- **Collision detection**: Verify player/enemy/bullet overlaps trigger correctly
- **Power-up spawning**: Kill ~15 enemies per type to observe power-up drop rates
- **Boss patterns**: Reach stage 2+ to trigger bosses; confirm each boss fires correct pattern set
- **Audio synthesis**: Toggle mute and confirm beeps/noise bursts play in-game
- **Responsiveness**: Test on mobile (touch controls) and desktop (mouse + keyboard)

### Code Style

- Single-file structure; no module system
- All logic encapsulated in IIFE to avoid global pollution
- Magic numbers embedded inline (e.g., spawn timers, boss HP formulae); extract to constants if adding new difficulty tiers
- Canvas drawing functions grouped by subject (`drawPlane`, `drawBossShip`, etc.)
- Event listeners attached directly to DOM elements; no delegation layer

## Key Constants & Configuration

**Game Dimensions** (line 753)
- BASE_W: 480px, BASE_H: 720px (portrait-oriented)

**Difficulty Table** (line 485–489)
- `spawnBase` / `spawnMin`: Enemy spawn intervals (frames)
- `enemySpeed`: Multiplier on base enemy velocities
- `enemyShootChance`: Probability [0–1] that spawned enemy fires
- `lives`, `bombs`, `bossHpMul`: Difficulty-specific starting resources and scaling

**Themes** (line 493–500)
- Six named themes with sky gradients, cloud colors, sun tint
- Cycles per stage: `(stage - 1) % THEMES.length`

**Enemy Types** (line 502–506)
- `grunt`, `interceptor`, `tank` with HP, speed, color, score, stage requirement

**Boss Roster** (line 508–515)
- VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has hull colors, core glow, and `hpMul` scaling

## Common Edits

**Add a New Enemy Type**
1. Add entry to `ENEMY_TYPES` object (line 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn logic in `spawnEnemy()` (line 1022–1041) to set probability thresholds
3. Add SFX handler if needed in `EXPLODE_SFX` map (line 678) and corresponding `sfx` function

**Add a New Boss Pattern**
1. Define pattern function (e.g., `fireCustomPattern(b)`) using existing radial/aimed/column helpers (line 1324–1371)
2. Add to `BOSS_PATTERNS` array at the stage index (line 1372–1409)
3. Increase `BOSSES` array length or adjust the modulo in `spawnBoss()` if needed

**Adjust Difficulty Settings**
- Edit `DIFFICULTIES` object (line 485–489)
- All difficulty-dependent logic reads from `const d = DIFFICULTIES[difficulty]` in `update()`

**Add a New Theme**
- Append object to `THEMES` array (line 493–500)
- Provide `sky` [3-stop gradient array], `cloud` [RGB string], `sun` [RGB string], `name`

## localStorage Keys

All user preferences stored client-side:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle (boolean as '0'/'1')
- `skyfighter.difficulty`: Selected difficulty ('easy'/'normal'/'hard')
- `skyfighter.reducedMotion`: Accessibility preference (boolean as '0'/'1')

## Performance Considerations

- **Adaptive quality** (line 473–483): Samples last 45 frames; downgrades rendering if avg FPS drops below ~40
- **Particle budget** (line 1074): Explosion particle count scaled by quality setting (full → medium → low)
- **Canvas shadow effects**: Glow on bullets/enemies only enabled at full quality; reduces fillRate on lower-end devices
- **Touch event prevention**: Non-passive listeners on move zone and buttons to block default zoom behavior

## Accessibility

- Screen reader announcements via `#srLive` (ARIA live region) for key game events
- Reduced motion toggle respects system preference (`prefers-reduced-motion` media query)
- Color indicators (❤️, 💣, emojis) supplemented with text labels in HUD
- Cross-hair cursor signals mouse steering is active
