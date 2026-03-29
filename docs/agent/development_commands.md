# Development Commands

## Build Firmware

### GitHub Actions (Recommended)
Push to any branch to trigger automatic build. Download `.uf2` artifacts from the Actions tab.

### Local Build (requires west + ZMK toolchain)

```bash
# Init workspace (one-time)
git submodule update --init --recursive

# Build main firmware (xiao_ble + adept_board shield + ZMK Studio)
west build -b xiao_ble -s app

# Build settings reset firmware (for BLE clearing / factory reset)
west build -b xiao_ble -s app -- -DSHIELD=settings_reset

# Clean build
west clean && west build -b xiao_ble -s app
```

## Flash Firmware

```bash
# Flash via west (after build)
west flash

# Manual flash (UF2 drag-and-drop)
# Double-tap reset button on XIAO BLE to enter bootloader
# Copy build/zephyr/zmk.uf2 to the mounted XIAO drive
```

## ZMK Studio (Runtime Remapping)

The firmware is built with `snippet: studio-rpc-usb-uart` and `-DCONFIG_ZMK_STUDIO=y`.
Connect via USB and use ZMK Studio web app to remap keys without reflashing.

## Dependency Updates

Edit `config/west.yml` to change module revisions, then rebuild:

```bash
# After editing west.yml
west update
west build -b xiao_ble -s app
```

## Key Module Revisions (config/west.yml)

| Module | Remote | Current Revision |
|---|---|---|
| zmk | zmkfirmware | main |
| zmk-pmw3610-driver | efogdev | zephyr-4.1 |
| zmk-input-processor-report-rate-limit | badjeff | main |
| zmk-pointing-acceleration-alpha | nuovotaka | main |
