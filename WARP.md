# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This is a ZMK (Zephyr Mechanical Keyboard) firmware configuration repository for a Corne keyboard using Nice Nano v2 controllers. ZMK is a modern, wireless-first mechanical keyboard firmware built on the Zephyr RTOS.

## Architecture & Structure

### Core Configuration Files
- **`config/corne.keymap`**: The primary keymap definition containing:
  - Layer definitions (default, character, numpad, gaming, bluetooth)
  - Custom behaviors (home row mods, space-shift combo)
  - Key position macros for left/right hand separation
- **`config/corne.conf`**: Board-specific configuration settings (RGB, OLED display options)
- **`config/west.yml`**: West manifest defining ZMK dependencies and project structure
- **`build.yaml`**: GitHub Actions matrix configuration defining which boards/shields to build

### Build System
- **`zephyr/module.yml`**: Zephyr module definition for custom board configurations
- **`.github/workflows/build.yml`**: Triggers ZMK's official build workflow for user configs
- **`boards/shields/`**: Directory for custom shield definitions (currently unused)

### Key Concepts

**Layers**: The keymap uses a 5-layer system:
1. **Default layer (0)**: QWERTY with home row mods
2. **Character layer (1)**: Symbols, navigation arrows, and special characters
3. **Numpad layer (2)**: Media controls, numbers, and app shortcuts
4. **Gaming layer (3)**: Standard QWERTY without home row mods
5. **Bluetooth layer (4)**: Bluetooth connection management

**Home Row Mods**: Custom hold-tap behaviors (`hml`/`hmr`) that act as modifiers when held, regular keys when tapped. Uses separate definitions for left/right hands to prevent accidental activation.

**Hold-Tap Parameters**:
- `tapping-term-ms`: How long to hold before activating hold behavior
- `quick-tap-ms`: Rapid tap threshold to prefer tap behavior
- `require-prior-idle-ms`: Idle time required before hold-tap can activate
- `hold-trigger-key-positions`: Keys that can trigger the hold behavior

## Common Commands

### Building Firmware
The repository uses GitHub Actions for automated building. Firmware is built automatically on:
- Push to any branch
- Pull requests
- Manual workflow dispatch

Local building requires ZMK development environment setup (not recommended for keymap changes).

### Testing Changes
1. Modify `config/corne.keymap` for layout changes
2. Commit and push changes to trigger GitHub Actions build
3. Download firmware artifacts from GitHub Actions results
4. Flash `.uf2` files to keyboard halves

### Configuration Changes
- **Enable RGB underglow**: Uncomment RGB lines in `config/corne.conf`
- **Enable OLED display**: Uncomment display line in `config/corne.conf`
- **Add new boards/shields**: Modify `build.yaml` include section
- **Update ZMK version**: Change `revision` in `config/west.yml`

### Keymap Modifications
When editing the keymap:
- Key positions are defined by the 3x6+3 Corne layout (42 keys total)
- Use ZMK key codes from `dt-bindings/zmk/keys.h`
- Layer numbers start at 0 and must be referenced correctly in `&mo` (momentary) and `&to` (toggle) functions
- Home row mod positions are defined by `KEYS_L`, `KEYS_R`, `THUMBS_L`, `THUMBS_R` macros

### Bluetooth Management
Layer 4 provides bluetooth controls:
- `&bt BT_SEL n`: Select bluetooth profile (0-4)
- `&bt BT_CLR`: Clear current profile
- `&bt BT_CLR_ALL`: Clear all profiles
- `&bt BT_PRV`/`&bt BT_NXT`: Navigate between profiles

## Development Workflow

1. **Make keymap changes** in `config/corne.keymap`
2. **Test locally** using ZMK's keymap drawer or simulator (optional)
3. **Commit changes** with descriptive commit messages
4. **Push to GitHub** to trigger automatic build
5. **Download firmware** from GitHub Actions artifacts
6. **Flash to keyboard** by copying `.uf2` files to keyboard in bootloader mode

## Repository Dependencies

- **ZMK Firmware**: Main dependency managed through West manifest
- **Zephyr RTOS**: Underlying operating system (managed by ZMK)
- **GitHub Actions**: Uses `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`

The configuration automatically stays updated with the latest ZMK main branch unless manually pinned to a specific revision in `west.yml`.