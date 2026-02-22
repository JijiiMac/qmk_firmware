# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A fork of [QMK Firmware](https://qmk.fm) — an open-source keyboard firmware based on TMK. This fork contains the keymap for a **Crossed Keys Nightmare** (50% keyboard with Pro Micro).

## Build Commands

```bash
# Build firmware for the Nightmare keyboard
make nightmare:default

# General pattern: make <keyboard>:<keymap>[:<target>]
make planck/rev4:default          # specific revision
make nightmare:default:dfu        # build and flash via DFU bootloader

# Build all keymaps for a keyboard
make nightmare:all

# Run all unit tests (Google Test, compiled natively)
make test

# Run a specific test group
make test:basic

# Clean test artifacts
make test:clean

# Clean build artifacts for a keyboard
make nightmare:default:clean
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
