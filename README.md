# ⛷️ Tiny Ski

A top-down skiing game built with HTML5 Canvas and vanilla JavaScript. Navigate a skier down a procedurally generated mountain slope, avoiding obstacles and outrunning the Yeti.

## Game Description

Tiny Ski is a fast-paced arcade skiing game where the player controls a skier descending a snowy mountain. The terrain scrolls beneath the player like a treadmill, with procedurally generated texture tiles creating a varied alpine environment.

### Gameplay

- **Objective:** Ski as far as possible without colliding with obstacles. Survive until 2048m to trigger the Yeti chase sequence.
- **Controls:**
  - **W** — Slow down
  - **S** — Speed up
  - **A** — Steer left
  - **D** — Steer right
- **Obstacles:** Trees, stumps, bushes, rocks, and snowmen are randomly scattered across the terrain. Collision with any obstacle ends the game.
- **Boundary Markers:** Visual markers line the left and right edges of the navigable terrain, repeating every 20 meters.
- **The Yeti:** After skiing 2048m, a Yeti appears and begins chasing the player. It accelerates as it gets closer and can navigate around obstacles. Collision with the Yeti ends the game.
- **Scoring:** Score equals distance traveled (in meters) plus bonus points for snowballs collected.

### Technical Details

- **Rendering:** HTML5 Canvas 2D with pixel-art sprite assets
- **Terrain:** Procedurally generated tile grid with 6 terrain textures (snow and ice variants)
- **Collision Detection:** Circle-based collision between the player and all obstacles/boundary markers
- **Graphics:** 16×16 pixel tile assets from the `assets/Tiles/` directory
- **Canvas Size:** 480 × 720 (portrait orientation)
- **Player:** Centered on screen; terrain scrolls to simulate forward movement

## Getting Started

1. Open `tinyski.html` in a modern web browser (Chrome, Firefox, Edge, or Safari).
2. Click **Start Skiing** to begin.
3. Use **W/S** to control speed and **A/D** to steer.
4. Avoid obstacles and try to survive as long as possible!

> **Note:** `tinyski.html` relies on sprite assets located in the `assets/Tiles/` folder. Ensure the assets directory is present when running the game.

## Assets

All graphical assets are stored in `assets/Tiles/`:

| Folder | Contents |
|--------|----------|
| `terrain_textures/` | Snow and ice ground tiles (tile_0000 through tile_0052) |
| `player_sprite/female/` | Player skier sprite frames |
| `yeti_sprite/` | Yeti chase sprite frames |
| `collision_objects/` | Obstacle sprites (trees, rocks, snowmen, bushes) |
| `alphanum_objects/` | Alphanumeric sprite assets |
| `ski_lift_objects/` | Ski lift decorative sprites |
| `slalom_objects/` | Slalom gate sprites |
| `unused/` | Unused/legacy assets |

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for a detailed history of changes and bug fixes.

## Bug Fixes

See [BUGFIX.md](BUGFIX.md) for a summary of identified and resolved bugs.
