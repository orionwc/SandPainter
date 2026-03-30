# SandPainter

SandPainter is a custom image-and-color painting workflow for the CrunchLabs Sand Garden Hackpack.  
It combines:

1. Arduino firmware (`SandPainter.ino`) that draws patterns in sand and mirrors the path on a circular RGB LED display.
2. A browser-based pattern editor (`index.html`) that lets you "paint" LED-aligned points and export pattern data in firmware-ready format.

Unlike the original [Image2Sand](https://github.com/orionwc/Image2Sand/), SandPainter focuses on **LED-aware, color-aware pattern authoring** (point types + RGB color per point), with startup calibration support for aligning the LED ring to the real sand motion.

---

## Files In This Repo

- `SandPainter.ino` - Arduino firmware for motion, pattern selection, calibration, and LED-circle rendering.
- `index.html` - Local web tool for creating/editing SandPainter point data.

---

## Hardware Needed

### Required

- CrunchLabs Sand Garden Hackpack (Arduino + motors + joystick)
- Circular RGB LED display (WS2812/NeoPixel-compatible), configured for:
  - 241 total LEDs
  - 9 concentric rings
  - Ring counts: `60, 48, 40, 32, 24, 16, 12, 8, 1`

### Wiring (LED Circle)

In `SandPainter.ino`, the LED circle data pin is:

- `CIRCULAR_LED_PIN 2` -> connect LED circle **data input** to Arduino **D2**

Power notes:

- The LED circle should be powered by a **separate external power source** (as requested).
- Share **ground** between the external LED power source and Arduino ground so data signaling is reliable.

---

## What SandPainter Does

SandPainter extends the picture workflow to support:

- **Point types** per coordinate:
  - `0` = draw/marble movement
  - `1` = fill point (LED-only paint with short delay)
  - `2` = detail point (LED-only paint with longer delay)
- **Explicit RGB color** at each point (`LEDpos` format)
- Circular LED tracing that matches the intended pattern geometry
- A startup calibration path and theta-offset adjustment so LED orientation can be aligned with physical sand orientation

This means you can create richer, color-coded light patterns while still drawing in sand.

---

## How To Use `SandPainter.ino`

## 1) Configure and upload

Open `SandPainter.ino` and confirm key settings:

- `USE_LED_CIRCLE true`
- `ENABLE_COLORED_PATTERN_SUPPORT true`
- `ENABLE_CALIBRATION true`
- `PAUSE_AFTER_START true`
- `AUTO_START_PATTERN false` (default pattern-selection startup)

Then upload to your Sand Garden Arduino.

## 2) First run behavior (what to expect)

On boot, firmware will:

1. Home the radial axis.
2. Enter pattern selection mode (since `AUTO_START_PATTERN` is false).
3. When you start a pattern, it first runs a calibration path:
   - center -> nominal outer reference `(1000, 0)` -> center
4. It then pauses in calibration mode.

During calibration pause:

- Move joystick left/right to adjust LED angular offset (theta offset, `THETA_OFFSET_JUMP_DEG` steps).
- Press joystick button to accept calibration and start the selected pattern.

## 3) Pattern selection and start

- Use joystick up/down to change selected pattern.
- Press joystick button to start the current pattern.
- With `PAUSE_AFTER_START true`, the pattern pauses after reaching its first point:
  - Press joystick button again to continue full drawing.

## 4) During drawing

- Type `0` points move the marble and draw path segments.
- Type `1` and `2` points update LED circle colors without moving the marble (with timing delays).
- If `PATTERN_LOOP_ON_COMPLETE` is false (default), pattern ends and returns to selection mode.

---

## Customizing Patterns in `SandPainter.ino`

- Enable/disable custom slots (`ENABLE_CUSTOM_PATTERN_A` ... `J`).
- Paste exported data into the corresponding pattern arrays.
- The expected colored format is:

`{{r, theta}, type, {R, G, B}}`

Where:

- `r` is `0..1000`
- `theta` is `0..3599` (tenths of a degree)
- `type` is `0`, `1`, or `2`
- `R/G/B` are `0..255`

---

## `index.html` LEDpaint Program

`index.html` is a local visual editor that maps clicks to the same concentric LED geometry used in firmware and generates copy-paste-ready SandPainter points.

## What it does

- Renders a 9-ring LED circle matching firmware topology.
- Lets you paint points with:
  - Draw mode (type `0`)
  - Fill mode (type `1`)
  - Detail mode (type `2`)
- Lets you pick RGB color by color picker or numeric entry.
- Produces output in `LEDpos` format used by `SandPainter.ino`.
- Includes playback simulation of generated points.

## Main features

- Draw path with optional "Point per LED"
- Fill selection tools:
  - Select click/drag
  - Flood select by same-color region
  - Rectangle selection
  - Select/Deselect actions
- Undo / Redo (`Ctrl+Z`, `Ctrl+Shift+Z`)
- Clear pattern
- Copy output
- Simulate output path and timing
- Hidden rainbow mode via Konami code

## How to use it

1. Open `index.html` in a browser.
2. Pick a point type (`Draw`, `Fill`, or `Detail`).
3. Set color (picker or RGB fields).
4. Paint points on the LED canvas.
5. Use `Simulate` to preview behavior.
6. Copy output text.
7. Paste into one of the enabled pattern arrays in `SandPainter.ino`.
8. Upload firmware and select that pattern on device.

---

## Credit

SandPainter builds on the Sand Garden image workflow concept originally published in [Image2Sand](https://github.com/orionwc/Image2Sand/).
