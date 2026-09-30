# Claude Code Guidance for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The game features:
- Player-controlled fighter aircraft
- Multiple weapon types with switching mechanics
- Smart bomb power-ups
- Stage-based boss encounters
- Responsive design for desktop and mobile

## Project Structure

- **index.html** — The main game file containing all HTML, CSS, and JavaScript. This is a single-file web application.
- **README.md** — Project metadata

## Key Technical Details

### Game Engine
- **Canvas API** — 2D rendering engine using HTML5 Canvas
- **Vanilla JavaScript** — No frameworks or build tools required
- **Responsive Design** — Scales to different screen sizes with `clamp()` for fluid typography
- **Mobile Support** — Touch controls with `touch-action: none`

### Game Mechanics
- **Player Aircraft** — Sprite-based movement with weapon system
- **Weapon System** — Multiple weapon types with switching capability
- **Power-ups** — Smart bombs for area damage
- **Boss Encounters** — Stage-based boss battles
- **HUD** — Real-time stats display (health, score, etc.)

### Visual Design
- **Color Scheme** — Dark blue theme (#05122b, #0b1d3a) with gradient sky effects
- **Typography** — Responsive font sizing using `clamp()`
- **Styling** — CSS Grid/Flexbox for layout, CSS variables for theming
- **Accessibility** — Safe-area insets for notched devices

## Development Guidelines

### When Working on This Project

1. **Single File Consideration** — All game code lives in index.html. When making changes:
   - Keep HTML structure minimal (mainly the canvas and HUD container)
   - Separate concerns in the `<style>` and `<script>` sections logically
   - Use clear comments for major game systems (physics, rendering, input, etc.)

2. **Testing the Game** — Run the game locally by opening index.html in a browser. Use:
   - Browser DevTools Console for JavaScript errors
   - Chrome DevTools Performance tab for FPS monitoring
   - Mobile device testing via localhost (if serving on network)

3. **Common Changes**
   - **Weapon balancing** — Look for weapon damage/rate values in the game code
   - **Visual tweaks** — Modify CSS variables or canvas rendering code
   - **Game mechanics** — Physics calculations and entity update loops
   - **UI updates** — HUD HTML structure and CSS styling

### Claude Code Workflow

- **Read the full file** when starting work on game logic — game state is interconnected
- **Use Bash to serve** the project: `python3 -m http.server 8000` (then open http://localhost:8000)
- **Avoid splitting** game code into multiple files unless specifically requested
- **Test thoroughly** after changes — game behavior can have subtle dependencies

## Common Tasks

### Run the Game
```bash
# Start a simple HTTP server
python3 -m http.server 8000
# Open http://localhost:8000 in a browser
```

### Add a New Feature
1. Identify the game system it belongs to (rendering, physics, input, etc.)
2. Find relevant code sections in the script block
3. Add the feature following existing patterns
4. Test in the browser DevTools

### Debug Issues
1. Open Browser DevTools (F12)
2. Check the Console for errors
3. Use breakpoints on relevant game functions
4. Monitor canvas performance in Performance tab

## Asset and Resource Notes

- All assets are either inline SVG (favicon) or rendered procedurally via Canvas
- No external image or sound files are required
- Styling uses system fonts only (Segoe UI, Roboto, Arial fallback)

## Dependencies

None — this is a standalone HTML/CSS/JavaScript game with no build step or npm dependencies.

## Version Control

- Work on feature branches (e.g., `claude/inspiring-planck-g26jcf`)
- Commit logical changes with clear messages describing the game system affected
- Test before pushing to ensure gameplay remains stable
