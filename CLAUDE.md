# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/CSS/JavaScript application. The entire game engine, graphics rendering, game logic, UI, and input handling are contained in `index.html` (1669 lines).

## How to Run

Simply open `index.html` in a web browser. The game runs entirely client-side with no build process or dependencies required.

- **Direct file access**: Open `index.html` in your browser
- **HTTP server**: Serve the file via any HTTP server if needed (e.g., `python -m http.server`)

## Code Architecture

The `index.html` file contains three main sections:

### 1. **Markup & Styling** (lines 1-420)
- **DOM Elements**: Canvas for game rendering, HUD elements (score, health, bombs, weapons), overlay menu, touch controls, accessibility elements
- **CSS**: Comprehensive styling for responsive design with viewport scaling, animations (heartbeat, pulsing, screen flash), dark theme, touch-friendly buttons
- **Key CSS classes**:
  - `.stat`: HUD stat displays with backdrop blur
  - `.diffBtn`: Difficulty selection buttons
  - `.iconBtn`: Top-bar buttons (mute, motion, pause)
  - Touch control zones for mobile gameplay

### 2. **Game Engine** (lines 422-1669)
The JavaScript engine is organized by functionality:

#### **Core Systems**
- **Initialization & Setup** (lines 423-471): DOM references, storage keys, utility functions
- **Performance Tracking** (lines 473-483): Adaptive quality system that adjusts graphical detail based on frame rate
- **Game Configuration** (lines 485-515): Difficulty settings, spawn rates, themes (6 visual themes with different color palettes)

#### **Game State Management**
- **Game State Object** (mid-section): Tracks active game session (score, lives, bombs, stage, multiplier, weapon level)
- **Game Loop** (late section): `requestAnimationFrame` update and render cycle with delta time tracking

#### **Entity System**
- **Player**: Single instance, tracks position, velocity, health, weapon, input state
- **Enemies**: Array-based pool, each with position, velocity, health, shoot timer, animation state
- **Bullets**: Separate pools for player and enemy bullets, simple physics (linear movement)
- **Particles/Explosions**: Visual effects for hits, bomb detonations, stage transitions
- **Bosses**: Special enemy variant with larger health pool, name display, health bar UI update

#### **Input Handling**
- **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Movement steering (cursor position), click to fire
- **Touch**: Two-zone system (left 55% = movement joystick, right 45% = action buttons)
- **Touch Joystick**: Dynamic base rendering when dragging in movement zone

#### **Rendering System**
- **Canvas 2D**: All graphics drawn via canvas context (`ctx`)
- **Sprite-like rendering**: Shapes drawn procedurally (triangles for planes, circles for bullets/enemies)
- **HUD Rendering**: Score, best, weapon, hearts, bombs, multiplier updated each frame
- **Visual Effects**: Vignette pulse on damage, white flash on hit/bomb, text pop-ups for combos and stage names
- **Themes**: 6 themed sky gradients applied per-stage

#### **Game Mechanics**
- **Scoring**: Base points + multiplier, combo detection triggers pop-up
- **Weapon System**: 3 weapon types (Vulcan, Laser, Spread) with 5 levels each, found in enemy drop pickups
- **Bomb System**: Limited smart bombs that clear enemies and bullets, refills with pickups
- **Enemy Spawning**: Wave-based spawning with difficulty-adjusted spawn rates
- **Boss Encounters**: One boss per stage, larger health pool, always present when spawned
- **Stages**: Progression tracked by kill count (18 kills per stage), visual theme rotates

#### **Audio**
- **Muting**: Global mute toggle with persistent storage (localStorage key: `skyfighter.muted`)
- **Web Audio API**: Not explicitly implemented, but muting framework is in place for future sound effects

#### **Storage & Persistence**
- **localStorage Keys**:
  - `skyfighter.bestScore`: High score persistence
  - `skyfighter.muted`: Mute state
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessibility preference

#### **Accessibility**
- **Screen Reader Support**: `aria-live` region for announcements, `sr-only` class for hidden labels
- **Reduced Motion**: Respects `prefers-reduced-motion` media query, disables animations in CSS
- **Keyboard Navigation**: Fully keyboard playable
- **Color Contrast**: Text shadows and color choices for visibility

## Key Code Sections for Common Tasks

### Adding a New Weapon Type
1. Locate the weapon definitions in the DIFFICULTIES config and weapon name mappings
2. Add weapon logic in the player update/shoot section
3. Add visual rendering for the new shot pattern in the canvas draw phase

### Adjusting Difficulty Parameters
Edit the `DIFFICULTIES` object (lines 485-489):
- `spawnBase` / `spawnMin`: Controls enemy spawn frequency
- `enemySpeed`: Multiplier for enemy velocity
- `enemyShootChance`: Probability enemies fire per frame
- `lives` / `bombs`: Starting health and bomb count
- `bossHpMul`: Boss health scaling

### Changing Visual Themes
Edit the `THEMES` array (lines 493-500) or add new theme objects with properties:
- `sky`: Array of 3 color stops for background gradient
- `cloud`: RGB values for cloud particles
- `sun`: RGB values for sun/light particles
- `name`: Display name for theme

### Fixing or Adding Game Mechanics
Most game state updates happen in the main update loop within the game loop function. Key phases:
1. Input processing (keyboard/mouse/touch)
2. Player movement and firing
3. Enemy spawning and AI
4. Collision detection (bullets vs entities)
5. Particle and effect updates
6. UI state synchronization

## Important Implementation Details

### Single-File Monolith
All code is in one file to enable direct deployment as a static web asset. Future refactoring into modules is possible but requires bundling.

### Canvas Rendering Performance
- All drawing is immediate-mode (no retained graphics tree)
- Adaptive quality system throttles particle counts and effect complexity at low frame rates
- No WebGL or GPU acceleration currently used

### Touch Control Complexity
Mobile controls use a dual-zone system with dynamic joystick rendering. The joystick base appears only when actively dragging in the movement zone to avoid visual clutter.

### Stage and Boss Progression
Stages are determined by kill count (18 kills per stage). Bosses appear based on internal stage counter. Visual theme rotates automatically through the 6 THEMES array.

### Asset-free Design
No external assets (images, audio, fonts). All visuals are procedural, using canvas shapes and CSS filters. This keeps the file size manageable and the game instantly deployable.

## Common Development Patterns

### Updating HUD Display
HUD elements (`#score`, `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`) are updated via `textContent` or `classList` modifications within the game loop update phase.

### Playing Visual Effects
- White flash on hit: `flashEl.classList.add('hit')`
- Combo popup: Modify `#comboPop` element class and text, animation plays automatically
- Vignette pulse: `vignetteEl.classList.toggle('on')` for damage indication

### Managing Game State Transitions
Overlay menu shows/hides via display style changes. Game state variables control whether the game loop processes updates or waits at the menu.

## Testing During Development

Since there are no automated tests, manual testing focuses on:
- Game loop frame rate stability (check developer console performance tab)
- Input responsiveness across keyboard, mouse, and touch
- Difficulty balance (adjust DIFFICULTIES and test gameplay)
- Visual polish (theme colors, animation timings, particle effects)
- Cross-browser and mobile device compatibility
