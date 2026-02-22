# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A fork of [QMK Firmware](https://qmk.fm) — an open-source keyboard firmware based on TMK. This fork contains the custom keymap for a **Helix rev2** split keyboard (4-row configuration with OLED enabled).

## This Fork's Custom Keymap

The owner's keymap lives at `keyboards/helix/rev2/keymaps/jijiimac/`. It defines 3 layers on a 4-row Helix rev2:

- **Layer 0** — QWERTY base. MO(1) on right thumb for Layer 1, MO(2) on left thumb for Layer 2.
- **Layer 1** — Arrow keys (HJKL-style on left) + numpad (right side).
- **Layer 2** — Symbols: `!@#$%^` row, `*&[]_=` row, `<>` row.

Key `rules.mk` settings: `HELIX_ROWS = 4`, `OLED_ENABLE = yes`, `LED_ANIMATIONS = yes`. All other features (mouse keys, audio, backlight, MIDI, Bluetooth, etc.) are disabled to save firmware size.

## Build Commands

```bash
# Build this fork's keymap
make helix/rev2:jijiimac

# Flash to keyboard via avrdude
make helix/rev2:jijiimac:avrdude

# General pattern: make <keyboard>:<keymap>[:<target>]
make planck/rev4:default          # specific revision

# Build all keymaps for a keyboard
make helix/rev2:all

# Run all unit tests (Google Test, compiled natively)
make test

# Run a specific test group
make test:basic

# Clean test artifacts
make test:clean

# Clean build artifacts for a keyboard
make helix/rev2:jijiimac:clean
```

Build output goes to `.build/`. Final firmware is a `.hex` (AVR) or `.bin` (ARM) file at the repo root.

## Architecture

**Layer model (bottom to top):**

1. **TMK Core** (`tmk_core/`) — Base keyboard firmware: matrix scanning, keycodes, action processing, USB protocols (LUFA for AVR, ChibiOS for ARM)
2. **Quantum** (`quantum/`) — QMK's feature layer on top of TMK: tap dance, combos, leader key, RGB/LED matrix, audio, MIDI, unicode, macros, and more
3. **Keyboard definition** (`keyboards/<name>/`) — Hardware-specific: pin mapping, matrix layout, default features
4. **Keymap** (`keyboards/<name>/keymaps/<keymap>/`) — User key layout and custom behavior

**Configuration inheritance:** Each level has `rules.mk` (build flags/features) and `config.h` (compile-time settings). Keymaps inherit from keyboard, which inherits from quantum/TMK defaults. Keyboards can nest up to 5 directory levels deep (e.g., `keyboards/vendor/line/model/revision/`).

**Feature system:** Features are compile-time opt-in via `rules.mk` flags (e.g., `AUDIO_ENABLE = yes`, `COMBO_ENABLE = yes`). The `common_features.mk` file maps these flags to source file includes.

**Build dependency chain:**
`Makefile` → `build_keyboard.mk` → keyboard `rules.mk` → `common_features.mk` → `quantum/mcu_selection.mk` → TMK platform rules → compiler

## Key Directories

- `keyboards/` — All keyboard definitions (thousands), organized by manufacturer/name
- `quantum/` — QMK features (`quantum.h` is the master include)
- `quantum/process_keycode/` — Per-keycode feature processing (tap dance, unicode, etc.)
- `tmk_core/common/` — Core action/keymap/matrix handling
- `tmk_core/protocol/` — USB protocol implementations (LUFA, V-USB, ChibiOS)
- `drivers/` — Hardware drivers (LED, OLED, haptic, GPIO)
- `layouts/` — Community-shared layouts reusable across compatible keyboards
- `tests/` — Google Test unit tests with mock matrix/drivers
- `util/` — Setup scripts, flashing utilities, keyboard/keymap generators

## Code Style

- C: 4-space indent, no tabs (except Makefiles which use tabs). See `.clang-format` (Google-based, 1000 col limit, pointer right-aligned).
- Python: 4-space indent, 200 char max line length.
- Makefiles/`.mk`: tabs for indentation, LF line endings.
