# ZMK CONFIG FOR THE CHARYBDIS 4X6 WIRELESS SPLIT KEYBOARD ZEPHYR 4.1

Этот репозиторий содержит конфигурацию ZMK для беспроводной сплит‑клавиатуры Charybdis 4x6 на базе Zephyr 4.1.

## Tester Pro Micro Shield

В репозитории есть **ZMK Tester Shield** (`tester_pro_micro`) для диагностики и проверки плат, совместимых с форм‑фактором Pro Micro (например, Nice!Nano, Seeeduino XIAO и др.). Он отображает 18 GPIO (D0–D10, D14–D16, D18–D21) как «виртуальные клавиши», которые печатают `PIN X` при срабатывании — так можно быстро убедиться, что нужные пины исправны до окончательной сборки.

**Как использовать:**

1. Прошейте `tester_pro_micro-nice_nano-zmk.uf2` на контроллер (см. раздел BUILDING FIRMWARE).
2. Подключите плату по USB к компьютеру.
3. Откройте любой текстовый редактор.
4. Замкните выбранный GPIO на GND (кнопкой/проводом).
5. Плата напечатает `PIN X`, где X — номер пина (например, `PIN 14`).

Тестер работает только по USB (BLE отключён) и содержит два physical layout для визуализации в ZMK Studio.

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
│           ├── Kconfig.shield                        # Shield identifier
│           ├── Kconfig.defconfig                     # Shield defaults (USB-only, no BLE)
│           ├── tester_pro_micro.zmk.yml              # Shield metadata
│           ├── tester_pro_micro.overlay              # GPIO pin definitions (18 pins)
│           ├── tester_pro_micro.keymap               # Pin test macros
│           └── tester_pro_micro-layouts.dtsi         # Physical layouts (pinout + single row)
├── config/                          # Main ZMK configuration directory (keymap + west manifest)
│   ├── charybdis.conf               # Global ZMK configuration
│   ├── charybdis.keymap             # Keymap definition file
│   ├── charybdis.zmk.yml            # ZMK build configuration
│   ├── info.json                    # Repository metadata
│   └── west.yml                     # West manifest
├── manual_build/                    # Local build scripts
│   ├── build.py                     # Interactive build script
│   └── BUILD_README.md              # Build instructions
├── docs/                            # Documentation
│   ├── keymap/                      # Keymap documentation
│   │   ├── config.yaml              # Keymap drawer configuration
│   │   ├── keymap.yaml              # Base keymap definition
│   │   ├── keymap.svg               # Generated: Visual keymap (created by render.sh)
│   │   └── render.sh                # Script to parse keymap and generate SVG
│   └── picture/                     # Images
│       └── wireless-charybdis.png
├── build.yaml                       # GitHub Actions build configuration
├── zephyr/
│   └── module.yml                   # Zephyr module marker (enables discovering boards/shields/)
└── readme.md                        # This file
```

## Keymap

Раскладка настраивается в `/config/charybdis.keymap`, а для генерации SVG используется `/docs/keymap/render.sh`.

## Trackball Sensitivity Configuration

Чувствительность трекбола можно настроить как на уровне «железа» (CPI), так и программно через input processors.

### Hardware Sensor Sensitivity (CPI/DPI)

CPI для датчика PMW3610 задаётся в `boards/shields/charybdis/charybdis_right_common.dtsi`:

```dts
trackball: trackball@0 {
    compatible = "pixart,pmw3610";
    cpi = <800>;  // Change this value
    // ...
};
```

### Software Scaling (Movement Speed)

Программное масштабирование скорости задаётся по слоям в `boards/shields/charybdis/charybdis_trackball_processors.dtsi`.

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

### How Scaler Values Work

Формула работы scaler: `output = (input × multiplier) / divisor`.

⚠️ Важно: используйте значения ≤ 16 для multiplier и divisor, чтобы избежать переполнения.

## ZMK Studio Support

В конфигурации включена поддержка ZMK Studio для интерактивной проверки и настройки раскладки.

### Studio Unlock

Чтобы разблокировать ZMK Studio, нажмите одновременно три правые клавиши большого пальца:

- RET
- SYMBOLS
- RAISE

## BUILDING FIRMWARE

Сборка прошивок выполняется через GitHub Actions на основе `build.yaml`, артефакты содержат `.uf2` файлы для нужных конфигураций.

Для локальной сборки через Docker используйте инструкции в `manual_build/BUILD_README.md`.

## FLASHING FIRMWARE

1. Дважды нажмите RESET на плате, чтобы войти в bootloader.
2. Плата появится как USB‑накопитель.
3. Скопируйте подходящий `.uf2` файл на этот накопитель.
4. Плата автоматически прошьётся и перезагрузится.

### Flashing checklist (reset settings first)

1. Прошейте `settings_reset-nice_nano-zmk.uf2` на обе половины.
2. Прошейте `charybdis_left-nice_nano-zmk.uf2` на левую половину.
3. Прошейте `charybdis_right_standalone-nice_nano-zmk.uf2` на правую половину.

### Tester Pro Micro (GPIO Testing)

1. Прошейте `tester_pro_micro-nice_nano-zmk.uf2` на контроллер.
2. Подключите его по USB.
3. Замыкайте GPIO на GND и проверяйте вывод `PIN X`.
