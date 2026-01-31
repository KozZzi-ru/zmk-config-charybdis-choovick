# ZMK CONFIG ДЛЯ CHARYBDIS 4X6 WIRELESS SPLIT KEYBOARD (ZEPHYR 4.1)

Этот репозиторий содержит конфигурацию ZMK для беспроводной сплит-клавиатуры Charybdis 4x6 (Nice!Nano / nRF52840) с трекболом PMW3610. [cite:24]

Поддерживается только **Standalone Mode**: правая половинка работает как central и подключается напрямую к хосту по USB/BLE, левая половинка подключается к правой. [cite:24]

## Repository Structure

```text
zmk-config-charybdis/
├── boards/                          # Module-based shields (Zephyr 4.1+ recommended layout)
│   └── shields/
│       ├── charybdis/               # Charybdis shield configuration
│       │   ├── charybdis.dtsi                        # Common device tree (keyboard layout, kscan)
│       │   ├── charybdis_layers.h                    # Shared layer definitions
│       │   ├── charybdis_trackball_processors.dtsi   # Shared trackball processing config
│       │   ├── charybdis_right_common.dtsi           # Shared right keyboard hardware config
│       │   ├── charybdis_left.conf                   # Left side Kconfig options (empty)
│       │   ├── charybdis_left.overlay                # Left side device tree overlay
│       │   ├── charybdis_right_standalone.conf       # Right side Kconfig (standalone mode)
│       │   ├── charybdis_right_standalone.overlay    # Right side overlay (standalone mode)
│       │   ├── Kconfig.defconfig                     # Shield Kconfig definitions
│       │   └── Kconfig.shield                        # Shield Kconfig options
│       └── tester_pro_micro/         # Pro Micro GPIO tester shield
├── config/                          # Main ZMK configuration directory (keymap + west manifest)
│   ├── charybdis.conf               # Global ZMK configuration
│   ├── charybdis.keymap             # Keymap definition file
│   ├── charybdis.zmk.yml            # ZMK build configuration
│   ├── info.json                    # Repository metadata
│   └── west.yml                     # West manifest
├── manual_build/                    # Local build scripts
├── docs/                            # Documentation (keymap, pictures, etc.)
├── build.yaml                       # GitHub Actions build configuration
└── zephyr/
    └── module.yml                   # Zephyr module marker
```

## Keymap

Редактируй раскладку в [/config/charybdis.keymap](/config/charybdis.keymap) и при необходимости генерируй SVG в [/docs/keymap](/docs/keymap). [cite:24]

## Trackball Sensitivity Configuration

Настройки трекбола делятся на два уровня: аппаратная чувствительность сенсора (CPI) и программное масштабирование (скорость курсора/скролла). [cite:24]

### Hardware Sensor Sensitivity (CPI/DPI)

CPI для PMW3610 задаётся в [`boards/shields/charybdis/charybdis_right_common.dtsi`](/boards/shields/charybdis/charybdis_right_common.dtsi). [cite:24]

```dts
trackball: trackball@0 {
    compatible = "pixart,pmw3610";
    cpi = <800>;  // Change this value
    // ...
};
```

### Software Scaling (Movement Speed)

Масштабирование скорости задаётся по слоям в [`boards/shields/charybdis/charybdis_trackball_processors.dtsi`](/boards/shields/charybdis/charybdis_trackball_processors.dtsi). [cite:24]

```dts
// Normal cursor movement (BASE and POINTER layers)
move {
    layers = <BASE POINTER>;
    input-processors = <&zip_xy_scaler 7 6>;  // multiplier divisor
};

// Precise movement (SNIPING layer)
snipe {
    layers = <SNIPING>;
    input-processors = <&zip_xy_scaler 1 3>;  // 1/3 speed for precision
};

// Scroll mode (SCROLL layer)
scroll {
    layers = <SCROLL>;
    input-processors = <&zip_xy_scaler 1 10>;  // Adjust scroll speed
};
```

## ZMK Studio Support

Поддержка ZMK Studio включена для правой половинки через настройки сборки в [`build.yaml`](/build.yaml). [cite:24]

```yaml
- board: nice_nano
  shield: charybdis_right_standalone
  snippet: studio-rpc-usb-uart
  cmake-args: -DCONFIG_ZMK_STUDIO=y
```

Чтобы отключить Studio, закомментируй `snippet` и `cmake-args` в `build.yaml`. [cite:24]

### Studio Unlock

Чтобы открыть режим настройки в ZMK Studio, нажми три правые клавиши большого пальца одновременно. [cite:24]

- **RET** (Return/Enter) [cite:24]
- **SYMBOLS** (hold) / **SPACE** (tap) [cite:24]
- **RAISE** (hold) / **BSPC** (tap) [cite:24]

## Building Firmware

Сборка в GitHub Actions формирует только нужные прошивки из [`build.yaml`](/build.yaml). [cite:24]

- `charybdis_left-nice_nano-zmk.uf2` [cite:24]
- `charybdis_right_standalone-nice_nano-zmk.uf2` [cite:24]
- `settings_reset-nice_nano-zmk.uf2` [cite:24]

## Flashing (Standalone Mode)

1. Прошей `settings_reset-nice_nano-zmk.uf2` на **обе** половинки. [cite:24]
2. Прошей `charybdis_left-nice_nano-zmk.uf2` на левую половинку. [cite:24]
3. Прошей `charybdis_right_standalone-nice_nano-zmk.uf2` на правую половинку. [cite:24]
4. Подключай USB к правой половинке. [cite:24]
