# AGENTS.md

This file provides guidance to codeflicker when working with code in this repository.

## WHY: Purpose and Goals

ZMK firmware configuration for a 3-pin modded Ploopy Adept trackball running on Seeeduino XIAO BLE (nRF52840). Features ZMK Studio runtime remapping, scroll layer, PMW3610 cursor acceleration, and BLE report rate optimization (~125Hz).

## WHAT: Technical Stack

- **Firmware:** ZMK (Zephyr-based)
- **MCU:** Seeeduino XIAO BLE (nRF52840)
- **Sensor:** PMW3610 optical (SPI, 600 CPI)
- **Build system:** west (Zephyr)
- **CI:** GitHub Actions (ZMK official build workflow)
- **External modules:** zmk-pmw3610-driver (efogdev), zmk-input-processor-report-rate-limit (badjeff), zmk-pointing-acceleration-alpha (nuovotaka)

## HOW: Core Development Workflow

```bash
# Build firmware via GitHub Actions (push to trigger)
git push

# Local build (requires west + ZMK toolchain installed)
west build -b xiao_ble -s app

# Build settings reset firmware
west build -b xiao_ble -s app -- -DSHIELD=settings_reset

# Init submodules
git submodule update --init --recursive
```

## Progressive Disclosure

For detailed information, consult these documents as needed:

- `docs/agent/development_commands.md` - Build, flash, and west commands
- `docs/agent/architecture.md` - Shield structure, input pipeline, layer/combo system

**When working on a task, first determine which documentation is relevant, then read only those files.**
