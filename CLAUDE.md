# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5/Canvas application. It features dynamic difficulty levels, weapon power-ups, smart bombs, and progressive boss encounters across multiple stages.

- **File**: `index.html` (standalone, ~58KB)
- **Type**: Browser-based game
- **Technology**: HTML5 Canvas, Vanilla JavaScript (ES6)
- **Target**: Desktop (mouse/keyboard) and mobile (touch controls)

---

## Game Mechanics

### Core Systems

#### Player Ship
- **Starting HP**: Varies by difficulty (3-4 hearts)
- **Weapon System**: Multiple weapon types (Vulcan, Spread, Pulse, Laser, Shrapnel)
- **Weapon Levels**: 1-5 per weapon, increased by pickups
- **Movement**: Full-screen, constrained to canvas bounds
- **Special Ability**: Smart bombs (limited stock: 2-3 per difficulty)

#### Enemy System
- **Spawning**: Time-based with difficulty-dependent intervals
- **Enemy Types**: Generic fighter enemies (not currently differentiated by sprite)
- **Firing Pattern**: Random chance per enemy per frame (difficulty-dependent)
- **Boss Encounters**: Appear every 18 kills per stage; boss HP scales with difficulty

#### Scoring & Progression
- **Kill Points**: +100 per regular enemy
- **Multiplier**: Increases with consecutive kills (without taking damage)
- **Multiplier Reset**: Resets on player damage
- **Stages**: Progress after 18 enemy kills; boss stage appears as stage final
- **Best Score**: Persisted to localStorage

### Difficulty Modes

| Aspect | Easy | Normal | Hard |
|--------|------|--------|------|
| Enemy Spawn Interval | 75→40 frames | 58→26 frames | 40→18 frames |
| Enemy Speed | 1.0× | 1.4× | 1.9× |
| Shoot Chance | 45% | 70% | 95% |
| Player Lives | 4 | 3 | 3 |
| Starting Bombs | 3 | 2 | 2 |
| Boss HP Multiplier | 0.8× | 1.0× | 1.3× |

---

## Code Structure

### HTML Layout (Lines 362–419)
- **Canvas**: Fullscreen 2D drawing surface
- **HUD Elements**: Score, best, multiplier, weapon status, health/bombs
- **Overlay**: Title card, difficulty selection, end-game stats
- **Touch Controls**: Mobile joystick zone (left) and action buttons (right)
- **Top Bar**: Mute, reduced-motion, and pause buttons

### CSS Styling (Lines 10–358)
- **Color Scheme**: Dark blue background (#05122b), accent golds/greens
- **Animations**: Keyframes for combo pulses, stage transitions, vignette effects
- **Responsive**: Uses `clamp()` for fluid scaling on mobile/desktop
- **Accessibility**: Dark mode forced, reduced-motion respected via media query
- **Visual Effects**: 
  - Vignette (red pulsing on damage)
  - Screen flash (white on hit, bomb)
  - Combo popup (gold text)
  - Stage banners with glow

### JavaScript Game Loop (Lines 422+)
**Core State:**
```javascript
const STORAGE_BEST = 'skyfighter.bestScore';
const STORAGE_MUTE = 'skyfighter.muted';
const STORAGE_DIFF = 'skyfighter.difficulty';
const STORAGE_REDUCED = 'skyfighter.reducedMotion';

const DIFFICULTIES = { easy, normal, hard };
const THEMES = [6 color themes]; // Daylight, Sunset, Night, Storm, Arctic, Volcanic
const KILLS_PER_STAGE = 18;
```

**Game Entities:**
- **Player**: Position, velocity, HP, weapon state, bomb count
- **Enemies**: Array of active enemies with position, velocity, HP, firing state
- **Projectiles**: Player bullets and enemy bullets (separate arrays)
- **Explosions**: Particle effects for kills and impacts
- **Vignette/Flash**: Transient damage feedback

**Main Loop Flow:**
1. Input polling (keyboard, mouse, touch)
2. Entity updates (movement, collision, firing)
3. Particle/animation updates
4. Collision detection (player↔enemy, bullet↔entity)
5. Canvas render (background gradient, all entities, HUD)
6. Performance tracking (adaptive quality)

**Key Functions (inferred from structure):**
- `announce(text)`: ARIA live region updates for accessibility
- `trackPerf(dt)`: Monitors frame time; reduces particle quality if FPS drops below 40
- Enemy spawning logic with dynamic interval reduction
- Weapon class definitions (different bullet patterns, speeds, damage)
- Boss AI (larger enemy with phase HP and attack patterns)
- Touch joystick (follows finger, triggers movement)

---

## Weapon System

Six weapon types, each with 5 levels:
1. **Vulcan**: Narrow forward spread, high fire rate
2. **Spread**: Wide fan of bullets, lower damage per bullet
3. **Pulse**: Burst fire, gaps between bursts
4. **Laser**: Single powerful beam (rarer?)
5. **Shrapnel**: Shatter rounds that create child projectiles on impact
6. (Possible 6th weapon)

Weapons are unlocked by collecting enemy pickups on kill. Collecting same weapon again levels it up (1→5).

---

## Bomb System

**Mechanics:**
- Limited count per game (2-3 per difficulty)
- Activated via Space / Shift / Mobile BOMB button
- Clears all visible enemy projectiles
- Deals area damage to enemies
- Visual: Full-screen flash, red vignette pulse

**Recharging:**
- No recharge mid-game; count is fixed per difficulty
- Bombs display as emoji row in HUD (🔶/opaque vs. dimmed)

---

## Performance & Adaptivity

**Quality Levels:**
- **0 (Full)**: All particles, shadows, effects
- **1 (Medium)**: Reduced particle count
- **2 (Low)**: Minimal particles, simplified rendering

**Trigger:**
- If average FPS over 45 frames drops below 25 (dt > 1/25), increment quality
- If average FPS exceeds ~56 (dt < 1/56), decrement quality
- Prevents degradation during intense scenes on low-end devices

---

## Input & Controls

### Desktop
- **Mouse**: Move ship to follow cursor; click to fire
- **Keyboard**: 
  - Arrows or WASD to move
  - Space to fire (or mouse click)
  - X or Shift to bomb
  - P or Esc to pause

### Mobile
- **Left Zone** (55% left): Drag finger to move ship (joystick base + stick visual feedback)
- **Right Zone** (45% right): 
  - **FIRE** button (red): Shoot
  - **BOMB** button (gold): Activate bomb

**Touch Feedback:**
- Visual joystick base and stick circle appear under finger
- Button press changes background opacity

---

## Persistence & Settings

**localStorage Keys:**
- `skyfighter.bestScore`: Best ever score (number)
- `skyfighter.muted`: Audio muted flag (boolean)
- `skyfighter.difficulty`: Last selected difficulty (easy|normal|hard)
- `skyfighter.reducedMotion`: User motion preference (boolean)

**Behaviors:**
- Best score displays in HUD and end-game overlay; "🏆 New Best!" message if beaten
- Mute button persists across sessions
- Reduced motion disables all CSS animations (respects `prefers-reduced-motion`)

---

## Themes & Visual Design

Six dynamic sky themes (selected per stage):
1. **Daylight Front**: Classic blue sky (#0d2f5e → #7fc1ff)
2. **Sunset Corridor**: Orange/purple gradient
3. **Night Approach**: Deep blues and dark purples
4. **Storm Front**: Green-tinted clouds
5. **Arctic Wastes**: Pale cyan skies
6. **Volcanic Ridge**: Fire reds and oranges

Each theme defines:
- Sky gradient colors (3-stop)
- Cloud color (RGB string for fog/billowing effects)
- Sun/glow color

---

## Accessibility

**Features:**
- ARIA live region (`#srLive`) for dynamic game events
- Screen-reader only class `.sr-only` for hidden but announced text
- High contrast color scheme (light text, dark background)
- Reduced motion support (CSS animations conditional on `prefers-reduced-motion` media query)
- Mute button for audio (no sound implemented yet, but framework exists)
- Mobile-friendly input (large touch targets)

**Missing:**
- Alt text for icon buttons (could improve)
- No keyboard-accessible pause menu beyond P key
- No font size scaling beyond viewport-based `clamp()`

---

## Known Limitations & Future Considerations

1. **Single File**: All code in one HTML file; could refactor to separate CSS/JS modules
2. **No Audio**: Sound framework exists but no audio implementation
3. **No Asset Pipeline**: Graphics are procedural (canvas drawing); no sprite sheets
4. **Mobile Optimizations**: Touch controls work but could be refined (haptic feedback, sensitivity tuning)
5. **Boss AI**: Basic patterns; could expand with multi-phase bosses, special attacks
6. **Power-up Variety**: Only weapon pickups; could add shields, temporary invulnerability, score multipliers
7. **Particle System**: Adaptive quality is basic; could optimize further with spatial partitioning
8. **Storage**: No cloud save; only browser localStorage (single device)

---

## Development Notes

### Adding Features

**New Weapon:**
1. Define bullet pattern in weapon class section
2. Add to weapon cycle order (likely in pickup/level-up logic)
3. Test firing rate, damage, and visual clarity

**New Boss:**
1. Define HP, movement pattern, and attack behavior
2. Stage boss encounter trigger (already at kill 18 per stage)
3. Add name/title to boss bar display

**New Theme:**
1. Add to `THEMES` array with sky gradient stops, cloud/sun colors, and name
2. Theme is selected per stage (index % THEMES.length)

### Debugging
- Check browser console for errors (no error handling visible in code; add try/catch as needed)
- Monitor FPS with DevTools Performance tab
- Inspect localStorage for persistence data
- Test mobile with Chrome DevTools device emulation or real phone

### Performance Profiling
- Use Chrome DevTools Performance recorder during a game
- Check for excessive repaints (canvas redraws every frame)
- Monitor input event frequency (could debounce if needed)
- Profile particle/collision loops if FPS drops below 30

---

## File Manifest

| File | Purpose | Size |
|------|---------|------|
| `index.html` | Complete game application | ~58KB |
| `README.md` | Project description | <1KB |
| `CLAUDE.md` | This documentation | — |

---

## Deployment

Currently a static HTML file; can be served from any web server or opened locally (`file:///...`).

**Recommended:**
- Deploy to GitHub Pages, Vercel, Netlify, or similar
- Use HTTP/HTTPS (required for localStorage and touch events on some browsers)
- Test on multiple devices (desktop, tablet, phone; various screen sizes)

---

**Last Updated:** 2026-10-07  
**Version:** 1.0 (Initial game release)
