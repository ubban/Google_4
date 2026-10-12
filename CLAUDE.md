# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. The entire game is contained in a single `index.html` file (~1,669 lines) combining CSS, HTML, and JavaScript.

**Core gameplay**: Navigate a fighter jet, destroy enemy waves, collect weapon power-ups, trigger smart bombs, and survive procedurally escalating boss encounters across multiple stages.

## Architecture

### Single-File Design

The project uses a monolithic architecture with all game code embedded in `index.html`. This makes it trivial to deploy (just serve the file) but requires disciplined code organization within the file.

**Sections** (in order of appearance in index.html):
1. **DOCTYPE & Head metadata** (lines 1–9): Title, viewport, favicon
2. **CSS Styles** (lines 10–358): Responsive layout, animations, theming, HUD, overlays, touch controls, accessibility
3. **HTML Structure** (lines 361–420): Game canvas, UI elements, HUD, overlay menu, touch controls
4. **JavaScript** (lines 422+): Complete game engine in IIFE

### JavaScript Game Engine (IIFE Pattern)

All JavaScript runs in a single IIFE (`(function () { 'use strict'; ... })()`), preventing global scope pollution.

**Core Systems**:

#### 1. **Game State & Configuration**
- `DIFFICULTIES`: Three difficulty profiles (easy/normal/hard) controlling spawn rates, enemy speed, shooting chance, lives, bombs, boss HP
- `THEMES`: Six visual color schemes for dynamic stage backgrounds
- `KILLS_PER_STAGE`: Progression mechanic (18 kills triggers stage/boss transition)
- `quality`: Adaptive performance setting (0=full, 1=medium, 2=low) adjusted via frame-time tracking

#### 2. **Core Loop & Timing**
- Main loop is `requestAnimationFrame`-based
- Frame time tracking for adaptive performance scaling
- Game state: playing, paused, game-over

#### 3. **Player Entity**
- Position, velocity, health (lives)
- Weapon state (current weapon, level, ammo type)
- Bomb inventory and bomb activation logic
- Controls: keyboard (arrow keys, WASD, Space, X, Shift, P/Esc) and mouse (move-to-aim, click-to-fire)
- Mobile/touch support: drag-to-move joystick, FIRE/BOMB buttons

#### 4. **Enemy System**
- Enemy spawning with difficulty-based wave timings
- Enemy types and behaviors defined in `DIFFICULTIES`
- Enemy shooting with variable chance and projectile patterns
- Enemy-player collision detection for damage
- Enemy death triggers weapon drop probability

#### 5. **Weapon System**
- Multiple weapon types (base weapon appears to be VULCAN)
- Level progression (displayed in HUD as weapon level)
- Ammunition/fire rate varies by weapon type
- Weapon pickup logic for enemy drops

#### 6. **Boss System**
- Boss encounters triggered after `KILLS_PER_STAGE` enemy kills
- Boss health bar (visual element #bossBarWrap, #bossBarFill)
- Boss HP scaled by difficulty multiplier (`bossHpMul`)
- Boss defeat triggers stage progression

#### 7. **Scoring & Progression**
- Score accumulation from kills
- Combo/multiplier tracking (displayed as "x" in HUD)
- Best score persistence via localStorage
- Multi-stage progression (currently supports undefined number of stages)

#### 8. **Visual & Audio**
- Canvas 2D rendering for all game entities
- CSS animations for HUD, combo popups, stage banners
- Vignette effect (red pulse on damage)
- Flash effect on hit/bomb detonation
- Mute toggle (persisted to localStorage)
- Audio playback state managed separately (muted by default or per localStorage)

#### 9. **Accessibility & Responsiveness**
- Screen reader support via `#srLive` (aria-live region)
- Reduced motion support (disables animations if `prefers-reduced-motion`)
- Responsive design via CSS `clamp()` for scalable text/sizing
- Touch-safe controls with large interactive areas
- Safe area insets for notched devices (`env(safe-area-inset-*)`)

#### 10. **Persistence (localStorage)**
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User motion preference

### UI/UX Structure

**HTML elements** (all children of #container):
- `#gameCanvas`: Main 2D rendering surface
- `#hud`: Top-center stats display (score, best, multiplier, weapon, hearts, bombs)
- `#bossBarWrap`: Boss health indicator (hidden until boss appears)
- `#overlay`: Game menu/pause/game-over modal
- `#topBar`: Settings buttons (mute, motion, pause)
- `#touchControls`: Mobile joystick + FIRE/BOMB buttons
- `#vignette`, `#flash`, `#comboPop`, `#stageBanner`: Visual effect layers

**Responsive breakpoints**: Uses viewport units (vw, vh, clamp) for scalability across mobile to desktop.

## Development Commands

Since this is a single HTML file, there are no build steps, linters, or test runners configured.

**Run the game**:
```bash
# Option 1: Use Python's built-in server (Python 3)
python3 -m http.server 8000
# Then open http://localhost:8000/index.html in browser

# Option 2: Use Node.js http-server
npx http-server
```

**Making changes**:
- Edit `index.html` directly
- Refresh the browser to test (hard refresh with Ctrl+Shift+R / Cmd+Shift+R to bypass cache)
- Changes are isolated; no build or dependency installation needed

## Common Customization Points

### Difficulty Balance
Modify `DIFFICULTIES` object (line ~485) to adjust:
- Enemy spawn frequency (`spawnBase`, `spawnMin`)
- Enemy movement speed (`enemySpeed`)
- Enemy shooting aggression (`enemyShootChance`)
- Player lives and bombs
- Boss health scaling (`bossHpMul`)

### Visual Themes
Add new theme to `THEMES` array (line ~493) with custom `sky`, `cloud`, `sun` color arrays and a `name`.

### Weapon & Enemy Tweaks
Search for:
- `const VULCAN` or weapon definitions: adjust fire rate, damage, spread
- Enemy spawn logic: modify frequency curves or behavior patterns
- Boss behavior: health, attack patterns, drop rates

### Controls
Mobile/desktop control logic is in event listeners (touch, keyboard, mouse). Key elements:
- `#moveZone`, `#actionZone`, `#fireBtn`, `#bombBtn` for touch
- `keydown`/`keyup` listeners for keyboard
- `mousemove`, `click` for mouse

### Accessibility
- Reduced motion: CSS respects `prefers-reduced-motion` media query; add more rules around line 349
- Screen reader announcements: Use `announce()` function to update `#srLive`
- Color contrast: Update HUD colors in CSS if accessibility audit required

## Testing the Game

**Manual testing flow**:
1. Start game, verify menu overlay shows with difficulty buttons and start button
2. Select difficulty, start game
3. Keyboard/mouse: move with arrow keys, shoot with space, bomb with X
4. Mobile: test joystick drag in left zone, tap FIRE/BOMB buttons
5. Verify HUD updates (score, lives, bomb count, weapon level)
6. Trigger 18 kills to reach boss stage
7. Defeat boss to progress to next stage (verify theme changes, stage banner shows)
8. Test pause (P or pause button)
9. Test mute toggle
10. Test reduced motion toggle (animations should stop)
11. Verify best score persists after game over and page reload

**Performance debugging**:
- Open DevTools (F12), go to Performance tab
- Record a game session, look for frame drops below 60fps
- The game auto-adjusts quality if sustained frame time > 1/40s (quality increases, reducing visual detail)
- Check Console for any JS errors

## Key Design Decisions

1. **Single-file deployment**: No build step, no external dependencies. Trade-off: harder to refactor, but trivial to host.
2. **Canvas 2D rendering**: Fast enough for this game scope; simpler than WebGL or Three.js.
3. **Difficulty presets over infinite scaling**: Fixed difficulty tiers are easier to balance than algorithmic scaling.
4. **localStorage persistence**: Lightweight, no server needed. Limited to ~10MB per origin.
5. **CSS animations for HUD effects**: Offloads animation to browser rendering, not game loop, reducing CPU cost.
6. **Mobile-first responsive design**: Uses `clamp()` instead of media queries for smoother scaling across all viewport sizes.

## Future Enhancement Ideas

- **Score upload/leaderboard**: Add POST endpoint to submit scores to a backend
- **Multi-stage bosses**: Extend boss system with unique attack patterns per boss
- **Sound effects & music**: Add Web Audio API or HTML audio elements for SFX and BGM
- **Level editor**: Create a UI to design custom enemy waves or boss patterns
- **Replay system**: Record input, replay to allow sharing clips
- **Multiplayer**: WebSocket or WebRTC for co-op or PvP (significant architecture refactor)
