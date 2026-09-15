# Changelog

All notable changes to Tiny Ski are documented in this file.

## [Unreleased]

### Added

- **Gamepad Input Support** — Full gamepad controller support alongside existing WASD keyboard controls.
  - Left stick or D-pad for steering (left/right)
  - Left stick up or D-pad up for slowing down
  - Left stick down or D-pad down for speeding up
  - Deadzone filtering on analog stick to prevent drift
  - D-pad takes priority over stick for steering when pressed
  - Keyboard and gamepad inputs are merged (OR logic) — either source can activate controls
  - Compatible with Xbox, PlayStation, and standard USB gamepads
- **Any Gamepad Button Starts/Restarts Game** — Pressing any button on a connected gamepad now activates the "Start Skiing" and "Ski Again" screens, matching the existing keyboard behavior where any key press starts the game. Gamepad button detection runs independently of the game loop so it works on both the initial start screen and the game-over screen. Compatible with Nintendo Switch Pro Controller and all standard Gamepad API controllers.

### Fixed

- **Bug: WASD keyboard controls unresponsive when gamepad is connected** — Rewrote `applyGamepadInput()` to properly merge keyboard and gamepad input without overwriting keyboard state. Previously, the function unconditionally reset `wHeld`, `sHeld`, and `steerDir` to `false`/`0` every frame when the gamepad was idle (stick centered), which cleared any keyboard-set values. Now `wHeld` and `sHeld` are computed as `keysDown[key] || gamepadFlag` (OR logic), and `steerDir` is only cleared when both gamepad and keyboard are idle. Keyboard and gamepad inputs now work correctly independently and when merged.
- **Bug: Player sprite stuck steering after releasing gamepad** — Fixed steering reset logic in `applyGamepadInput()` so `steerDir` is always cleared to `0` when the gamepad stick/D-pad is released. Previously, the condition `if (gamepadSteer !== 0)` prevented reset when the stick returned to center, leaving the player drifting. Now `steerDir` is explicitly set to `0` when both gamepad and keyboard are idle.
- **Bug: D-pad up/down not working** — D-pad up (button 12) and down (button 13) were not being checked for speed control. Added proper button checks for D-pad up/down to set `gamepadSlow` and `gamepadSpeed` flags.
- **Bug: Joystick up not working** — Switch Pro Controller and some other controllers use axis 3 for the left stick Y-axis instead of axis 1. Now checks both axis 1 and axis 3 and sums them for reliable Y-axis reading across all controllers.
- **Bug: Joystick down has too much acceleration** — Acceleration was constant regardless of stick deflection. Now scales acceleration by the amount of stick deflection beyond the threshold, so partial deflection gives proportional acceleration and full deflection gives maximum acceleration. This provides a smooth, consistent experience across all gamepad inputs.
- **Bug: Left joystick up not working for slow down** — The left stick Y-axis was incorrectly summed with axis 3 (right stick Y-axis on Switch Pro Controller and many other controllers). Pushing the right stick would corrupt the slow/speed detection. Fixed to only read axis 1 (the standard left stick Y-axis per the Gamepad API spec).
- **Bug: Right joystick down speeds up the player** — Direct consequence of the axis 3 bug above. Since `axes[3]` was the right stick Y-axis being summed into `stickY`, pushing the right stick down triggered `gamepadSpeed`. Fixed by removing the axis 3 sum.
- **Bug: D-pad up/down conflicting with stick speed detection** — When D-pad up was pressed while the stick was in the speed-up zone, both `gamepadSlow` and `gamepadSpeed` could be true simultaneously, causing unpredictable speed behavior. Now D-pad up clears `gamepadSpeed` and D-pad down clears `gamepadSlow` to prevent conflicts.
- **Bug: Slow down function does not work on any input** — Two issues were causing this: (1) The `slowAmount` formula `Math.min(1, Math.abs(stickY + GAMEPAD_SLOW_THRESHOLD) / (1 - GAMEPAD_SLOW_THRESHOLD))` evaluated to 0 when the stick was exactly at the threshold (-0.5), meaning no deceleration happened at the minimum deflection point. Fixed by using `Math.max(1, ...)` so slowAmount is always at least 1x at the threshold, scaling up to 2x at full stick deflection. (2) `GAMEPAD_SPEED_RAMP_SCALE` (3.0) was being applied to keyboard input whenever a gamepad was connected, making keyboard slow down 3x faster than intended. Now keyboard slow down uses the base `PLAYER_SPEED_RAMP` rate, while gamepad slow down applies the scale factor only when the stick is actively being used for speed control.
- **Bug: Left D-pad down button reverses player (causes slow down instead of speed up)** — The Switch Pro Controller maps D-pad buttons to indices 16-19, not the standard Gamepad API indices 12-15. When D-pad down was pressed, the code checked button 13 (which was not pressed on the Switch Pro), so `gamepadSpeed` was never set to true. Meanwhile, if the left stick was in the slow-down zone, `gamepadSlow` remained true, causing the player to slow down instead of speed up — the opposite of what the player expected. Fixed by adding Nintendo-style button mapping (buttons 16-19) for D-pad up/down/left/right alongside the standard mapping (buttons 12-15) for broad compatibility.
- **Bug: Slow down function does not work when pressing 'W'** — The keyboard input handling was correct, but there was a logic issue in how gamepad and keyboard inputs were being merged. Fixed by ensuring that `wHeld` is properly set to true when the 'W' key is pressed regardless of gamepad state.
- **Bug: Speed up function does notwork when pressing 'S'** — The keyboard input handling for speed control was correct, but there was a conflict in how D-pad and stick inputs were processed. Fixed by ensuring that `sHeld` properly reflects the keyboard 'S' key press.
- **Bug: Gamepad start/restart functionality non-operational during menus/game over** — The gamepad polling loop was previously tied to the main game loop, which stopped running upon collision. Added a dedicated `inputLoop` to ensure continuous monitoring of gamepad buttons for starting and restarting the game.

### Changed

- **Yeti trigger distance reduced from 2048m to 500m** — The Yeti now appears much earlier in the run, making the chase sequence more accessible and frequent.
- **Removed in-game Yeti warning UI** — The flashing "YETI APPROACHING!" warning banner and countdown timer have been removed. The Yeti now appears without any advance warning, making it a surprise encounter for players. The Yeti chase mechanics, collision detection, rendering, and sprite assets remain fully functional.

- **Improved gamepad compatibility** — Enhanced support for various controllers including Nintendo Switch Pro Controller with proper button mapping.