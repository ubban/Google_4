# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in a single HTML file (`index.html`). The game features:
- Canvas-based 2D rendering
- WebAudio synthesis for sound effects (no external audio files)
- Multiple difficulty levels with adaptive performance
- Weapon and bomb systems
- Boss encounters with health bars
- Touch controls for mobile devices
- Accessibility features (reduced motion, screen reader support)

## Architecture

The entire game is contained in `index.html` (~1669 lines) with three main sections:

### 1. HTML Structure (Lines 2-420)
- **Canvas element**: `#gameCanvas` - the main game rendering surface
- **UI Elements**: HUD (score, health, weapons), boss health bar, pause overlay
- **Touch Controls**: Mobile movement zone and action buttons
- **Accessibility**: Screen reader live region (`#srLive`)

### 2. CSS Styles (Lines 10-359)
- **Responsive design**: Uses `clamp()` for adaptive font sizes based on viewport
- **Animations**: Keyframe animations for UI elements (combo popups, stage banners, heart beat)
- **Dark theme**: Game uses a dark blue color scheme (`#05122b`, `#0b1d3a`)
- **Motion accessibility**: `body.reduced-motion` class disables all animations when preferred

### 3. JavaScript Game Engine (Lines 422-1669)
Organized in a strict self-invoking function to avoid global scope pollution.

#### Core Game Loop
- **Frame-based**: Uses `requestAnimationFrame` for 60+ FPS targeting
- **Adaptive performance**: Tracks frame time to adjust quality (shadow/glow effects) dynamically
- **Pause state**: Can pause and resume without resetting game state

#### Key Systems

**Entity Management**
- `player`: Object with position, health, weapon, invulnerability state
- `bullets`: Array of player bullets (either normal or laser type)
- `enemyBullets`: Array of enemy projectiles
- `enemies`: Array of active enemies with types: `grunt`, `interceptor`, `tank`
- `particles`: Visual effects (explosions, score popups)

**Difficulty & Progression**
- `DIFFICULTIES` object: Defines spawn rates, enemy speed, lives, bombs for each difficulty
- `THEMES`: Six different visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) that cycle per stage
- `BOSSES`: Six unique boss types with distinct visual designs and HP multipliers
- `stage` counter: Increments after `KILLS_PER_STAGE` (18) kills; stage affects boss selection and theme

**Weapon System**
- `player.weapon`: Current weapon type (`'vulcan'`, `'laser'`, `'spread'`)
- `player.weaponLvl`: Weapon level (1-3) affects firing rate and damage
- Weapon upgrades are placed randomly on killed enemies
- Each weapon has distinct firing patterns and visual effects

**Bomb System**
- `player.bombs`: Current bomb count (0-3)
- Uses WebAudio to create a powerful sound burst effect when detonated
- Clears all enemy bullets and deals damage to bosses

**Audio (WebAudio Synthesis)**
- All sound effects are generated in real-time using WebAudio API
- No external audio files needed
- Audio context initialized on first user interaction due to browser autoplay policies
- `ensureAudio()` function ensures context exists before playing sounds
- Mute state persists in localStorage

**Input Handling**
- **Keyboard**: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Movement and click to fire
- **Touch**: Drag left zone to move, tap fire/bomb buttons
- Touch controls are automatically enabled on mobile and disabled on desktop

**Rendering Pipeline**
1. Clear canvas and apply screen shake (if hit)
2. Draw background (gradient sky with sun glow)
3. Draw parallax clouds at different speeds
4. Draw player with invulnerability blink
5. Draw all bullets and enemy bullets
6. Draw enemies with color variations
7. Draw particles (explosions, score popups)
8. Apply vignette effect during damage
9. Render HUD and UI overlays

## Common Development Tasks

### Running the Game
Simply open `index.html` in a modern web browser. No build process is needed.

### Testing Gameplay
- Use difficulty buttons to switch between Easy, Normal, and Hard
- Press `P` or `Esc` to pause/unpause
- Press mute button (🔊) to toggle audio
- Press motion button (🎆) to toggle screen shake and animations

### Adding New Content

**New Enemy Type**
Add to `ENEMY_TYPES` object (~line 502):
```javascript
newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#color', dark: '#darkcolor', score: 20, minStage: 3 }
```

**New Boss**
Add to `BOSSES` array (~line 508):
```javascript
{ name: 'BOSSNAME', hullTop: '#color1', hullBottom: '#color2', wing: '#color3', core: '#color4', hpMul: 1.0 }
```

**New Theme**
Add to `THEMES` array (~line 493):
```javascript
{ sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```

### Modifying Difficulty Parameters
Edit `DIFFICULTIES` object (~line 485) to adjust:
- `spawnBase`: Base frame counter before spawning enemies
- `spawnMin`: Minimum frames between spawns (lower = more enemies)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability per frame that enemies fire
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss health multiplier

### Adjusting Game Balance
- `KILLS_PER_STAGE` (~line 491): Number of kills needed to progress to next stage
- Enemy spawn locations and patterns: See `updateEnemies()` function
- Weapon fire rates and spread patterns: See `shootVulcan()`, `shootLaser()`, `shootSpread()`

## Performance Considerations

### Adaptive Quality System
The game monitors frame time and automatically adjusts visual quality:
- **Quality 0** (Full): Glows around bullets and enemies, shadow effects
- **Quality 1** (Medium): Some effects disabled
- **Quality 2** (Low): Minimal visual effects for low-end devices

This is controlled by the `quality` variable and `trackPerf()` function (~line 476).

### Canvas Rendering Optimization
- Bullets use simple `fillRect()` (not arc/path)
- Enemies use pre-calculated drawing paths
- Screen shake uses canvas translate for efficiency
- Particles (explosions) have maximum count limits

## Storage & Persistence

Game uses localStorage for:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (boolean)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference for motion

## Accessibility Features

- **Screen reader support**: Uses `announce()` function to provide feedback to assistive devices
- **Reduced motion mode**: Disables animations and screen shake for users who prefer it
- **High contrast**: Game uses bright colors on dark background for readability
- **Keyboard-only play**: Full game can be played without mouse/touch
- **Touch-friendly controls**: Large hit targets (78px buttons) on mobile
