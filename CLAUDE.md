# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file web application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (1669 lines). There are no build steps, external dependencies, or frameworks—it's pure vanilla JavaScript using only the Canvas API and Web Audio API.

## Development & Testing

### Running the Game

Open `index.html` directly in a web browser. No server or build step is required.

- **Desktop**: Full mouse/keyboard support
  - **Mouse**: Move to steer, click to fire
  - **Keyboard**: WASD/Arrows to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mobile**: Full touch support with on-screen joystick and buttons

### Testing Changes

Since there's no test suite, test changes by:
1. Opening the file in a browser
2. Playing through a full round on each difficulty (Easy, Normal, Hard)
3. Verifying:
   - Gameplay mechanics work (movement, shooting, collecting pickups)
   - Enemies spawn and behave correctly
   - Bosses appear and are defeatable
   - Audio plays (unless muted)
   - Touch controls work on mobile
   - Reduced motion toggle disables animations
   - Score persistence works (refresh the page)

### Code Style

- No external linters or formatters are configured
- The code uses `'use strict'` mode
- Spacing: 2 spaces for indentation in the game loop sections, consistent with existing style
- Avoid adding global variables; use the existing IIFE scope

## Architecture & Key Systems

The entire game is organized within a single IIFE (Immediately Invoked Function Expression) to maintain encapsulation. Key systems are organized as follows:

### 1. **Canvas & Rendering**
- **Location**: Lines ~426, ~600+ (main game loop and rendering)
- **Canvas setup**: 800×600 native; responsive scaling via CSS `max-width/max-height`
- **Rendering approach**: Immediate-mode drawing (no display list), redraw every frame
- **Background**: Dynamic gradient based on current theme; sky color animates with stage progression
- **Important**: The game uses adaptive quality settings (see `trackPerf()`) to throttle particle/effect rendering on lower-end devices

### 2. **Game State & Logic**
- **Difficulty system** (lines ~485–489):
  - Three tiers: Easy, Normal, Hard
  - Affects: spawn rates, enemy speed, shoot frequency, lives, bombs, boss HP
  - Persisted to localStorage
- **Stage progression**: 18 kills per stage; stages 1–6 cycle through themes; cycling restarts themes
- **Score & multiplier**: Multiplier increases with consecutive hits, resets on player damage
- **Lives & bombs**: Lost on player collision; bombs (smart explosions) are consumable

### 3. **Entities (Player, Enemies, Bosses, Projectiles)**
- **Player** (entity type: `ENTITY_PLAYER`):
  - Position-based collision with 14px radius
  - Weapons: 5 types (VULCAN, SPREAD, LASER, GUIDED, ION) with level-up system
  - Can hold max 2 bombs; dropped on hit
- **Enemies** (defined in `ENEMY_TYPES`):
  - Three base types: `grunt`, `interceptor`, `tank`
  - Each type has color, HP, speed multiplier, AI behavior (weave, etc.)
  - Spawn in waves; frequency increases per stage and difficulty
- **Bosses** (defined in `BOSSES` array):
  - Six unique bosses with distinct color schemes and health multipliers
  - One random boss per stage; pattern repeats after 6 stages
  - Boss shoots projectiles in patterns; defeated when HP reaches 0
- **Projectiles**: Player shots and enemy shots handled in separate arrays; collision checked each frame

### 4. **Weapon System**
- **Five weapons** with distinct firing patterns:
  - **VULCAN** (single-fire): rapid, straight shots
  - **SPREAD** (cone-fire): 3–5 bullets in a fan pattern
  - **LASER** (piercing): single beam; passes through enemies
  - **GUIDED** (homing): tracks the nearest enemy
  - **ION** (area): wide AoE burst
- **Leveling**: Weapon pickups (icon: ⬆) increase current weapon level (max 5)
- **Pickups** (lines ~1000+): Power-up entities that drift down; collision grants benefits

### 5. **Audio System (WebAudio Synthesis)**
- **No external audio files**: All sounds are generated procedurally using WebAudio API
- **Synth-based**: Uses oscillators and envelope modulation for effects
- **Sounds**:
  - Gun fire, explosions, hits, boss appearance, stage banners, combo notifications
  - Mute toggle: persisted to localStorage
- **Important**: Audio context must be initialized on user gesture (done on startBtn click)

### 6. **Input Handling**
- **Keyboard**: Global key-down/key-up listeners
- **Mouse**: Move updates player X; click fires; position used for guided weapon targeting
- **Touch**: Gesture-based on-screen joystick (left zone) and button controls (right zone)
- **Pause**: P or Esc pauses; overlay mode prevents game updates

### 7. **UI & Overlay System**
- **Overlay**: Semi-transparent modal covering the canvas (lines ~200–260)
  - Title card shows game intro, difficulty buttons, start button
  - Score card shows after game-over with stats and replay prompt
  - Transitions: card animates in/out
- **HUD (Heads-Up Display)**: Always-visible stats
  - Score, best, multiplier, weapon name/level, hearts (lives), bombs
  - Positioned absolutely; responsive sizing with `clamp()`
- **Boss bar**: Shows boss name and health; appears when boss spawns
- **Accessibility**: Screen reader live region (`#srLive`) for announce updates; reduced motion toggle

### 8. **Performance & Optimization**
- **Adaptive quality** (lines ~473–483):
  - Tracks frame times; reduces quality (lowers particle/effect count) if average FPS < 40
  - Increases quality back if FPS > 56 for smooth handling
- **Entity pooling**: Not explicitly used; entities are created/destroyed dynamically
- **Culling**: Off-screen entities are removed to prevent memory bloat
- **Canvas rendering**: Uses `.clearRect()` each frame and redraws all entities (no layer optimization)

### 9. **State Management & Persistence**
- **localStorage keys** (lines ~463–466):
  - `skyfighter.bestScore`: Best score achieved
  - `skyfighter.muted`: Audio mute preference
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessibility motion setting
- **No save/load mid-game**: Game state is ephemeral; only stats and preferences persist

## Important Implementation Notes

### Entity Types & IDs
Entities are categorized by numeric type IDs:
- `ENTITY_PLAYER` = 0
- `ENTITY_PLAYER_SHOT` = 1
- `ENTITY_ENEMY` = 2
- `ENTITY_ENEMY_SHOT` = 3
- `ENTITY_PICKUP` = 4
- `ENTITY_BOSS` = 5
- `ENTITY_BOMB_WAVE` = 6 (boss/enemy bombs)

Collision detection iterates entity arrays and checks proximity (distance-based).

### Theming & Visual Progression
The `THEMES` array (6 themes) determines sky gradients and colors. Themes cycle as the player progresses through stages. This creates a sense of progression and variety without loading external assets.

### Common Patterns for Modification

**Adding a new weapon**:
1. Add entry to `WEAPONS` array with name, icon, and firing logic
2. Update weapon-switching logic to include it
3. Add rendering code in the weapon-drawing section (lines ~1200+)

**Adding a new enemy type**:
1. Add entry to `ENEMY_TYPES` with stats (w, h, hp, color, speed, score, minStage)
2. Update spawn logic to include it when minStage condition is met
3. Add rendering code if a unique visual is needed

**Adding a new boss**:
1. Add entry to `BOSSES` array with name and color scheme
2. Boss list cycles; new boss automatically appears after existing bosses are rotated

**Modifying difficulty**:
1. Adjust `DIFFICULTIES` object values (spawn rates, speeds, lives, bombs, boss HP)
2. Changes apply on next difficulty selection and game start

## File Structure

```
/
├── index.html          [1669 lines] Main game file (HTML, CSS, JavaScript)
├── README.md           [Minimal] Project metadata
├── .git/               Git history
└── CLAUDE.md           This file
```

## Performance Considerations

- **Single-canvas rendering**: All game elements drawn to one 800×600 canvas
- **Frame budget**: Target 60 FPS; adaptive quality reduces visual fidelity at ~40 FPS to maintain playability
- **No external assets**: Game is entirely self-contained; no HTTP requests after initial load
- **Mobile optimization**: Canvas is rendered at device pixel ratio; touch input is hardware-accelerated

## Next Steps for Development

When adding features:
1. Identify which system is affected (input, rendering, entities, audio, UI)
2. Locate the relevant section in the code (comments mark major sections)
3. Make the smallest change needed; avoid refactoring unrelated code
4. Test in browser on desktop and mobile (if touch is involved)
5. Commit with a clear message describing the change and why
