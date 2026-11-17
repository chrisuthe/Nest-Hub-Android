# Google Nest Hub 1st gen (Google Home Hub, H1A, codename "astro") — peripheral hardware reference for a replacement mainboard

Research date: 2026-11-17. Primary sources used:

- **Fuchsia open-source tree, `main` branch, read 2026-11-17** (board driver `src/devices/board/drivers/astro/` and display driver `src/graphics/display/drivers/amlogic-display/`). "Astro" is the Fuchsia board name for this product. This is Google's own board support code, so part names, I2C addresses and SoC GPIO assignments from it are treated as confirmed. Abbreviated below as FX-ASTRO = `https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro` and FX-DISP = `https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display`.
- **EDN teardown by Brian Dipert, published 2020-10-14** (photos and chip markings).
- **Google support spec page**, read 2026-11-17.

Important scope caveat for the report writer: the Fuchsia sources give the **SoC-side** view (which S905D2 pin / bus each peripheral uses). They do NOT give connector part numbers, FPC pin counts/pitch, or connector pinouts. **No public source found documents any connector pinout for this device.** All connector-level facts must be measured on a real unit.

## Display panel (size, resolution, part number, interface, backlight, init sequence, connector)

### Takeaway
Two interchangeable 7.0" 600x1024 (portrait-native, mounted landscape) 4-lane MIPI-DSI panels were shipped: BOE TV070WSM-TG1 and Innolux P070ACB-DB0, both with a Fitipower JD9364 driver IC; full DSI init sequences and timings are published in Fuchsia. Backlight is driven by a TI LP8556 at I2C 0x2C. Connector type, pitch, pin count and pinout are not publicly documented.

### Cited Findings
- Panel variant A: "Panel: BOE TV070WSM-TG1, DDIC: Fitipower JD9364, Board: Astro, Touch IC: FocalTech FT3x27". Panel variant B: "Panel: Innolux P070ACB-DB0, DDIC: Fitipower JD9364, Board: Astro, Touch IC: Goodix GT9293" — [Fuchsia display-panel.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h)
- The board distinguishes the two panels at boot by reading one GPIO strap named "disp-soc-vid": 0 = BOE, 1 = Innolux — [FX-ASTRO post-init/post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/post-init.cc)
- SoC pins: `GPIO_LCD_RESET` = GPIOH_6, `GPIO_PANEL_DETECT` = GPIOH_5, `GPIO_BACKLIGHT_ENABLE` = GPIOA_10 — [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h)
- Interface: MIPI DSI, `dphy_data_lane_count = 4` for both panels. BOE config caps the D-PHY clock lane at 180 MHz (panel spec allows 145–250 MHz); Innolux config caps at 200 MHz (panel spec 125–250 MHz). JD9364 allows 80–500 Mbps per lane with 4-lane 24-bit pixel data — [FX-DISP panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- BOE TV070WSM timing as used by Google: H active 600, HFP 40 (datasheet typical 20), HSYNC 24, HBP 36; V active 1024, VFP 19 (datasheet typical 6), VSYNC 2, VBP 8; pixel clock 44.226 MHz (datasheet typical 42.4 MHz); 60 Hz; hsync and vsync negative polarity — [FX-DISP panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Innolux P070ACB-DB0 timing (datasheet typical values): H active 600, HFP 80, HSYNC 10, HBP 80; V active 1024, VFP 20, VSYNC 6, VBP 20; pixel clock 49.434 MHz; 60 Hz; negative sync polarities — [FX-DISP panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Full DSI init sequence for the BOE panel on Astro is published (a ~1,480-line annotated header, `lcd_init_sequence_BOE_TV070WSM_FITIPOWER_JD9364_ASTRO`). It starts with: wait 10 ms after VCC, reset pin high 30 ms, low 10 ms, high 30 ms, then reads DCS 0x04 (display ID, 3 bytes), then JD9364 page-select (0xE0) register writes. Comments cite the BOE spec: VCC-ready to reset > 10 ms; reset low pulse > 10 µs; reset to MIPI LP mode > 5 ms — [FX-DISP panel/boe-tv070wsm-fitipower-jd9364-astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv070wsm-fitipower-jd9364-astro.h)
- Full DSI init sequence for the Innolux panel is also published (~250 lines) — [FX-DISP panel/innolux-p070acb-fitipower-jd9364.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/innolux-p070acb-fitipower-jd9364.h)
- Backlight driver: TI LP8556, I2C address 0x2C, on the same I2C bus as the audio amp (Fuchsia "I2C_3", SoC pins GPIOA_14 SDA / GPIOA_15 SCL). Registers Google writes at init: 0x01=0x85, 0xA2=0x30, 0xA3=0x32, 0xA5=0x54, 0xA7=0xF4, 0xA9=0x60, 0xAE=0x09. `backlight_max_brightness = 400.0` (nits) — [FX-ASTRO post-init/backlight.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/backlight.cc), [FX-ASTRO astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h), [FX-ASTRO astro-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-i2c.cc)
- Product-level: 7" LCD, 1024x600, 169.5 ppi — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Physically, the display assembly is glued to the body and joined to the mainboard by a single flex PCB cable ("the flex PCB cable interconnecting the two halves"). The mainboard back side also has a set of conductive pads that press against the back of the screen (author guesses grounding) — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Panelook lists TV070WSM-TG1 as a BOE 7.0" a-Si TFT-LCD "Assembly", 600(RGB)x1024 WSVGA, RGB vertical stripe (search snippet only; the page itself was blocked by a bot check) — [Panelook](https://m.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_overview_51122.html)
- The Fuchsia display driver README lists the reference documents Google used: Fitipower/Jadard JD9364 Data Sheet v1.07 (2021-02-26), JD9364 User Guide v1.01 (2018-05-31) — [FX-DISP README.md](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/README.md)

### Inferences
- Because the EDN teardown shows only one flex between the screen half and the body, DSI, panel power, backlight LED strings and touch I2C/INT/RST very likely share that single FPC (not confirmed by pin-level evidence).
- The LP8556 is on a mainboard I2C bus and is configured by the SoC, which suggests the backlight boost/driver IC is on the mainboard and only LED anode/cathode strings go to the panel. If so, a replacement board needs its own LED backlight driver. Not confirmed: the LP8556 could instead sit on the panel FPC. Must be checked on a unit.
- "GPIO_PANEL_DETECT" (GPIOH_5) is probably the "disp-soc-vid" vendor strap; the name mapping lives in astro-gpio.cc and was not read line by line.
- A replacement board must support both panel variants (or the builder must identify which one their unit has, e.g. by the strap pin or by which touch address answers: 0x38 = BOE unit, 0x5D = Innolux unit).
- JD9364-based 600x1024 4-lane panels are common in tablets, so mainline Linux `panel-jadard-jd9365da-h3`-style drivers are a plausible starting point, but no mainline Linux driver naming these exact panels was verified in this research.

### Gaps
- **Connector part number, FPC pitch, pin count, pinout: not found in any public source.** Must be measured (microscope + continuity from DSI lanes / known nets).
- Panel supply rails and voltages (VDD/VDDI, any AVDD/AVEE generated on-board vs on-FPC) not found. BOE and Innolux datasheets are cited by Fuchsia comments but were not located publicly.
- Backlight LED string count, forward voltage and current: not found. The LP8556 register values above encode the current/string config and can be decoded against the TI LP8556 datasheet, but that decode was not done here.
- The panel power-on/off GPIO sequence table (`kLcdPowerOnSequenceForAstroSherlockNelson`) is referenced in panel-config.cc but its definition was not located in the files read.
- Panelook parameter page could not be read (bot verification), so outline dimensions, brightness and viewing-angle figures are unconfirmed.

## Touch panel (controller, location, bus, interrupt/reset, connector)

### Takeaway
Touch controller depends on the panel variant: FocalTech FT3x27 at I2C 0x38 (BOE units) or Goodix GT9293 at I2C 0x5D (Innolux units), on a dedicated I2C bus with one interrupt and one reset line. Whether the controller IC sits on the touch FPC or the mainboard is not stated in any source.

### Cited Findings
- I2C addresses: `I2C_FOCALTECH_TOUCH_ADDR = 0x38`, `I2C_GOODIX_TOUCH_ADDR = 0x5d` — [FX-ASTRO astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h)
- Both touch devices are on Fuchsia bus "I2C_2" (S905D2 I2C2); SoC pins `GPIO_SOC_TOUCH_I2C_SDA` = GPIOZ_14, `GPIO_SOC_TOUCH_I2C_SCL` = GPIOZ_15; `GPIO_TOUCH_INTERRUPT` = GPIOZ_4; `GPIO_TOUCH_RESET` = GPIOZ_9 — [FX-ASTRO astro-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-i2c.cc), [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h)
- "Innolux P070ACB panel on Astro uses Goodix touch controller chip"; "BOE TV070WSM panel on Astro uses Focaltech FT3x27 touch controller chip" — [FX-ASTRO post-init/touch.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/touch.cc)
- Goodix variant: driver expects interrupt line pulled up and reset line pulled down (SoC internal pulls). FocalTech variant: interrupt line has no SoC pull ("driven by the touch controller"); metadata `device_id = kFt3X27`, `needs_firmware = false` (no firmware download needed on Astro) — [FX-ASTRO post-init/touch.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/touch.cc)
- Goodix part is named as GT9293 in the panel-type comment — [Fuchsia display-panel.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h)

### Inferences
- `needs_firmware = false` and the fact that the controller type follows the panel vendor (not the mainboard revision) strongly suggest the touch IC is part of the display/touch module (on its FPC), so it survives a mainboard swap. Not confirmed visually.
- Both controllers have mainline Linux drivers (`edt-ft5x06` family for FocalTech, `goodix` for GT9xx), so OS support on a replacement board should be straightforward; exact compatibility of "FT3x27" and "GT9293" with those drivers was not verified here.

### Gaps
- Touch I/O voltage (1.8 V vs 3.3 V), supply rail, and which FPC pins carry SDA/SCL/INT/RST: not documented; measure.
- Whether touch shares the display FPC or has a separate tail/connector: not confirmed (EDN mentions only one flex to the screen half).
- Exact FocalTech part (FT3427 vs FT3327 etc.): source says only "FT3x27".

## Microphones (count, type, part numbers, location, connector)

### Takeaway
Two MEMS microphones with PDM digital output, mounted on a small secondary PCB (with the ambient light sensor and mic-mute switch) at the top of the unit, connected to the mainboard by its own flex cable. Part numbers are not public.

### Cited Findings
- "2 Far-field microphones" — [Google support spec page, read 2026-11-17](https://support.google.com/googlehome/answer/7072284?hl=en)
- "a secondary PCB containing the ambient light sensor and two MEMS microphones"; "The flex PCB cable at the top in the foreground connects the primary PCB to the secondary PCB"; mic openings are on the front, one each side of the ambient light sensor; the secondary board carries "the two MEMS microphones along with the ambient light sensor/microphone mute switch cluster" — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Mic input is PDM: SoC pins GPIOA_7 = PDM_DCLK, GPIOA_8 = PDM_DIN0; PDM config `number_of_channels = 2`, `sysClockDivFactor = 4`, `dClockDivFactor = 250`; audio HIFI PLL set to 768 MHz — [FX-ASTRO astro-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-audio.cc)

### Inferences
- Two channels on a single PDM data pin (DIN0) implies the two mics share one data line with opposite L/R select (standard stereo PDM). A replacement board needs one PDM clock out and one PDM data in.
- 768 MHz / 250 = 3.072 MHz PDM clock (48 kHz x 64), if the divider is applied directly to the HIFI PLL. This is a calculation from the config values, not a stated figure.
- Since the mics are on the daughterboard, they are reusable with a new mainboard.

### Gaps
- Microphone manufacturer/part number: not found.
- Daughterboard flex connector type, pin count, pinout, supply voltage: not found; measure.
- Whether the mic-mute slide switch electrically cuts mic power/clock or is only a GPIO read by software: not documented (Fuchsia reads it as a GPIO; see Buttons section).
- Google lists "Ultrasound sensing" for this model; no dedicated ultrasound hardware was identified in the sources (probably uses the speaker and mics; unconfirmed).

## Speaker and amplifier

### Takeaway
One full-range driver in a sealed plastic enclosure in the base, connected to the mainboard by a two-wire cable. The stock amplifier is a TI TAS2770-family (TAS27xx) digital-input mono Class-D amp at I2C 0x48, fed by I2S. Driver size, impedance and power rating are not reliably documented.

### Cited Findings
- "Full-range speaker" is the only official description for the 1st gen Nest Hub — [Google support spec page, read 2026-11-17](https://support.google.com/googlehome/answer/7072284?hl=en)
- iFixit's device page lists "2 in Full-Range speaker" for the Home Hub 1st gen (search snippet; unsourced wiki text) — [iFixit device page](https://www.ifixit.com/Device/Google_Home_Hub_1st_Generation)
- "the two-wire cable in the background connects the primary PCB to the speaker"; speaker cone faces into the device and fires up through vents; speaker sits in a heavy two-piece black plastic enclosure; strong magnet — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Amplifier/codec: Fuchsia binds a TI TAS2770 driver (`BIND_PLATFORM_DEV_DID_TAS2770`, codec type `Tas27xx`), device name "audio-codec-48", `I2C_AUDIO_CODEC_ADDR = 0x48`, on I2C_3 (GPIOA_14 SDA / GPIOA_15 SCL) — [FX-ASTRO astro-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-audio.cc), [FX-ASTRO astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h)
- Amp digital audio: S905D2 TDM_B in I2S mode, 16-bit samples in 32-bit slots; pins GPIOA_1 = TDMB_SCLK, GPIOA_2 = TDMB_FS, GPIOA_3 = TDMB_D0 (out), GPIOA_6 = TDMB_DIN3 (in); codec uses the right I2S slot; mono; 48 kHz and 96 kHz listed; delta gain -1.5 dB — [FX-ASTRO astro-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-audio.cc)
- Amp control pins: `GPIO_SOC_AUDIO_EN` = GPIOA_5 (pulsed low 1 ms then high as hardware reset), `GPIO_AUDIO_SOC_FAULT_L` = GPIOA_4 — [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h), [FX-ASTRO astro-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-audio.cc)

### Inferences
- The amplifier is on the mainboard (it is addressed on a mainboard I2C bus and only a 2-wire cable goes to the speaker), so it is lost with the stock board; a replacement needs its own mono amp. A TAS2770 (or similar I2S smart amp) is the like-for-like reference.
- TDMB_DIN3 is most likely the amp's I/V-sense feedback data returned to the SoC (TAS2770 feature); not stated in the source.
- The 14 V input rail is consistent with feeding a Class-D amp directly from the adapter rail (TAS2770 accepts up to 16 V PVDD per TI's public datasheet); the actual PVDD source was not confirmed.
- A search snippet attributing a "1.7 in (43.5 mm) driver" and "three far-field microphones" belongs to the Nest Hub 2nd gen section of Google's spec page, not this model. Do not use it for gen 1.

### Gaps
- Speaker impedance, rated power, exact driver diameter: not found in any reliable source. Measure DCR with a multimeter; check for markings on the driver.
- Speaker cable connector type (2-pin wire-to-board; family/pitch unknown): measure.
- TAS2770 init register sequence is compiled in from a non-public config header (`TAS2770_CONFIG_PATH`), so Google's DSP/limiter tuning is not public.
- EDN did not identify the amplifier chip marking, so the exact TAS27xx variant is not visually confirmed.

## Sensors (ambient light / colour "Ambient EQ", others)

### Takeaway
The Ambient EQ sensor is an ams TCS3400 RGB+clear colour light sensor at I2C 0x39 with an interrupt line, located on the top daughterboard with the microphones. The only other sensor found is a 100 k NTC thermistor read by the SoC ADC.

### Cited Findings
- Light sensor: device "tcs3400-light-39", `I2C_AMBIENTLIGHT_ADDR = 0x39`, on the always-on I2C bus (I2C_AO_0); SoC pins `GPIO_SOC_SENSORS_I2C_SDA` = GPIOAO_2, `GPIO_SOC_SENSORS_I2C_SCL` = GPIOAO_3; interrupt `GPIO_LIGHT_INTERRUPT` = GPIOAO_5. Driver config: gain 64, integration time 711.68 ms, polling 700 ms — [FX-ASTRO astro-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-light.cc), [FX-ASTRO astro-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-i2c.cc), [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h)
- The ambient light sensor is on the secondary PCB with the two mics, between the two mic openings at the top front — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Official sensor list: "Ambient EQ light sensor", "Ultrasound sensing", "Capacitive touch controls"; no camera — [Google support spec page, read 2026-11-17](https://support.google.com/googlehome/answer/7072284?hl=en)
- Thermistor: NTC part string "ncpXXwf104" with a 100 kΩ @ 25 °C table, read through the SoC SAR ADC — [FX-ASTRO astro-thermistor.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-thermistor.cc)
- An "AMBER_LED" light is defined on GPIOAO_11, PWM-dimmed (PWM_AO_A) — [FX-ASTRO astro-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-light.cc)

### Inferences
- The TCS3400 is on the daughterboard, so it is reusable; its I2C + INT lines travel over the same daughterboard flex as the PDM mic signals.
- "ncpXXwf104" matches Murata's NCP-series 100 k NTC naming; the thermistors are almost certainly on the mainboard and would be lost (replace with any 100 k NTC if thermal monitoring is wanted).
- The amber LED is probably the mic-mute indicator; its physical location (daughterboard vs mainboard) is not documented.
- The sensor sits behind the display cover glass ink, so lux/colour calibration is unit-design-specific; Google's calibration coefficients were not found.

### Gaps
- Daughterboard schematic/pinout and supply voltage for the TCS3400: not found; measure.
- Number and placement of thermistors: not extracted (ADC channel list not read in detail).
- No accelerometer, proximity, or temperature/humidity sensor was found in the board driver; absence in Fuchsia is good but not absolute proof.

## Buttons and switches

### Takeaway
Volume up, volume down and the mic-mute slide switch are plain GPIO inputs to the SoC; volume buttons are active-low with pull-ups. Their wiring path (which flex) is only partly documented.

### Cited Findings
- `GPIO_VOLUME_UP` = GPIOZ_5, `GPIO_VOLUME_DOWN` = GPIOZ_6, `GPIO_VOLUME_BOTH` = GPIOAO_10 (used as factory-data-reset input), `GPIO_MIC_PRIVACY` = GPIOZ_2 — [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h)
- Volume up/down: SoC internal pull-up enabled, logic inverted (active low). Volume-both and mic-privacy: no internal pull ("external pull-ups used"); volume-both inverted, mic-privacy not inverted — [FX-ASTRO astro-buttons.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-buttons.cc)
- Physical placement: mic mute switch at the top of the back; volume up/down along the left edge of the back; mic-mute switch is in the cluster on the top secondary PCB — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)

### Inferences
- "VOLUME_BOTH" on a separate always-on-domain pin implies a small diode/logic AND of the two volume buttons on the stock board so that holding both wakes/reset-triggers the SoC; a replacement board can ignore this and read the two buttons separately.
- The volume button switches are probably on the mainboard edge or a small flex behind the rear housing; EDN does not say which.

### Gaps
- Where the volume switches physically sit and how they connect (mainboard-mounted tact switches vs flex): not documented; inspect.
- Whether the mic-mute switch has a hardware cut of the mic supply/clock in addition to the GPIO: not documented.

## Power input

### Takeaway
14 V DC, 1.1 A (15 W) from a wall adapter via a barrel jack. The jack is not on the mainboard: it sits on a small passive connector board in the base (together with a hidden micro-USB service port) that links to the mainboard by flex. Barrel dimensions are not authoritatively documented.

### Cited Findings
- Adapter label: input 100–240 V 50–60 Hz 0.4 A; output 14 V / 1.1 A (~15 W); barrel connector; 1.5 m cord — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- "15 W power adapter", "DC power jack" — [Google support spec page, read 2026-11-17](https://support.google.com/googlehome/answer/7072284?hl=en)
- Under the glued rubber foot is a hidden micro-USB connector next to the power input. The power jack + micro-USB are on a small module that "contains no active circuitry, only signal routing traces (and the connectors themselves)", held by a plastic reinforcement bracket, and joined to the mainboard by a flex PCB cable — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Third-party sellers list the adapter model as W18-015N1A (14 V 1.1 A) and adapter-cable listings describe the plug as 3.5 mm x 1.35 mm (retail listing snippets, low reliability; one is a Q&A on a 2nd-gen product page) — [AliExpress listing](https://www.aliexpress.com/i/1005008505390379.html), [Target Q&A](https://www.target.com/p/google-nest-hub-with-sleepsense-sand/-/A-80209779?showOnlyQuestions=true)

### Inferences
- Because the jack board is passive and reusable, a replacement mainboard can take 14 V in over the stock base flex (if its pinout is traced) and must provide its own buck converters (e.g. 14 V to 5 V / 3.3 V / 1.8 V / core rails). All stock regulators are on the mainboard and are lost.
- The micro-USB port on the same passive board gives a ready-made route for a USB data port on a replacement design (USB D+/D-/VBUS/ID run up the same flex), subject to tracing.
- Centre-positive polarity is typical for this adapter family but was not confirmed by a source; check the adapter label.

### Gaps
- Barrel plug outer/inner diameter and polarity: no authoritative source; measure with calipers (3.5 x 1.35 mm is an unverified seller claim).
- Base flex connector type, pin count and pinout (14 V, GND, USB signals): not found; trace with a multimeter.
- Input protection components (fuse, TVS, reverse-polarity) on the stock board: not documented.

## Mechanical: disassembly, board outline, mounting, thermal

### Takeaway
Entry is destructive-risk: the display is glued to the body and must be pried/heated off first; everything else is Torx screws. The mainboard carries a metal heat sink over two shield cans and sits behind the display above the speaker enclosure. No board dimensions or mounting-hole drawings are published.

### Cited Findings
- External size: depth 67.3 mm, width 178.5 mm, height 118 mm; weight 480 g — [Google support spec page, read 2026-11-17](https://support.google.com/googlehome/answer/7072284?hl=en)
- Disassembly order: (1) separate the adhesive-bonded display assembly from the body (the author cracked the glass using a thin screwdriver); (2) disconnect the single display flex; (3) remove "several dozen" screws, mostly T4 or T8 Torx plus one Phillips near the antennas; (4) lift out the PCB assembly (mainboard + heat sink + top daughterboard), leaving the speaker/power-jack assembly; (5) rubber foot is only glued and peels off to expose micro-USB — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Three interconnects leave the mainboard besides the display flex: a flex to the base (power jack + micro-USB board), a flex to the top daughterboard (mics/ALS/mute switch), and a two-wire speaker cable — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Thermal: a metal heat sink covers two shield cans; thermal tape between heat sink and can, and between can and SoC — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Antennas: an orange flex antenna and a second antenna structure sit beside the mainboard near the Wi-Fi/BT chip — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Speaker faces inward and uses the chassis as its acoustic enclosure/port path, venting at the top — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- FCC filing exists with Internal Photos exhibit (listed as dated 2019-03-19 on the index page) — [fccid.io A4RH1A](https://fccid.io/A4RH1A). Note: EDN's text gives the FCC ID as "AR4F1A", which appears to be a typo for A4RH1A.
- A YouTube screen-replacement video exists for this model (not reviewed) — [Google Home Hub 1 - Screen Replacement](https://www.youtube.com/watch?v=AE75AVNT12w)

### Inferences
- Stock thermal design is passive for an S905D2-class SoC (a few watts) in a sealed plastic shell; a replacement SoC of similar or lower power with a comparable heat spreader should be fine, while a hotter SoC (e.g. RK3588-class) would need thermal validation. This is engineering judgement, not sourced.
- The internal antennas are separate parts connected to the mainboard, so they may be reusable with a replacement Wi-Fi module if the connector type (not documented) matches.
- Since the speaker uses the chassis volume acoustically, air leaks or blocking the internal vent path with a larger board would change the sound.

### Gaps
- Mainboard outline, dimensions, mounting-hole positions, component height limits: not published anywhere found. Must be measured/scanned from a real board.
- FCC internal photos PDF was not opened in this research; it may show connector positions and is worth reviewing.
- No iFixit teardown for this exact model was found (only a device page and Q&A).
- Antenna connector type (U.FL/MHF vs spring contacts) not documented.

## Stock compute hardware (reference only) and what is lost with the mainboard

### Takeaway
Stock board: Amlogic S905D2G SoC, about 1.5 GB DDR3 (four Nanya chips), 512 MB (4 Gbit) SLC raw NAND, Broadcom BCM4345 Wi-Fi/BT. On the mainboard and therefore lost: SoC/RAM/NAND, Wi-Fi/BT, all regulators, LP8556 backlight driver (probably), TAS2770 speaker amp, thermistors. Off-board and reusable: display + touch module, mic/ALS/mute daughterboard, speaker, power-jack/micro-USB board, antennas.

### Cited Findings
- SoC: Amlogic S905D2G (quad Cortex-A53) — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- RAM: 2x Nanya NT5CC256M16EP-EK (4 Gbit DDR3 each) on the top side + 2x Nanya NT5CC128M16JR-EK (2 Gbit each) on the bottom side = 12 Gbit (1.5 GB) total — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Storage: Kioxia/Toshiba TC58NVG2S0HBAI6, 4 Gbit SLC NAND — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/); Fuchsia confirms raw NAND (astro-rawnand.cc) — [FX-ASTRO directory](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/)
- Wireless: Broadcom BCM4345 dual-band Wi-Fi + Bluetooth in its own shield can; Wi-Fi 802.11b/g/n/ac, Bluetooth 5.0. Fuchsia shows it on SDIO with a 32.768 kHz LPO clock from GPIOX_16 (PWM_E), BT_REG_ON GPIOX_17, SDIO reset GPIOX_6 — [EDN teardown, 2020-10-14](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/), [FX-ASTRO astro-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h)
- Board revision straps: HW_ID0 = GPIOZ_7, HW_ID1 = GPIOZ_8, HW_ID2 = GPIOZ_3; only DVT and PVT revisions supported — [FX-ASTRO astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h), [FX-ASTRO post-init/post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/post-init.cc)
- "S905D2 u200" reference board is cited by a Reddit hacking thread as the Nest Hub's CPU platform, with a USB port under the foot (community post, 2021) — [Reddit r/googlehome](https://www.reddit.com/r/googlehome/comments/puru6r/hacking_the_google_nest_hub_firmware/)

### Inferences
- Summary table of SoC-side signal budget a replacement board must provide to reuse all peripherals: 4-lane MIPI DSI + LCD reset GPIO + panel-ID strap input; backlight LED driver (enable + dimming); I2C #1 for touch (0x38 or 0x5D) + INT + RST; I2C #2 for TCS3400 (0x39) + INT; stereo PDM in (1 clock, 1 data); mono I2S out to a new amp; 3–4 GPIO inputs for vol+/vol-/mute; optional LED PWM; USB 2.0 to the base micro-USB; 14 V input conversion.
- Three separate I2C buses on stock is a layout convenience; there are no address clashes among 0x2C, 0x38, 0x39, 0x48, 0x5D, so fewer buses could be used, subject to I/O voltage domains (unknown).

### Gaps
- PMIC/regulator part numbers on the stock board: not identified by EDN or Fuchsia.
- I/O voltage of each peripheral bus (S905D2 GPIOZ, GPIOA, GPIOAO banks can be 1.8 V or 3.3 V): not documented for this board; measure before connecting.

## Public driver sources naming the parts

### Takeaway
The Fuchsia tree is the single best public reference: it names the panels, DDIC, touch ICs, backlight driver, light sensor, amp, thermistor, and gives addresses, pins, timings and full panel init code. No mainline Linux device tree for this board was confirmed.

### Cited Findings
- Board driver file list (astro-audio.cc, astro-buttons.cc, astro-gpios.h, astro-i2c.cc, astro-light.cc, astro-thermistor.cc, astro-pwm.cc, astro-sdio.cc, astro-bluetooth.cc, astro-rawnand.cc, astro-usb.cc, post-init/{display,touch,backlight}.cc) — [FX-ASTRO directory](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/)
- I2C map (7-bit): 0x2C LP8556 backlight (I2C_3), 0x38 FocalTech touch (I2C_2), 0x39 TCS3400 (I2C_AO_0), 0x48 TAS2770 (I2C_3), 0x5D Goodix touch (I2C_2) — [FX-ASTRO astro.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h), [FX-ASTRO astro-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-i2c.cc)
- Panel configs, timings and init code — [FX-DISP panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc), [FX-DISP panel/ directory files](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv070wsm-fitipower-jd9364-astro.h)
- Hackaday's "Google Nest Hub Teardown" (2023-05-29) covers the 2nd gen (S905D3G, Soli radar), not this model; do not use it for gen 1 — [Hackaday](https://hackaday.com/2023/05/29/google-nest-hub-teardown/)

### Inferences
- Fuchsia init sequences use a simple opcode stream (sleep / GPIO / DCS write / read-reg) that can be transcribed into a Linux `panel-simple-dsi`-style or JD9365-family panel driver init table.
- The Fuchsia `main` branch may drop Astro support in future; the files should be archived (commit-pinned) by anyone relying on them.

### Gaps
- Mainline Linux device tree / panel driver for "astro" or these exact panel part numbers: not searched in depth and none confirmed.
- Google's GPL kernel/U-Boot source drops for this product (which would contain an Amlogic vendor DTS with LCD power sequencing, backlight LED config and regulator names) were not located in this research; worth a follow-up search.
- No hobbyist project reusing the gen 1 display or documenting its FPC pinout was found (Hackaday, GitHub, Reddit searches returned nothing specific).
