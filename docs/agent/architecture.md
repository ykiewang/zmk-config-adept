# Architecture

## Project Structure

```
zmk-config-adept/
├── build.yaml                        # west build targets
├── config/
│   ├── west.yml                      # Module dependencies
│   ├── adept.conf                    # KConfig settings
│   ├── adept.keymap                  # Layers, combos, behaviors
│   └── info.json                     # ZMK Studio layout metadata
├── boards/shields/adept/
│   ├── adept.zmk.yml                 # Shield metadata (requires xiao_ble)
│   ├── Kconfig.shield                # Shield selection guard
│   ├── Kconfig.defconfig             # Default KConfig when shield selected
│   ├── adept.dtsi                    # Matrix transform (6 buttons, 1 row)
│   ├── adept-layouts.dtsi            # Physical key positions for ZMK Studio
│   ├── adept_board.overlay           # GPIO mapping + input processing pipeline
│   └── pmw3610.dtsi                  # SPI sensor device tree
└── zephyr/module.yml                 # board_root declaration
```

## Input Processing Pipeline

Sensor data flows through a chain of input processors before becoming HID reports:

```
PMW3610 (SPI, 600 CPI, MOT interrupt on P1.15)
    ↓
&pointer_accel       (nuovotaka: standard level, custom preset)
    sensitivity=0.8x, max-factor=3.0x, exponent=1
    threshold=900 counts/sec, max=1800 counts/sec
    ↓
&zip_ble_report_rate_limit 8    (badjeff: 8ms = ~125Hz avg)
    ↓
&zip_xy_transform XY_SWAP       (coordinate axis swap)
    ↓
trackball_listener (ZMK input listener)
    ├── Layer 0 (mouse_layer):  cursor movement → BLE HID mouse report
    └── Layer 1 (scroll_layer):
            &zip_xy_transform XY_SWAP|Y_INVERT
            &zip_xy_scaler 1 16
            &zip_xy_to_scroll_mapper → scroll events
```

## Layer System

| Layer | Name | Purpose |
|---|---|---|
| 0 | mouse_layer | Default: MB1/MB2, Ctrl+PgUp/Dn, MB4/MB5 |
| 1 | scroll_layer | Trackball → scroll; media controls |
| 2-3 | (reserved) | Empty |
| 4 | studio_layer | ZMK Studio unlock |

## Combo System (8 combos)

All interactions use combos because there are only 6 buttons:

| Keys | Action |
|---|---|
| 4+5 | Middle mouse button |
| 2+3 | Toggle scroll layer |
| 0+1 | Escape |
| 1+2 | Ctrl+W (close tab) |
| 0+4 | Alt+Tab (switch app) |
| 0+3 | Alt+F4 (close app) |
| 0+1+2+3 | BLE clear (factory reset) |
| 0+3+4+5 | Bootloader |

## GPIO Pin Mapping (adept_board.overlay)

| Button | Pin | Pull |
|---|---|---|
| sw_1 | P0.3 | PULL_UP, ACTIVE_LOW |
| sw_2 | P0.28 | PULL_UP, ACTIVE_LOW |
| sw_3 | P0.29 | PULL_UP, ACTIVE_LOW |
| sw_4 | P0.4 | PULL_UP, ACTIVE_LOW |
| sw_5 | P0.5 | PULL_UP, ACTIVE_LOW |
| sw_6 | P1.11 | PULL_UP, ACTIVE_LOW |

**SPI (PMW3610):** SCLK=P1.13, MOSI/MISO=P1.14 (bidir), NCS=P1.12, MOT=P1.15

## Key Config Decisions (adept.conf)

- `CONFIG_PMW3610_REPORT_INTERVAL_MIN=4` — 4ms poll, capped at ~125Hz by BLE
- `CONFIG_ZMK_BLE_MOUSE_REPORT_QUEUE_SIZE=1` — prevents report queue buildup
- `CONFIG_ZMK_SLEEP=n` — sleep disabled (avoids reconnect latency)
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` — max TX power for BLE range
- `CONFIG_BT_GATT_ENFORCE_SUBSCRIPTION=n` — fixes battery reporting on Windows
