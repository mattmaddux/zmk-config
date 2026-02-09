# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ZMK firmware configuration for a Corne (3x6+3, 42-key) split keyboard using Nice Nano v2 controllers. There is no local build system — firmware is built via GitHub Actions on push/PR/manual dispatch using `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`.

## Build & Deploy

There are no local build, lint, or test commands. The workflow is:
1. Edit keymap/config files
2. Push to GitHub to trigger the Actions build
3. Download `.uf2` artifacts from the Actions run
4. Flash each half by copying the `.uf2` to the keyboard in bootloader mode

`build.yaml` defines the build matrix: `nice_nano_v2` board with `corne_left` and `corne_right` shields.

## Architecture

### Key Files
- **`config/corne.keymap`** — The main file you'll edit. Contains layer definitions, custom behaviors, and key bindings in Devicetree syntax.
- **`config/corne.conf`** — Board config toggles (RGB underglow, OLED display). Features are commented out by default.
- **`config/west.yml`** — West manifest pinning ZMK dependency (currently tracks `main`).
- **`build.yaml`** — GitHub Actions build matrix.

### Keymap Structure

The keymap uses Devicetree overlay syntax with ZMK-specific bindings from `<behaviors.dtsi>`, `<dt-bindings/zmk/keys.h>`, and `<dt-bindings/zmk/bt.h>`.

**4 layers** (indexed 0-3):
- **0 (default)**: QWERTY with home row mods
- **1 (char)**: Symbols, navigation arrows, brackets
- **2 (numpad)**: Numbers, volume controls
- **3 (bluetooth)**: BT profile management

Layer access: `&mo 1` (momentary) on left thumb activates char layer; `&thmbshft 2 SPACE` on right thumb gives space on tap, numpad layer on hold; `&mo 3` in numpad layer reaches bluetooth.

### Home Row Mods

Two mirrored hold-tap behaviors prevent misfires by restricting which keys can trigger the hold:
- **`hml`** (left hand): hold-trigger-key-positions = right hand keys + right thumb
- **`hmr`** (right hand): hold-trigger-key-positions = left hand keys + left thumb

Key position macros define the split: `KEYS_L` (0-5, 12-17, 24-29), `KEYS_R` (6-11, 18-23, 30-35), `THUMBS_L` (36-38), `THUMBS_R` (39-41).

Shared hold-tap parameters: `balanced` flavor, `tapping-term-ms=300`, `quick-tap-ms=175`, `require-prior-idle-ms=100`, `hold-trigger-on-release`.

### Custom Behavior: `thmbshft`

A hold-tap bound to `<&mo>, <&kp>` — hold activates a layer (`mo`), tap sends a keypress. Used for the right thumb space key (tap=SPACE, hold=layer 2).

## Editing Guidelines

- Key codes come from ZMK's `keys.h` (e.g., `LSHFT`, `SEMI`, `FSLH`, `BSPC`). Reference: https://zmk.dev/docs/keymaps/list-of-keycodes
- The Corne layout is 6 columns x 3 rows + 3 thumb keys per side. Each layer binding block must have exactly 42 entries.
- When adding/changing home row mods, keep `hold-trigger-key-positions` consistent with the hand split — left-hand mods should only list right-hand positions and vice versa.
- Layer numbers are implicit (order of appearance in the `keymap` node). Adding/removing/reordering layers changes all subsequent layer indices — update `&mo` and `&to` references accordingly.
