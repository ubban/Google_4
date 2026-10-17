# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML/JavaScript with Canvas rendering. The game is a single-file application (`index.html`) with no build system, dependencies, or external assets.

### Key Features
- 6 themed stages with dynamic sky backgrounds
- 6 boss encounters with distinct attack patterns
- 3 enemy types (Grunt, Interceptor, Tank) that scale with difficulty
- 2 weapon systems (Vulcan and Laser) with 3 upgrade levels
- Smart bomb mechanic
- Score multiplier / combo system based on consecutive kills
- 3 difficulty modes (Easy, Normal, Hard)
- Full Web Audio synthesis (no external audio files)
- Mobile touch controls with on-screen joystick
- Keyboard and mouse controls
- LocalStorage persistence for best score, preferences, and settings
- Reduced motion accessibility support
- Adaptive performance quality system (adjusts particle count and effects)

## Running the Game

Simply open `index.html` in a web browser. No build step, no server, no dependencies needed.

**Local development**: You can use any simple HTTP server if needed for CORS or to test on different devices:
```bash
python3 -m http.server 8000
# or with Python 2: python -m SimpleHTTPServer 8000
# then visit http://localhost:8000
```

## Architecture & Code Structure

The entire game runs in an IIFE (Immediately Invoked Function Expression) in `<script>` tags within `index.html`. The code is organized logically:

### 1. **DOM References & Constants** (lines 426-527)
- Canvas and UI element references
- Storage keys for localStorage
- Game constants (DIFFICULTIES, KILLS_PER_STAGE, THEMES, ENEMY_TYPES, BOSSES)

### 2. **Audio System** (lines 542-676)
- WebAudio context initialization (`ensureAudio()`)
- Synthesized sound effects using oscillators, noise, and filters
- `beep()` and `noiseBurst()` for tone generation
- `sfx` object with pre-configured sound effects (shoot, laser, explode, bomb, etc.)

### 3. **UI & Input Handling** (lines 680-861)
- Heart and bomb display rendering
- Flash, combo, and stage banner visual effects
- Screen shake system
- Difficulty button handling
- Canvas resizing
- Keyboard input (arrows, WASD, Space, X, P, Escape)
- Mouse steering (cursor follows, click to fire)
- Touch controls (joystick on left, buttons on right)

### 4. **Game State & Core Logic** (lines 863-1302)
- Game state variables (player, bullets, enemies, bosses, particles, score, lives, etc.)
- `resetGame()`: Initializes a new game
- `update(dt)`: Main game loop update logic
  - Player movement from keyboard/mouse/touch input
  - Bullet lifecycle and cleanup
  - Enemy spawning with difficulty-based rates
  - Boss spawn triggering (after KILLS_PER_STAGE kills)
  - Collision detection (bullets to enemies, player to hazards, power-ups)
  - Weapon firing
  - Chain/combo multiplier logic
- Enemy AI with two attack patterns: spread-shot and aimed (player-tracking)
- Weaving movement for interceptors

### 5. **Boss System** (lines 1043-1453)
- `spawnBoss()`: Creates a boss with design data
- 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Boss patterns array (BOSS_PATTERNS) with 6 pattern sets:
  - Each boss has 3 attack patterns that cycle
  - Patterns include: radial bursts, aimed spreads, columns, spirals, crosses, homing missiles
  - Boss telegraph/warning system before attacks
- `updateBoss(dt)`: Boss movement and attack scheduling
- `killBoss()`: Triggers stage progression and next boss spawn

### 6. **Rendering System** (lines 1507-1638)
- `draw()`: Renders the scene each frame
- Theme-based sky gradients and backgrounds
- Cloud parallax at two speeds
- Player ship with weapon-specific visual feedback
- Enemies, bullets, particles, power-ups
- Boss ship with animated glowing core
- Adaptive glow effects based on performance quality setting
- `drawPlane()`: Procedural ship drawing (used for player and enemies)
- `drawBossShip()`: Procedural boss rendering with animation

### 7. **Game Loop** (lines 1640-1666)
- `loop(now)`: requestAnimationFrame loop
- Delta-time calculation with 50ms cap
- Performance tracking and quality adjustment
- Update and draw phases only when game is running and not paused

## Key Game Mechanics

### Difficulty System
Three pre-set difficulty modes affect spawn rate, enemy behavior, and boss HP multiplier. Settings are persisted to localStorage.

### Weapon Progression
- **Vulcan**: Spread shot, upgrades increase horizontal spread
- **Laser**: Focused vertical, upgrades add side lanes and pierce through enemies
- Weapon powerups cycle between weapons or upgrade current level
- Laser weapons can pierce through multiple enemies with a pierce counter

### Scoring & Chain
- Kills within 2.2 seconds of the last kill extend the chain
- Multiplier increases by 1 for every 5 kills in a chain (1x → 10x max)
- Chain breaks on taking damage or timing out
- Boss kills grant fixed 500 points

### Enemy Types
- **Grunt**: Basic red enemies, spawns from stage 1
- **Interceptor**: Fast orange enemies with weaving movement, spawns from stage 1
- **Tank**: Heavy purple enemies with 4 HP, spawns from stage 2+

### Stage & Boss Progression
- Each stage requires 18 kills (`KILLS_PER_STAGE`) to unlock the boss
- 6 bosses cycle based on (stage - 1) % 6
- Boss HP scales with difficulty and stage number
- Defeating a boss progresses to the next stage and theme

### Power-ups
- **Life** (red, 6% chance): +1 HP
- **Bomb** (yellow, 8% chance): +1 smart bomb
- **Weapon** (green, 20% chance): Weapon upgrade or switch
- Drop from enemy kills, fade if not collected

### Accessibility
- Reduced motion preference respects `prefers-reduced-motion` media query
- Toggle button in top-right to override
- Disables animations when active (screen shake, flash, vignette pulse, etc.)

## Performance Optimization

The game tracks frame times and adjusts quality dynamically:
- **Quality 0** (Full): All visual effects, full particle counts, shadow glows
- **Quality 1** (Medium): ~60% particle count, reduced glow
- **Quality 2** (Low): ~35% particle count, no glow
- Quality degrades if average frame time > 1/40s, improves if < 1/56s

## Making Changes

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add explosion sound effect to `EXPLODE_SFX` if needed
3. Adjust spawn logic in `spawnEnemy()` to include the new type

### Adding a New Boss
1. Add to `BOSSES` array with hull colors, wing, core, and `hpMul`
2. Create a pattern set in `BOSS_PATTERNS` (3 attack functions each)
3. Pattern functions use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
4. Boss design automatically cycles based on (stage - 1) % BOSSES.length

### Adjusting Difficulty
Modify the `DIFFICULTIES` object properties:
- `spawnBase` / `spawnMin`: Enemy spawn rate (frames between spawns)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemy will shoot when in range
- `lives` / `bombs`: Starting counts
- `bossHpMul`: Boss health multiplier

### Adding a New Theme
Add to `THEMES` array with sky gradients, cloud color, sun color, and name. Themes cycle based on (stage - 1) % THEMES.length.

### Creating New Sound Effects
Use the `beep()` and `noiseBurst()` functions in the `sfx` object. Options include frequency, duration, type, volume, sweep targets, filters, panning.

## Testing Locally

Open the game in your browser and test:
- **Gameplay**: All three difficulty modes, boss patterns
- **Input**: Keyboard controls (arrows, WASD, Space, X), mouse steering, touch controls on mobile
- **Visuals**: All 6 themes display correctly, reduced motion toggle works
- **Audio**: Sound effects play, mute button works
- **Persistence**: Close and reopen, best score and preferences persist
- **Performance**: No frame drops on lower-end devices (quality auto-adjusts)

## Git Workflow

This is a single-file project. When making changes:
1. Edit `index.html` directly
2. Test in browser
3. Commit with a clear message describing the change
4. Push to the designated branch

Common commit patterns:
- Feature additions: "Add [feature] system" (e.g., "Add third boss pattern")
- Bug fixes: "Fix [issue]" (e.g., "Fix laser pierce not working on tanks")
- Balance: "Adjust [parameter]" (e.g., "Adjust hard difficulty spawn rates")
- Visual: "Update [element]" (e.g., "Update boss core glow effect")
