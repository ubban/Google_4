# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based, Raiden-style vertical shoot-'em-up game. Players control an aircraft, manage multiple weapons, deploy smart bombs, and progress through stage-based boss encounters. The game runs entirely in the browser using HTML5 Canvas and vanilla JavaScript.

**Key Features:**
- Vertical scrolling gameplay with dynamic difficulty
- Multiple weapon systems (Standard, Spread, Rapid, Homing)
- Smart bomb mechanic for clearing enemies
- Health and bomb inventory systems
- Score tracking with multiplier bonuses
- Responsive design supporting desktop and mobile play

---

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project metadata
└── CLAUDE.md          # This file
```

The entire game is contained in a single `index.html` file (58KB). All HTML structure, styling (CSS), and game logic (JavaScript) are inline within this single document.

---

## Game Architecture

### Core Game Systems

#### 1. **Canvas & Rendering**
- Main game loop runs at ~60 FPS via `requestAnimationFrame`
- Canvas ID: `#gameCanvas` (configurable viewport)
- Background: Gradient from `#0d2f5e` → `#1e6fd8` → `#7fc1ff`
- Rendering order: Background → Game objects → UI overlay

#### 2. **Player Aircraft**
- Single player-controlled entity
- Position: Center of canvas, controlled by mouse/touch
- Health: Variable (typically 3 HP, displayed as hearts ❤️)
- Bombs: Limited inventory (typically 3), displayed as bomb icons 💣
- Weapon system: Cycles through 4 weapon types

#### 3. **Weapon System**
Available weapons (cycle with spacebar):
- **Standard**: Single-center bullet stream
- **Spread**: Wide-angle fan pattern
- **Rapid**: High fire-rate, low damage
- **Homing**: Bullets track enemy movement

Weapon switch affects visual appearance and multiplier penalty.

#### 4. **Enemies & Bosses**
- Auto-spawning waves during stages
- Boss entities at stage end (stage-specific behavior)
- Collision detection with bullets
- Health bar display for bosses

#### 5. **Game State Machine**
- **Playing**: Active gameplay
- **GameOver**: Player defeated (display score, offer restart)
- **Victory**: Stage/game completed
- States managed via game loop logic

#### 6. **UI/HUD System**
```html
#hud (top-center)
├── Hearts (#hearts) — Health display
├── Bombs (#bombs) — Bomb inventory
├── Weapon Stat (#weaponStat) — Current weapon name
├── Multiplier Stat (#multStat) — Score multiplier
└── Score display
```

Visual effects:
- Vignette (#vignette): Red border pulse on hit
- Flash (#flash): White screen flash on hit or bomb

---

## Key Game Mechanics

### Input Handling
- **Mouse/Touch**: Move player to cursor position
- **Spacebar**: Cycle to next weapon
- **Left Click / Touch**: Fire primary weapon
- **Right Click / Shift + Click**: Deploy smart bomb (if inventory > 0)

### Collision System
- **Bullet-Enemy**: Removes bullet, damages/removes enemy
- **Enemy-Player**: Deals damage, knockback
- **Bomb Explosion**: Clears nearby enemies in radius

### Score System
- Base points per enemy: `10 × (weapon_dmg / weapon_base_dmg)`
- Multiplier increases per consecutive enemies without weapon switch
- Multiplier resets on weapon change (balance mechanic)

### Difficulty Progression
- Waves increase in size/speed per stage
- Boss HP scales with progression
- Enemy spawn rate accelerates

---

## Development Guidelines

### Code Style & Conventions
1. **Vanilla JavaScript** — No frameworks or dependencies
2. **Canvas API** — Direct drawing via `ctx.fillRect()`, `ctx.strokeRect()`, etc.
3. **Inline Logic** — All code lives in `<script>` tag in HTML
4. **Comments** — Add only for non-obvious game logic or tricky collision math
5. **Naming** — Use camelCase for variables/functions, UPPER_CASE for constants

### Adding New Features

**New Weapon Type:**
1. Define weapon object in weapon array with `name`, `dmg`, `fireRate`, `pattern`
2. Add rendering logic in bullet-draw section
3. Update HUD color/name display
4. Test multiplier balance

**New Enemy Type:**
1. Add to enemy spawn logic with `type`, `hp`, `speed`, `sprite`
2. Implement collision-specific behavior
3. Assign loot/score value
4. Test spawn density and difficulty curve

**Boss Mechanics:**
1. Define boss object with `name`, `maxHp`, `stage`, `attacks[]`
2. Implement attack pattern in game loop (move/fire based on health threshold)
3. Visual differentiation (larger sprite, health bar)
4. Victory condition check

### Testing Before Commit

1. **Gameplay Tests:**
   - [ ] Player movement smooth and responsive
   - [ ] All 4 weapons fire and cycle correctly
   - [ ] Bombs deploy and clear enemies
   - [ ] Collisions feel fair (no phantom hits)
   - [ ] Score accumulates and multiplier updates
   - [ ] Game-over and victory states trigger correctly

2. **Responsiveness:**
   - [ ] Desktop mouse controls work
   - [ ] Mobile touch controls responsive
   - [ ] Canvas scales to viewport
   - [ ] HUD readable on small screens

3. **Visual Polish:**
   - [ ] No flickering or rendering gaps
   - [ ] Flash/vignette effects play smoothly
   - [ ] Text legible (check contrast, font sizes)
   - [ ] Colors match dark theme

4. **Performance:**
   - [ ] Frame rate stable at 60 FPS
   - [ ] No lag spikes during heavy enemy spawn
   - [ ] No memory leaks (check DevTools Memory profiler)

### Common Tasks

**Adjust Game Balance:**
- Weapon damage: Modify `dmg` in weapon object
- Enemy health: Adjust enemy spawn `hp`
- Difficulty curve: Tweak spawn rates and wave sizes per stage
- Bomb cooldown: Modify bomb regeneration timer

**Tweak Visuals:**
- Player color: Change `fillStyle` in player-draw code
- Background gradient: Edit canvas background CSS
- HUD styling: Modify `.stat` and `.label` CSS
- Explosion particles: Adjust `createExplosion()` values

**Bug Fixes:**
- Collision issues: Check distance-calculation math in collision functions
- Rendering glitches: Ensure draw order is correct (background → objects → UI)
- Input delays: Verify event listeners are attached to correct elements

---

## File Organization Within index.html

The single `index.html` file is organized in sections:

1. **DOCTYPE & Meta Tags** — HTML5 document setup, viewport, theme color
2. **Head Styles** — All CSS (dark theme, canvas styling, HUD layout, animations)
3. **Body Structure** — Minimal HTML: `#container` → `#gameCanvas` + HUD elements
4. **Script Tag** — All game logic in vanilla JavaScript

To maintain readability despite the file size:
- Use clear section comments in the `<script>` tag (`// ===== SECTION NAME =====`)
- Group related functions together (input handling, rendering, game state, etc.)
- Keep functions focused and well-named

---

## Performance Considerations

- **Canvas Size**: Dynamically set to match viewport; larger canvas = more redraw cost
- **Enemy Count**: Typical max ~50 enemies on screen; monitor FPS if adding more
- **Bullet Pooling**: Consider object pooling for bullets if performance degrades
- **Animation Frames**: Keep animations (pulsing vignette, heart beat) lightweight

---

## Browser Compatibility

- **Supported**: Chrome, Firefox, Safari (modern versions)
- **Required APIs**: Canvas 2D, Touch Events, Pointer Events
- **Not Supported**: IE11 and below (uses modern ES6+ syntax)

---

## Deployment & Distribution

The game is **self-contained** in a single HTML file:
- No build process required
- No external dependencies
- Simply serve `index.html` to play
- Can be embedded in any web page via `<iframe>`

---

## Future Enhancement Ideas

- Persistent high-score leaderboard (localStorage or cloud API)
- Sound effects and background music (audio system)
- Additional weapon types and visual effects
- Difficulty settings (Easy/Normal/Hard)
- Pause menu and settings panel
- Power-up items (health, ammo, temporary boost)
- Unlockable aircraft skins

---

## Questions & Support

For questions about the codebase or architecture, refer to this file. For feature requests or bugs, document them with steps to reproduce and expected vs. actual behavior.

---

**Last Updated:** 2026-10-10  
**Game Title:** Sky Fighter  
**Repository:** ubban/Google_4  
**Branch:** claude/inspiring-planck-weoqkz
