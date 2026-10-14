# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The entire codebase is in `index.html` (1669 lines), containing:

- **HTML structure**: Canvas, overlay UI, HUD, touch controls
- **CSS**: Dark theme, responsive layout with clamp() for scaling, animations
- **JavaScript**: Game logic in a single self-executing function

The game is fully self-contained—no build process, no dependencies. Just open `index.html` in a browser to play.

## Running the Game

**View/test locally:**
```bash
# Simple HTTP server (Python 3)
python3 -m http.server 8000

# Or Node.js
npx http-server

# Then open: http://localhost:8000
```

**No linting, testing, or build steps exist.** The codebase is a single file designed for direct browser execution.

## Game Architecture & Features

### Core Game Loop
The JavaScript code uses a `requestAnimationFrame` pattern for rendering. Game state is tracked globally within the self-executing function:
- Player position, health, bombs, score, multiplier
- Active enemies, projectiles, pickups
- Current difficulty, stage progress, boss state
- Adaptive performance tracking (adjusts particle/effect quality based on frame time)

### Key Mechanics

**Difficulty System** (3 levels: Easy, Normal, Hard)
- Affects spawn rates, enemy speed, shoot chances, health pools
- Stored in `STORAGE_DIFF` (localStorage)
- Difficulty buttons in the overlay determine initial game state

**Weapon System**
- Multiple weapons: Vulcan, Spread, Homing (4 types total)
- Each has 3 levels (stored as `weaponLvl`)
- Weapons are upgraded via pickups or stage progression
- Current weapon name displayed in HUD

**Boss Encounters**
- Boss appears after defeating KILLS_PER_STAGE (18) enemies per stage
- Boss health scaled by difficulty (`bossHpMul`)
- Health bar shown in top center when boss is active
- Boss defined in a pattern (see THEMES array for thematic variations)

**Scoring & Multiplier**
- Base score from kills, multiplied by current multiplier
- Combo system: consecutive hits increase multiplier
- Best score saved to localStorage (`STORAGE_BEST`)

**Themes** (6 color schemes)
- Each stage/run selects a theme that affects sky gradient, clouds, sun colors
- Stored in THEMES array (sky colors, cloud/sun RGB values, name)

### Touch Controls (Mobile)
- Left 55% of screen: move zone (drag to fly)
- Right 45% of screen: action zone (Fire and Bomb buttons)
- Joystick visualization appears on touch
- Desktop uses mouse/keyboard instead

### Accessibility Features
- **Screen reader support**: `#srLive` region announces game events
- **Reduced motion**: Toggle via motion button; disables animations for users with motion sensitivity
- **Responsive design**: Uses `clamp()` for font and spacing scaling
- **Semantic HTML**: ARIA labels, proper button roles

### Performance Optimization
- Adaptive quality system: tracks frame times over 45 frames, adjusts rendering quality (full → medium → low) if avg frame time > 1/40s
- Disables heavy effects on low-end devices

## Key State Variables & Storage

```javascript
const STORAGE_BEST = 'skyfighter.bestScore';      // Best score ever
const STORAGE_MUTE = 'skyfighter.muted';          // Mute state
const STORAGE_DIFF = 'skyfighter.difficulty';     // Last selected difficulty
const STORAGE_REDUCED = 'skyfighter.reducedMotion'; // Motion preference
```

## Making Changes

**Game balance adjustments:**
- Enemy spawn rates: Edit `DIFFICULTIES` object (spawnBase, spawnMin)
- Enemy speed/behavior: enemySpeed, enemyShootChance
- Player health/bombs: lives, bombs properties
- Boss difficulty: bossHpMul
- Stage length: KILLS_PER_STAGE

**Visual changes:**
- Add/modify themes in THEMES array
- Update CSS variables in `<style>` block (colors, animations, spacing)
- Modify canvas rendering code in the game loop

**New features:**
- Insert logic within the main game loop function
- Use existing state variables or add new ones
- Remember to update HUD or overlay to reflect new state

## Testing/Validation

Since there's no automated test suite:
- **Manually test in browser** after changes
- Test all three difficulty levels
- Verify touch controls work on mobile devices
- Check that reduced-motion preference disables animations
- Confirm localStorage persists best score and settings across sessions
- Test boss encounters and stage progression

## Performance Considerations

- Canvas rendering is the primary performance cost
- Adaptive quality system (`quality` variable) reduces particle/effect density on slow devices
- Mobile devices benefit from reduced animations (toggle via motion button)
- No network calls; all assets are inline or procedurally generated
