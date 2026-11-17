# Stock kernel device tree findings

Read 2026-11-17 from Google's GPL kernel source as mirrored at https://github.com/frederic/elaine-linux (Linux 4.9, `arch/arm64/boot/dts/amlogic/` and `arch/arm64/configs/`).

The tree covers **all three models**, not only the 2nd gen. Google's Cast codenames differ from the Fuchsia board names:

| Model | Cast codename (this tree) | Fuchsia board | Files read |
|---|---|---|---|
| Nest Hub 1st gen | estelle | astro | `estelle-p2.dts`, `estelle-panel.dtsi`, `estelle-p2_defconfig` |
| Nest Hub 2nd gen | elaine | nelson | `elaine-b4.dts`, `elaine-sm1-panel.dtsi`, `elaine-bx_defconfig` |
| Nest Hub Max | newman | sherlock | `newman-b4.dts`, `newman-common.dtsi`, `newman-panel.dtsi`, `newman-bx_defconfig` |

Caveats:

- These describe the SoC side: buses, addresses, GPIOs and timings. **They do not give connector pin numbers.** The pinout gap is unchanged.
- Board revisions matter. `b4` is the latest revision in the tree for the 2nd gen and Max. For the 1st gen only prototype revisions (`p0`–`p2`) are present, so its details may differ from retail units.
- Only the files listed were read. Other revisions were not compared.
- GPIO names are Amlogic SoC pins. They matter to us as a list of signals the new board must provide, not as numbers to copy.

## What this settles

- **2nd gen input voltage is 14 V.** The board has a TI INA231 power monitor labelled `ina231_14V` with an undervoltage alert at 11 V. This was unconfirmed before.
- **2nd gen backlight has 3 LED strings; the Max has 6.** The LP8556 is configured for "3 phase drivers" on both 7-inch models and "6-phase, 6 drivers" on the Max, matching the 7S3P and 6S6P datasheet figures.
- **The 1st gen backlight current is 20 mA maximum**, per the LP8556 comment "20mA MAX_CURRENT". This matches the earlier inference.
- **Complete LP8556 register settings** are published for all three models (below). Reusing the LP8556 means copying these values.
- **Vendor touch drivers exist for all three models**, including the FocalTech FT3427 (1st gen), which the driver matrix listed as having no specific driver.
- **Panel power sequencing is simple.** No model shows a multi-rail sequence in the device tree; it is a reset pulse followed by the init table.

## Display

### Panels listed

| Model | Entries in the panel file |
|---|---|
| 1st gen | `TV070WSM`, `TV070WSM_FT`, `P070ACB`, `P070ACB_FT` (plus an unrelated `B080XAN01` reference entry) |
| 2nd gen | `TV070WSM_FT9364`, `TV070WSM_FT9365`, `P070ACB_FT9364`, `KD070D82_FT9364`, `KD070D82_FT9365`, `TV070WSM_ST7703I` |
| Max | `TV101WXM`, `TV101WXM_FT`, `G101B158`, `G101B158_FT` (plus 7-inch entries carried over) |

New information: the 2nd gen has a BOE variant with a **Sitronix ST7703** driver chip, not a Fitipower one. A replacement board must handle it or the panel must be identified first. Mainline Linux has an ST7703 panel driver.

### Timings (2nd gen)

| | BOE (`TV070WSM_*`) | Innolux / K&D |
|---|---|---|
| Active | 600 x 1024 | 600 x 1024 |
| Total (h, v) | 700, 1053 | 770, 1070 |
| Hsync width / back porch | 24 / 36 | 10 / 80 |
| Vsync width / back porch | 2 / 8 | 6 / 20 |
| Pixel clock | 44.226 MHz | 49.434 MHz |
| DSI | 4 lanes, 400 Mbit/s max per lane, burst video mode | same |

Max `TV101WXM_FT`: 800 x 1280 active, 890 x 1324 total, hsync 20 / back porch 50, vsync 4 / back porch 20, 70.7016 MHz, 4 lanes, 566 Mbit/s max.

### Power-on sequence (2nd gen, all Fitipower entries)

1. Drive panel GPIO index 1 low, wait 200 ms. The comment calls this "panel power on". Only one panel GPIO (index 0) is defined in the files read, so what index 1 drives is not shown.
2. Start the DSI signal.
3. Wait 10 ms. Reset high 30 ms, low 10 ms, high 30 ms. Reset is SoC pin GPIOZ_13.
4. Read panel register 0x04 (up to 3 tries) to check the panel responds.
5. Send the init table for the detected panel (the same data as the Fuchsia tables).

Power-off: stop DSI, wait 5 ms, reset low, wait 20 ms, GPIO index 1 high, wait 100 ms.

The ST7703 entry uses longer reset delays (100 / 10 / 220 ms) and checks registers 0xDA, 0xDB, 0xDC.

Max 10.1-inch entries: power-on is only "start DSI signal", then the same 30 / 10 / 30 ms reset pulse. Power-off holds reset low for 100 ms. **The ±5.5 V bias supply does not appear in the kernel device tree**, so it is presumably enabled earlier, by the bootloader. Its sequencing still has to be measured.

## Backlight (TI LP8556 at I2C 0x2C)

| Register | 1st gen (p2) | 2nd gen (b4) | Max | Meaning, per source comments |
|---|---|---|---|---|
| Device control | 0x05 | 0x85 | 0x85 | Brightness by I2C register only, backlight enabled |
| 0xA0 | 0xFF | – | – | Current LSB |
| 0xA1 | 0x37 | – | – | 20 mA maximum current |
| 0xA2 | 0x20 | 0x30 | 0x20 | Undervoltage protection (2.5 V; 5.2 V on 2nd gen) |
| 0xA3 | – | 0x32 | – | 50 ms slope, 12-bit resolution |
| 0xA5 | 0x54 | 0x54 | 0x04 | 9.6 kHz PWM; 3 phases (7-inch), 6 phases (Max) |
| 0xA7 | – | 0xF4 | 0xF4 | Boost inductor current limit 1.6 A |
| 0xA9 | – | 0x60 | 0x60 | Maximum boost voltage 25 V |
| 0xAE | – | 0x09 | 0x09 | LED fault threshold 3 V |

Initial brightness on the 1st gen is 0x800 of 0xFFF (half).

## Touch

| Model | Node | Address | Interrupt | Reset | Notes |
|---|---|---|---|---|---|
| 1st gen | Goodix `gt9xx` | 0x5D | GPIOZ_4 | GPIOZ_9 | Device tree carries a full Goodix config blob |
| 1st gen | FocalTech `fts` | 0x38 | GPIOZ_4 | GPIOZ_9 | 600 x 1023, 10 touches |
| 2nd gen | Goodix GTX8, compatible `goodix,gt9886` | 0x5D | GPIOZ_4, falling edge | GPIOZ_9 | 600 x 1024 |
| Max | FocalTech `fts` | 0x38 | GPIOZ_1 | GPIOZ_9 | 800 x 1280, 5 touches |

All at 400 kHz. On the 1st gen both controllers are declared and the one that answers is used, so a new board can do the same.

Vendor drivers in the tree: `focaltech_touch` (configs for FT3427 and FT5726), `goodix_touch` (GT9xx) and `goodix_touch_gtx8`. The 2nd gen config builds the GTX8 driver with firmware update support.

## Audio

- **1st gen:** TAS2770 at 0x48, I2S format, reset on GPIOA_5.
- **2nd gen:** amplifier at 0x2D, power-down pin GPIOA_5. The node is labelled `tas5805` but uses the `ti,tas5825m` driver, and the kernel config enables TAS5825M and not TAS5805. The two parts are closely related; which is fitted should be read from the chip.
- **Max:** three TAS5720 at 0x6F (woofer), 0x6C (left tweeter) and 0x6D (right tweeter), shutdown on GPIOH_7.
- **Microphones (2nd gen):** PDM clock plus two PDM data lines are enabled; a third data line is commented out. Fuchsia lists three microphones, so the microphone count should be checked on the board.

## Sensors and other devices

| Device | Models | Bus / address | Notes |
|---|---|---|---|
| TCS3400 light sensor | all | I2C 0x39 | Declared as `tcs3472`; the kernel config sets `CONFIG_USE_TCS3400` |
| SHTC3 | 2nd gen | I2C 0x70 | Driver `shtc1` |
| TMP112 temperature sensors | all | I2C 0x48–0x4B | **New.** Four per board, driven as LM75. On the Max they are named for the mic, Wi-Fi, audio and SoC areas. Removed in the Max `b4` revision. |
| NTC thermistors | all | SoC ADC | Murata NCP03WF104, 47 kΩ pull-up to 1.8 V. Four on the 7-inch models, three on the Max |
| INA231 power monitors | 2nd gen | I2C 0x40 (14 V), 0x49 (3.3 V) | **New.** 10 mΩ shunts |
| BMA456 accelerometer | 1st gen p2, Max | I2C 0x18 | **New.** Disabled in the Max `b4` revision; may be prototype-only |
| Radar | 2nd gen | SPI, 25 MHz, chip select GPIOH_6, interrupt GPIOH_3 | Exposed to userspace as `spidev` plus a UIO interrupt. Google also ran the radar from userspace, which supports our plan to do the same. |
| nRF52811 | 2nd gen | SPI, 100 kHz, chip select GPIOC_2 | **New.** Nordic radio chip (Thread). On the mainboard; lost in the swap |
| Camera | Max | I2C 0x36, 24 MHz clock | Reset GPIOZ_0, analog supply enable GPIOA_6, digital supply enable GPIOZ_12. Three control lines a new board must provide. |
| Buttons | all | GPIO | Volume up and down; separate mic mute switch (key code 248) |
| LEDs | all | PWM | Amber on all; green also on the Max |

## Effect on earlier documents

- Driver coverage matrix: the FT3x27 row has a vendor fallback (`focaltech_touch` with an FT3427 config). TMP112 and INA231 are driven here by the standard `lm75` and `ina2xx` drivers. Mainline support for the BMA456 was not checked; the tree carries a vendor `bma4xy` driver.
- Hardware report: 2nd gen supply voltage can be recorded as 14 V.
- Pinout findings: unchanged. Connector pin numbers still need the paid datasheet or a probed unit.

## Not done

- Other board revisions were not compared.
- The Max backlight node for the second panel vendor and the 1st gen panel power steps were only skimmed.
- Vendor driver source was not read, only listed.
- Nothing here was checked against hardware.
