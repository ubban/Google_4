# Sky Fighter - Developer Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The game features fast-paced arcade action with weapon progression, smart bombs, boss encounters, and multiple difficulty levels.

**Repository**: ubban/Google_4  
**Branch**: claude/inspiring-planck-k8iabw  
**Live file**: index.html (single-file HTML5 application)

## Architecture

### Single-File Design
The entire game is contained in a single `index.html` file (~1500 lines) with:
- **Inline CSS**: All styling (dark theme, animations, UI components)
- **Inline JavaScript**: Core game engine and logic wrapped in an IIFE

### Core Game Components

#### Game State Management
- `gameState`: Main object tracking game progress (score, lives, stage, multiplier)
- `difficulty`: Current difficulty settings affecting enemy spawn rates, AI aggression, health scaling
- `quality`: Adaptive performance level (0=full, 1=medium, 2=low) based on frame timing

#### Canvas & Rendering
- `canvas` / `ctx`: 2D canvas context for all drawing
- Game updates at 60 FPS via `requestAnimationFrame`
- Adaptive quality rendering reduces particles/effects on lower-end devices

#### Entity Systems

**Player**
- Single player aircraft with health (lives: 3-4 depending on difficulty)
- Position tracking and bounds checking
- Weapon system with multiple types and levels
- Bomb mechanic (limited use, screen-clearing damage)

**Enemies**
- Multiple enemy types with different behaviors
- Wave spawning based on difficulty and stage
- Basic AI: move toward player, shoot at intervals
- Drops: weapon power-ups, health, bombs

**Boss Encounters**
- One boss per stage with scaled health pools
- Special boss health bar UI component
- Phase-based behavior (increased aggression)

**Projectiles**
- Player bullets (single/spread patterns based on weapon)
- Enemy fire (basic straight-line projectiles)
- Smart bomb explosion wave

**Visual Effects**
- Combo popups (damage feedback)
- Stage banners (visual transitions)
- Flash effects (hit reactions, bomb detonation)
- Vignette pulse (damage warning)

#### UI & Controls

**HUD Elements**
- Score display
- Best score (persisted to localStorage)
- Weapon name and level
- Lives (heart display)
- Bombs remaining
- Multiplier (score multiplier for consecutive kills)
- Boss health bar (shown only during boss fights)

**Input Methods**
- **Desktop**: Mouse (aim), Click (shoot), Arrow/WASD (move), Space (shoot), X/Shift (bomb), P/Esc (pause)
- **Mobile**: Touch-based joystick (left zone), Fire/Bomb buttons (right zone)
- **Accessibility**: Screen reader support via #srLive ARIA live region

**Settings**
- Mute/Sound toggle (persisted)
- Reduced motion mode (accessibility feature)
- Pause function
- Difficulty selection (Easy/Normal/Hard)

#### Game Progression
- `KILLS_PER_STAGE = 18`: Number of enemies to defeat before advancing
- 6 visual themes with different sky gradients (changes per stage or randomly)
- Difficulty multipliers scale enemy stats and health

#### Storage (localStorage)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion preferences

## Key Game Systems

### Difficulty Balancing
Three preset configurations in `DIFFICULTIES` object:
- **Easy**: Longer spawn intervals, slower enemies, higher player health
- **Normal**: Balanced defaults (selected by default)
- **Hard**: Rapid spawning, aggressive enemies, higher boss HP

Each affects: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier

### Performance Adaptation
The engine monitors frame times and automatically downgrades render quality:
- Maintains 45-frame rolling average of delta time
- Upgrades/downgrades quality tier based on average frame rate
- Lower quality: reduced particle count, simpler effects

### Weapon Progression
- Starts with VULCAN (basic rapid fire)
- Power-ups add new weapons (spread, burst patterns)
- Weapon level increases damage/fire rate
- Current weapon displayed in HUD

### Score Multiplier
- Increases with consecutive kills without being hit
- Resets on player damage
- Displayed in orange in the HUD
- Increases score earnings exponentially

### Boss Encounters
- One boss per stage
- Health bar appears during fight
- Boss pattern varies by stage
- Health scales with difficulty setting

## File Structure

```
Google_4/
├── index.html       # Complete game (HTML + CSS + JS)
├── README.md        # Minimal project description
└── CLAUDE.md        # This file
```

## Common Tasks

### Running the Game
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. Click "Start Game" to begin
3. Select difficulty level if desired
4. Use mouse (click to shoot, move to aim) or keyboard controls

### Making Game Balance Changes
Look for these constants in the JavaScript section:
- `DIFFICULTIES` object (line ~485): spawn rates, speeds, health
- `KILLS_PER_STAGE` (line ~491): stage progression threshold
- Weapon configs in the game initialization code

### Adding Visual Polish
CSS animations are defined in the `<style>` block:
- `@keyframes heartBeat`, `comboPulse`, `stagePulse`, `flashHit`, `flashBomb`
- HUD styling: `.stat`, `.iconBtn`, `.diffBtn`
- Modify colors in CSS variables or inline rgba values

### Fixing Bugs
Common issue areas:
- **Collision detection**: Search for functions checking distance between entities
- **Touch controls**: Mobile input handling in `#moveZone` / `#actionZone` handlers
- **Mobile responsiveness**: Canvas sizing logic and viewport meta tags
- **Accessibility**: ARIA live region updates via `announce()` function

### Adding Features
1. **New enemy type**: Define movement/shoot patterns, add to spawn logic
2. **New weapon**: Create weapon object with bullet pattern, add to power-up drops
3. **New theme/stage visual**: Add entry to `THEMES` array
4. **Audio system**: Placeholder for future audio context setup

## Browser Compatibility
- Requires ES6+ JavaScript support
- Canvas 2D context (universal on modern browsers)
- LocalStorage API for persistence
- Touch Events API for mobile input
- CSS Grid/Flexbox for layout

## Performance Considerations
- Single canvas rendering pass per frame
- Particle pooling/reuse recommended for optimization
- Mobile devices automatically reduce effect complexity
- Frame rate monitoring ensures stable 60 FPS target

## Notes for Claude Code Sessions
- All changes are made directly in `index.html`
- Test changes in browser immediately (no build step needed)
- Monitor performance on mobile devices (adaptive quality scaling exists)
- Preserve accessibility features when modifying UI
- Keep localStorage keys consistent for user continuity
- Maintain CSS animation compatibility with `prefers-reduced-motion` media query

## Future Enhancements
- Audio system (background music, SFX)
- Leaderboard/Cloud sync
- Additional enemy/boss patterns
- Power-up variety (shields, speed boost)
- Stage progression with custom themes per stage
- Particle effect expansion
- Combo multiplier progression system
