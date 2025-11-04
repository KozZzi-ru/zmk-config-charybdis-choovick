# ZMK CONFIG FOR THE CHARYBDIS 4X6 WIRELESS SPLIT KEYBOARD

## BOM

Here is the BOM for this project (outdated): [BOM Charybdis 4x6 Wireless](/docs/bom/readme.md)

![Wireless Keyboard](/docs/picture/wireless-charybdis.png)

## Repository Structure

```
zmk-config-charybdis/
├── config/                          # Main ZMK configuration directory
│   ├── boards/                      # Shield board definitions
│   │   └── shields/
│   │       └── charybdis/           # Charybdis shield configuration
│   │           ├── charybdis.dtsi          # Common device tree (keyboard layout, kscan)
│   │           ├── charybdis_left.conf     # Left side Kconfig options
│   │           ├── charybdis_left.overlay  # Left side device tree overlay
│   │           ├── charybdis_right.conf    # Right side Kconfig options
│   │           ├── charybdis_right.overlay # Right side device tree overlay (includes trackball)
│   │           ├── Kconfig.defconfig       # Shield Kconfig definitions
│   │           └── Kconfig.shield          # Shield Kconfig options
│   ├── charybdis.conf               # Global ZMK configuration
│   ├── charybdis.keymap             # Keymap definition file
│   ├── charybdis.zmk.yml           # ZMK build configuration
│   ├── info.json                    # Repository metadata
│   └── west.yml                     # West manifest (see West.yml section below)
├── docs/                            # Documentation
│   ├── bom/                         # Bill of Materials
│   │   └── readme.md
│   ├── keymap/                      # Keymap documentation
│   │   ├── config.yaml
│   │   ├── keymap.svg               # Visual keymap representation
│   │   ├── keymap.yaml
│   │   └── render.sh                # Script to render keymap
│   └── picture/                     # Images
│       └── wireless-charybdis.png
├── build.yaml                       # GitHub Actions build configuration
└── readme.md                        # This file
```

### Key Files Explained

- **`config/charybdis.keymap`**: Defines all key layers, behaviors, and bindings
- **`config/charybdis.dtsi`**: Shared device tree definitions (keyboard matrix, kscan, layer definitions)
- **`config/boards/shields/charybdis/charybdis_right.overlay`**: Right side specific configuration including PMW3610 trackball setup
- **`config/boards/shields/charybdis/charybdis_left.overlay`**: Left side specific configuration
- **`config/west.yml`**: Defines external dependencies (see West.yml section below)

## West.yml Configuration

The `config/west.yml` file defines the ZMK firmware dependencies and external modules used in this configuration.

### Remotes Section

```yaml
remotes:
  - name: zmkfirmware
    url-base: https://github.com/zmkfirmware
  - name: badjeff
    url-base: https://github.com/badjeff
```

- **`zmkfirmware`**: The main ZMK firmware repository, containing the core ZMK application code
- **`badjeff`**: Repository containing the PMW3610 trackball driver used for the Charybdis trackball

### Projects Section

```yaml
projects:
  - name: zmk
    remote: zmkfirmware
    revision: main
    import: app/west.yml
  - name: zmk-pmw3610-driver
    remote: badjeff
    revision: main
```

- **`zmk`**: 
  - **Purpose**: Main ZMK firmware application
  - **Source**: `zmkfirmware` remote
  - **Version**: `main` branch
  - **Import**: Includes additional dependencies from `app/west.yml` in the ZMK repository

- **`zmk-pmw3610-driver`**:
  - **Purpose**: PMW3610 trackball sensor driver for ZMK
  - **Source**: `badjeff` remote
  - **Version**: `main` branch
  - **Note**: This driver provides device tree bindings and driver code for the PMW3610 trackball sensor used on the Charybdis right side

### Self Section

```yaml
self:
  path: config
```

- **`path: config`**: Tells west that this repository's configuration files are located in the `config/` directory

## Keymap

Can be updated at [/config/charybdis.keymap](/config/charybdis.keymap) and rendered with [render.sh](/docs/keymap/render.sh)

Generated with [Keymap Drawer](https://github.com/caksoylar/keymap-drawer-web/)

![Keymap](/docs/keymap/keymap.svg)

## ZMK Studio Support

This configuration includes support for [ZMK Studio](https://zmk.dev/docs/features/studio), which allows you to interactively configure and test your keyboard layout.

### Physical Layout Definition

The physical layout for ZMK Studio is defined in [`config/boards/shields/charybdis/charybdis.dtsi`](/config/boards/shields/charybdis/charybdis.dtsi) in the `charybdis_6col_layout` section. This defines the physical key positions, sizes, and rotations needed for the visual representation in ZMK Studio.

### Enabling/Disabling ZMK Studio

ZMK Studio support is enabled by default via the build configuration in [`build.yaml`](/build.yaml). The right side shield configuration includes:

```yaml
snippet: studio-rpc-usb-uart
cmake-args: -DCONFIG_ZMK_STUDIO=y
```

To disable ZMK Studio support, comment out these two lines (lines 7-8) in `build.yaml`:

```yaml
- board: nice_nano_v2
  shield: charybdis_right
  # snippet: studio-rpc-usb-uart
  # cmake-args: -DCONFIG_ZMK_STUDIO=y
```

### Studio Unlock

To unlock ZMK Studio for configuration, press all three right thumb keys simultaneously:

- **RET** (Return/Enter)
- **SYMBOLS** (hold) / **SPACE** (tap)
- **RAISE** (hold) / **BSPC** (tap)

This combo is defined in [`config/charybdis.keymap`](/config/charybdis.keymap) as `combo_studio_unlock` using key positions 53, 54, and 55.
