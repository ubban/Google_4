# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features:
- Player-controlled aircraft with weapon progression
- Multiple difficulty levels (Easy, Normal, Hard)
- Enemy types with varying behaviors and health
- Boss encounters with individual visual themes
- Smart bomb mechanics for area damage
- Touch and keyboard controls
- Procedural stage progression with kill-based advancement
- LocalStorage persistence for best score and settings
- Adaptive performance scaling based on frame time

## Architecture

The entire application lives in a single **index.html** file (~1,669 lines) combining HTML markup, embedded CSS, and JavaScript game logic. There is no build process, package manager, or external dependencies — the file is self-contained and runs directly in any modern browser.

### Code Structure (in index.html)

1. **HTML Structure** (lines 1-361)
   - Canvas for game rendering
   - HUD overlay with score, best, multiplier, weapon level, hearts, bombs
   - Boss health bar
   - Top-right control buttons (mute, motion effects toggle, pause)
   - Touch-based mobile controls (joystick zone, FIRE/BOMB buttons)
   - Main overlay for menu and end-game screens
   - Screen-reader-only live region for accessibility

2. **Embedded CSS** (lines 10-358)
   - Full responsive design with `clamp()` for fluid scaling across device sizes
   - Dark blue theme with accent colors (gold, neon green, red-orange)
   - CSS animations for UI elements: combo popups, stage banners, pulsing effects, flashes
   - Safe-area insets for notched devices
   - Reduced motion media query support

3. **JavaScript Game Logic** (lines 422–1669)
   - **Initialization & State Management**: Game state object tracks player, enemies, bosses, score, difficulty
   - **Performance Tracking**: Adaptive quality system that scales enemy spawning and visual effects based on FPS (lines 473–483)
   - **Audio System**: WebAudio API synth for all sound effects; no external audio files (lines 542–620)
   - **Difficulty Configuration**: Three presets with spawn rates, enemy speed, shooting chance, and boss health multipliers (lines 485–489)
   - **Theme System**: Six procedurally-selected sky themes with varying color schemes (lines 493–500)
   - **Enemy Types**: Grunt (basic), Interceptor (fast + weaving), Tank (slow + durable) (lines 502–506)
   - **Boss Types**: Six distinct bosses with unique color schemes and health multipliers (lines 508–515)
   - **Game Loop**: Main `gameLoop()` using `requestAnimationFrame` that updates all entities, handles collisions, and re-renders canvas (approximately line 900+)
   - **Input Handling**: Keyboard (arrow keys, WASD, space, X/Shift, P/Esc), mouse movement and click, touch (drag/tap)
   - **Touch Controls**: Adaptive joystick/button layout for mobile devices

### Key Game Systems

**Stage Progression**
- Players advance stages by destroying 18 enemies (defined by `KILLS_PER_STAGE`)
- Themes and bosses cycle across stages
- Difficulty parameters scale spawn rates as stage increases

**Player Mechanics**
- Lives system (3 by default, 4 on Easy)
- Smart bomb capacity (2 by default, 3 on Easy); regenerates per stage
- Weapon system with upgradeable levels (VULCAN → CANNON → LASER)
- Collision-based damage on enemy contact
- Score multiplier for consecutive hits without taking damage

**Enemy Spawning**
- Spawn rate decreases as player advances (spawnBase → spawnMin)
- Enemy types depend on stage and selected difficulty
- Enemies shoot bullets with difficulty-based chance
- Speed and shooting patterns scale with difficulty and spawn rate

**Collision & Damage**
- Player projectile ↔ enemy collision triggers enemy death
- Enemy projectile ↔ player collision causes health loss
- Smart bomb radius-based damage to all enemies and bullets
- Boss collision damage is handled separately

**LocalStorage Persistence**
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion effects preference

## Development & Modification

### Running the Game
Simply open `index.html` in any modern web browser. No server, build step, or dependencies required.

### Updating Game Parameters
All configurable constants are defined near the top of the JavaScript section (after line 422):
- `DIFFICULTIES` (line 485): Adjust spawn rates, speeds, chances, lives
- `KILLS_PER_STAGE` (line 491): How many kills to advance
- `THEMES` (line 493): Sky colors and visual themes
- `ENEMY_TYPES` (line 502): Enemy stats (size, HP, speed, score, appearance)
- `BOSSES` (line 508): Boss names, colors, health multipliers

### Modifying Game Mechanics
The main game loop typically runs in the second half of the script. Look for:
- `function gameLoop(dt)`: Main update loop
- `function update()`: Game state updates (player, enemies, bullets, bosses)
- `function draw()`: Canvas rendering of all game entities
- `function handleCollisions()`: Bullet ↔ enemy and enemy projectile ↔ player collision detection
- Input handlers: keyboard/mouse/touch event listeners

### Audio Design
All sound is procedurally generated using WebAudio oscillators and noise. To add or modify sounds:
- `beep(freq, dur, type, vol, opts)`: Single-frequency tone with optional sweep or filter
- `noiseBurst(dur, vol, opts)`: Noise-based sound (explosions, hits)
- Sounds are played with spatial panning based on game entity position (see `panFor()`)
- Mute state is respected in both functions

### Accessibility
- Game supports reduced-motion preferences (CSS + input handling)
- Screen-reader-only `#srLive` region announces key events
- Semantic HTML button elements for controls
- High contrast colors for visibility
- Mobile-friendly touch targets (78px diameter buttons)

### Performance Optimization
The game includes adaptive quality scaling (lines 473–483):
- Tracks frame times over 45 frames
- Reduces visual quality (enemy count, effect density) if average frame time exceeds 25ms (40 FPS)
- Increases quality if maintaining <17.86ms average (56 FPS)
- Quality levels affect enemy spawning logic

## Browser Compatibility
- Modern browsers with Canvas API support
- WebAudio API (with webkit prefix fallback)
- LocalStorage for persistence
- CSS Grid and Flexbox for layout
- Responsive design tested on desktop and mobile (touch support)

## Tips for Future Work
- Always test on both desktop (mouse/keyboard) and mobile (touch) after changes
- Performance changes: verify with slow devices or reduced CPU via DevTools throttling
- Audio changes: test on both muted and unmuted states
- Difficulty balance: Easy should feel forgiving, Hard should feel challenging — spawn and shoot rates are key
- Theme addition: add entries to `THEMES` array with RGB sky colors and alternate sun color
- New enemy type: add to `ENEMY_TYPES` with stats; remember `minStage` to control appearance timing
- Boss tuning: health multipliers in `BOSSES` array control difficulty; visual design (colors) is cosmetic
