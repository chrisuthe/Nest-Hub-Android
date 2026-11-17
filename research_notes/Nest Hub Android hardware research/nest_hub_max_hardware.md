# Google Nest Hub Max (2019, H2A, codename "sherlock") — peripheral hardware for a replacement mainboard

Research date: 2026-11-17. Primary source is the Fuchsia open-source tree (`refs/heads/main`, fetched 2026-11-17; the sherlock board driver is still present). Fuchsia source is authoritative for part numbers, buses, addresses and SoC pin assignments, but says nothing about connectors, FPC pinouts or mechanical layout. No component-level teardown (iFixit or otherwise) of the Hub Max was found; FCC internal photos exist but were not visually inspected in this session.

URL shorthand used below:
- `FX/` = `https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/`
- `SH/` = `FX/src/devices/board/drivers/sherlock/`
- SoC pin names (e.g. GPIOZ_1) are Amlogic T931 balls on the STOCK board. They are irrelevant to a new board except as proof of which signals exist and their polarity.

## Display panel (10.1", 1280x800)

### Takeaway
Two interchangeable panel modules are used: BOE TV101WXM-AG0 (DDIC Fitipower JD9364 or JD9365) and Innolux P101DEZ-DD0 (JD9364). Both are 4-lane MIPI DSI video-mode, natively PORTRAIT 800x1280 @ 60 Hz, with full init sequences published in Fuchsia; backlight is driven by a TI LP8556 at I2C 0x2C. Connector type, pin count and pinout are not public.

### Cited Findings
- Panel variants enumerated for board Sherlock: "BOE TV101WXM-AG0 / Fitipower JD9364", "Innolux P101DEZ-DD0 / Fitipower JD9364", "BOE TV101WXM-AG0 / Fitipower JD9365"; all paired with touch IC FocalTech FT5726 — [display-panel.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h)
- Stock board identifies the fitted panel with two strap pins on the display module: module pin `ID0` = vendor (0 = BOE, 1 = Innolux), module pin `ID1` = DDIC (0 = JD9365, 1 = JD9364). Innolux + JD9365 is rejected as unsupported — [post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/post-init.cc); the strap GPIOs are GPIO_PANEL_DETECT = GPIOH_0 and GPIO_DDIC_DETECT = GPIOH_2 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- All three configs: `dphy_data_lane_count = 4`; `maximum_dphy_clock_lane_frequency_hz = 283'000'000` (source comments note this exceeds the JD936x limit of 250 MHz clock / 500 Mbps per lane and the BOE spec typical of 250 MHz) — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- BOE TV101WXM timing: H active 800, HFP 20, HSYNC 20, HBP 50; V active 1280, VFP 20, VSYNC 4, VBP 20; pixel clock 70.702 MHz; 60 Hz; hsync/vsync negative; progressive — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Innolux P101DEZ timing: H active 800, HFP 46, HSYNC 24, HBP 20; V active 1280, VFP 20, VSYNC 4, VBP 20; pixel clock 70.702 MHz; 60 Hz — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Source cites a "BOE TV101WXM-AG0 spec, Section 6.1 Signal Timing Spec, page 18" and JD9364/JD9365 datasheets (section 11.3) — i.e. such datasheets exist but are not public in the tree — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Full DSI init sequences are published, one header per variant (297 / 264 / 376 lines): [boe-tv101wxm-fitipower-jd9364.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv101wxm-fitipower-jd9364.h), [boe-tv101wxm-fitipower-jd9365.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv101wxm-fitipower-jd9365.h), [innolux-p101dez-fitipower-jd9364.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/innolux-p101dez-fitipower-jd9364.h)
- Sequence shape (BOE variants): sleep 10 ms; LCD reset high 30 ms, low 10 ms, high 30 ms; read register 0x04 (3 bytes, panel ID); sleep 10 + 120 ms; then generic short writes (2 params) starting `E0=00, E1=93, E2=65, E3=F8, 80=03, E0=01 ...` (JD936x page-select style), ending `E0=00` then DCS 0x29 (display on) — [boe-tv101wxm-fitipower-jd9365.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv101wxm-fitipower-jd9365.h)
- LCD reset line: GPIO_LCD_RESET = GPIOH_6 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h); the display driver takes only this GPIO ("gpio-lcd-reset") — [post-init/display.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/display.cc)
- Rail power-on/off ordering is defined as `kLcdPowerOnSequenceForAstroSherlockNelson` / `...PowerOff...` in panel-config.cc (shared with the 7" Nest Hubs); contents were not transcribed in this session — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Backlight driver IC: TI LP8556 at I2C address 0x2C on SoC bus I2C3 (shared with camera sensor 0x36 and "lcd_bias" 0x3E) — [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- LP8556 register setup written by stock software: 0x01 (Device Control) = 0x85; 0xA2 (CFG2) = 0x20; 0xA3 (CFG3) = 0x32; 0xA5 (CFG5) = 0x04; 0xA7 (CFG7) = 0xF4; 0xA9 (CFG9) = 0x60; 0xAE (CFGE) = 0x09; `backlight_max_brightness = 350.0` (nits); same table for BOE and Innolux — [post-init/backlight.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/backlight.cc)
- Backlight enable: GPIO_BACKLIGHT_ENABLE = GPIOA_10 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- An I2C device named "lcd_bias" exists at 0x3E on I2C3; no part number is given — [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- Marketing spec: 10" HD touchscreen 1280x800 — [Google Store tech specs](https://store.google.com/gb/product/google_nest_hub_max_tech_specs?hl=en-GB)

### Inferences
- The replacement SoC must drive a portrait 800x1280 4-lane DSI panel and rotate 90 degrees in software/compositor; the panel is not natively landscape.
- 4 lanes x 24 bpp at 70.7 MHz pixel clock needs roughly 424 Mbps/lane (~212 MHz clock); Fuchsia's 283 MHz ceiling is a cap, not the operating point.
- A new board needs: 4-lane DSI + clock, LCD reset GPIO, two ID strap inputs (to select the init table), an I2C bus for LP8556 (if it lives on the panel side) and the bias IC, and a backlight-enable GPIO.
- The 0x3E "lcd_bias" device is probably a dual-output +/-5.x V LCD bias supply (TPS65132-class parts commonly sit at 0x3E). This is a guess from the address and name only.
- LP8556 CFG values encode LED string count/current/boost settings; decode them against the TI LP8556 datasheet to recover string current and boost frequency rather than guessing.

### Gaps
- NOT PUBLIC: panel FPC connector part, pitch, pin count, pinout. No panelook/datasheet page for TV101WXM-AG0 or P101DEZ-DD0 was found (search returned only other TV101WXM suffixes such as -NP1 eDP 30-pin and -NM0 "40 pins MIPI 4 data lanes", which must not be assumed to match). Must be buzzed out on a real unit.
- Whether the LP8556 and the bias IC are on the mainboard (lost) or on the display FPC (reusable) is not stated anywhere. Check: if the panel FPC carries I2C and LED strings are not exposed, they are panel-side.
- Backlight LED string count, forward voltage and current: not documented; measure or decode LP8556 config.
- Panel logic supply voltages (VDD 1.8/3.3 V, AVDD/AVEE): not documented; read the power-on sequence in panel-config.cc and measure.
- Mainline Linux: I believe JD9365-based panels are covered by `panel-jadard-jd9365da-h3` and LP8556 by `lp855x_bl`, but this was not verified in this session, and no mainline entry for these exact panel models was found.

## Touch panel

### Takeaway
Capacitive touch uses a FocalTech FT5726 at I2C 0x38 with interrupt and reset lines; the stock driver uploads firmware to it. Its physical location and connector are undocumented.

### Cited Findings
- Touch controller FocalTech FT5726, node "focaltech-touch-38", `needs_firmware = true` — [post-init/touch.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/touch.cc)
- I2C address 0x38 on SoC bus I2C2 (only device on that bus; SDA/SCL = GPIOZ_14/15) — [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- GPIO_TOUCH_INTERRUPT = GPIOZ_1, GPIO_TOUCH_RESET = GPIOZ_9 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- The touch driver also receives the panel type as metadata (so firmware/config may differ per panel vendor) — [post-init/touch.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/touch.cc)

### Inferences
- Touch needs 4 signals plus power: SDA, SCL, INT, RESET. The controller is most likely a COF/COB on the touch-sensor flex (typical for FT5x26 modules), which would make it reusable.
- "needs_firmware" means the stock OS can (re)flash controller firmware; a unit that has already run stock software should retain working firmware in the controller's flash, so a new host may not need to upload any. Verify by reading the chip/firmware ID registers.

### Gaps
- NOT PUBLIC: touch connector, pinout, supply voltage (1.8 V vs 3.3 V I/O), whether touch shares the display FPC or has its own.
- Linux support: the `edt-ft5x06` driver family handles FT5x26 parts in my recollection; FT5726 compatibility not verified this session.

## Camera

### Takeaway
The camera is a Sony IMX227 (confirmed by driver name and sensor ID 0x0227) on 2-lane MIPI CSI-2, RAW10, at I2C 0x36, clocked with 24 MHz from the SoC. A raw Bayer sensor means the new SoC needs an ISP pipeline. Module connector and the hardware privacy cut are undocumented.

### Cited Findings
- Sensor node "imx227-sensor"; I2C address 0x36 on bus I2C3 (comment "IMX227 Camera Sensor") — [sherlock-camera.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-camera.cc), [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- Sensor ID constant `kSensorId = 0x0227` — [imx227 constants.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/constants.h)
- Operating modes: 2200x2720 @ 30 fps, 1920x1080 @ 30 fps, plus a "fullsensor" mode (~28 fps) and test patterns; all `lanes = 2`, `mbps = 1000`, RAW10, linear — [imx227_modes.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/imx227_modes.h)
- Register init tables live in `imx227_seq.h`; OTP handling in `imx227_otp.cc` — [imx227 driver dir](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/)
- Control lines: GPIO_CAM_RESET = GPIOZ_0; GPIO_VANA_ENABLE = GPIOA_6; GPIO_VDIG_ENABLE = GPIOZ_12 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- 24 MHz sensor master clock (CLK_CAM_INCK_24M) output on GPIOAO_10, 4 mA drive — [sherlock-camera.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-camera.cc)
- Power-up order in driver: VANA enable high, 50 ms; VDIG enable high, 50 ms; enable 24 MHz clock, 10 ms; drive cam_rst low, 50 ms. Power-down reverses (cam_rst high, clock off, VDIG low, VANA low). Initial state is cam_rst high, enables low — [imx227.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/imx227.cc)
- Stock pipeline: CSI PHY0 -> Amlogic MIPI adapter -> ARM ISP, plus GDC (dewarp) and GE2D — [sherlock-camera.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-camera.cc)
- Marketing: 6.5 MP, 127 degree FOV, "privacy circuit and green/red LED indicator" — [GadgetGuy review, 2019-09-09](https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/)
- WikiDevi lists the camera as "Sony IMX277 - 12Mp ?" with a question mark — [WikiDevi, page dated 2020-10-04](https://wikidevi.wi-cat.ru/Google_Nest_Hub_Max_(H2A)); contradicted by the Fuchsia driver, which is the reliable source (IMX227).

### Inferences
- The stock "reset" GPIO is driven LOW to run the sensor and HIGH to hold it in reset, so there is probably an inverter/level shifter on the stock board (IMX sensors' XCLR is active-low). Confirm polarity at the module connector.
- VANA/VDIG "enable" GPIOs imply the sensor regulators (analog ~2.7-2.8 V, digital ~1.0-1.2 V, plus 1.8 V I/O) are on the mainboard and must be re-created; exact voltages come from the IMX227 datasheet, not from these sources.
- 2200x2720 is about 6.0 MP, consistent with the "6.5 MP" sensor being cropped. The wide FOV means a dewarp step (stock uses GDC) is needed for a normal-looking image.
- Camera, backlight driver and LCD bias share one I2C bus on the stock board; nothing forces that on a new design.

### Gaps
- NOT PUBLIC: camera module connector (type, pitch, pin count), pinout, module maker, lens details, whether the module has its own EEPROM beyond sensor OTP.
- Hardware camera/mic switch wiring: software only sees one GPIO (see Buttons). Whether the slide switch also physically cuts sensor power, MCLK or PDM clock ("privacy circuit") is not documented in any source found. Trace on a real unit.
- No mainline Linux IMX227 driver was identified in this session; the Fuchsia `imx227_seq.h` tables are the only public register sequences found.
- IMX227 datasheet not located.

## Microphones

### Takeaway
Two far-field digital PDM microphones share one PDM clock and one data line into the SoC. Part numbers, location and connector are undocumented.

### Cited Findings
- PDM input configured with `number_of_channels = 2`, `sysClockDivFactor = 4`, `dClockDivFactor = 250` — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- PDM pins: DCLK on GPIOA_7, DIN0 on GPIOA_8 (only one data input is muxed) — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- "Mic: 2 x far-field" — [GadgetGuy review, 2019-09-09](https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/)

### Inferences
- Two mics on one data line means one is strapped L and the other R (standard stereo PDM pair). A new board needs only PDM CLK + one DATA input (or an I2S/PDM codec).
- Mics are very likely on a separate small board/flex near the top bezel beside the camera (reusable) rather than on the mainboard, but no source confirms this.

### Gaps
- Mic part numbers, PDM clock frequency actually used, supply voltage, position, and the connector/flex carrying them: not documented.
- Whether the privacy switch gates mic power or the PDM clock in hardware: not documented.

## Speakers and stock amplifiers

### Takeaway
Three drivers (2 x 18 mm tweeters rated 10 W, 1 x 75 mm woofer rated 30 W) are each driven by its own TI TAS5720 mono class-D amp (I2C 0x6C/0x6D/0x6F) fed by two I2S data lanes; crossover is done digitally at about 1.2 kHz. The amps are mainboard parts and are lost; speaker impedance is undocumented.

### Cited Findings
- "Stereo speaker system (2x 18 mm 10 W tweeters, 1x 75 mm 30 W woofer)" — [Google Store tech specs](https://store.google.com/gb/product/google_nest_hub_max_tech_specs?hl=en-GB)
- Amplifiers: three TI TAS5720 on the always-on I2C bus: 0x6C = tweeter left, 0x6D = tweeter right, 0x6F = woofer — [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc), [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- Audio bus: TDM_C in I2S mode, 16-bit samples in 32-bit slots; pins D0 = GPIOZ_2, D1 = GPIOZ_3, FS = GPIOZ_6, SCLK = GPIOZ_7, MCLK = GPIOAO_9; both data lanes enabled with 2 channels each (`lanes_enable_mask[0] = 3`, `[1] = 3`) — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- Channel split (source comments): 4-channel stream; channel 0 unused; channel 1 = woofer (mono mix of L+R); channel 2 = left tweeter; channel 3 = right tweeter. Woofer amp listens to the right I2S slot, L tweeter the left slot, R tweeter the right slot — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- "Ranges could be wider, but only using them crossed-over at 1'200 Hz in this product"; tweeter amps run 6.4 dB below the woofer amp (`delta_gains = 0, -6.4, -6.4`) — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc)
- Amp control lines: GPIO_SOC_AUDIO_EN = GPIOH_7 (driven high to enable), GPIO_AUDIO_SOC_FAULT_L = GPIOZ_8 (active-low fault) — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- A thermistor channel named "therm-audio" exists (100 k NTC, 47 k pull-up) — [sherlock-thermistor.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-thermistor.cc)

### Inferences
- Lane 0 carries (unused, woofer) and lane 1 carries (L tweeter, R tweeter). A replacement design needs three mono amp channels with a DSP/software crossover around 1.2 kHz, woofer fed L+R mono.
- The 24 V input rail exists mainly for the amplifiers (TAS5720 runs from up to 26.4 V PVDD); a new board can reuse TAS5720 parts directly on 24 V.
- Tweeters passed full-range signal without a high-pass will be damaged; the stock crossover is entirely digital, so there is probably no passive crossover in the enclosure. Verify by inspection.

### Gaps
- Speaker impedance (ohms), true continuous power, and speaker connector types/pinouts: not documented. Measure DC resistance on a real unit.
- Stock EQ/limiter tuning is not in the board driver.
- Whether speaker leads are discrete wire pairs to board connectors or spring contacts: not documented (FCC internal photos would show this).

## Sensors

### Takeaway
The only environmental sensor on a bus is an AMS TCS3400 RGBC ambient light/colour sensor at I2C 0x39 with an interrupt line. There are also three 100 k NTC thermistors read by the SoC ADC. No radar, accelerometer or proximity sensor appears in the board driver.

### Cited Findings
- Light sensor node "tcs3400-light-39"; I2C 0x39 on the always-on I2C bus (shared with the three amplifiers) — [sherlock-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-light.cc), [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- GPIO_LIGHT_INTERRUPT = GPIOAO_5, input, no internal pull — [sherlock-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-light.cc)
- Thermistors: part family "ncpXXwf104" (100 k at 25 C; full R-T table in source), 47 k pull-ups, three channels named therm-base, therm-audio, therm-ambient on SoC SAR ADC channels exposed as "adc-1", "adc-2", "adc-3" — [sherlock-thermistor.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-thermistor.cc), [sherlock-adc.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-adc.cc)
- Status LEDs: "AMBER_LED" on GPIOAO_11 (with PWM_AO_A brightness) and "GREEN_LED" on GPIOH_5 (PWM), both brightness-controllable — [sherlock-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-light.cc); the review describes a green/red camera indicator — [GadgetGuy, 2019-09-09](https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/)

### Inferences
- The TCS3400 sits behind the front bezel (Google's "Ambient EQ" sensor) and is probably on the camera/mic/sensor flex rather than the mainboard, hence reusable; unconfirmed.
- The complete I2C device list in the board file is short (3 amps, ALS, touch, backlight, camera, LCD bias), so there are no other bus sensors to worry about.

### Gaps
- Physical location of the ALS, the LEDs and each thermistor (on mainboard vs remote): not documented.
- Which ADC channel number maps to which thermistor: names suggest channels 1/2/3 but the enum definition was not located.
- Linux driver for TCS3400 not verified.

## Buttons and switches

### Takeaway
Four inputs: volume up, volume down, a "volume both" line used as factory-reset, and the mic/camera privacy slide switch. All are plain GPIO inputs with external pull-ups.

### Cited Findings
- GPIO_VOLUME_UP = GPIOZ_4, GPIO_VOLUME_DOWN = GPIOZ_5, GPIO_VOLUME_BOTH = GPIOZ_13, GPIO_MIC_PRIVACY = GPIOH_3 — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- Button map: volume-up, volume-down, FDR (factory data reset) on the "volume-both" GPIO, and "MicAndCamMute" on the mic-privacy GPIO. Volume up/down/both are flagged inverted (active low); the privacy switch is not inverted (active high). Source comment: "No need for internal pull, external pull-ups used" (though internal pull-ups are also enabled on vol up/down) — [sherlock-buttons.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-buttons.cc)

### Inferences
- "volume-both" being a separate GPIO implies a small hardware AND/diode network that asserts a third line when both volume buttons are held; it may be on the mainboard or the button flex.
- The privacy switch gives the SoC a single logic level; one switch covers both mic and camera.

### Gaps
- Whether buttons/switch are on the mainboard edge (lost) or on a separate flex/daughterboard with a connector: not documented.
- Switch part, connector, pinout: not documented.

## Power input

### Takeaway
External supply is 24 V DC, 30 W (1.25 A) via a barrel jack. Barrel dimensions and polarity are not published by Google and must be measured.

### Cited Findings
- "30 W power adapter", "DC power jack" — [Google Nest device specifications](https://support.google.com/googlehome/answer/7072284?hl=en)
- "24V/1.25 (30W) pin plug adapter 1.5m cable (no battery)" — [GadgetGuy review, 2019-09-09](https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/)
- A Reddit thread (snippet only, not opened) claims "Nest Max: 24 Vdc, 2A ... Most Google Nest products accept same barrel connector and work with 14 Vdc. Only 10" MAX Hub needs 24 Vdc" — [r/googlehome](https://www.reddit.com/r/googlehome/comments/ep5ch8/how_on_earth_to_get_a_nest_hub_max_power_cable/); the 2 A figure conflicts with the 30 W / 1.25 A rating and is unverified.

### Inferences
- Budget a new board for 24 V in at up to ~1.25 A with the stock adapter; step down to 5 V/3.3 V/1.8 V for logic and feed 24 V to the amps.
- The barrel plug reportedly matches other Google Nest speakers physically while the voltage differs (14 V vs 24 V); do not interchange adapters.

### Gaps
- Barrel outer/inner diameter, centre polarity, adapter model number: not found in any source. Measure.
- Whether the DC jack is soldered to the mainboard or sits on a pigtail/daughterboard in the base: not documented.
- Stock input protection and PMIC parts: not documented.

## Mechanical: disassembly, board outline, mounting, thermal

### Takeaway
Essentially undocumented. No iFixit teardown or detailed disassembly write-up for the Hub Max was found; the only public internal imagery is the 10-page FCC internal photo exhibit.

### Cited Findings
- FCC "Internal Photos rev" exhibit, 10 pages, 1.4 MB PDF, public since 2019-11-03 — [fccid.io A4RH2A internal photos](https://fccid.io/A4RH2A/Internal-Photos/Internal-Photos-rev-4255949)
- FCC schematics, block diagram and theory of operation are held under long-term confidentiality (metadata only) — [fccid.io A4RH2A](https://fccid.io/A4RH2A)
- External dimensions 250.1 x 182.55 x 101.23 mm, 1.365 kg — [GadgetGuy, 2019-09-09](https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/); weight 2.91 lb (1.32 kg) — [Wccftech](https://wccftech.com/google-nest-hub-max-has-a-10-display-6-5mp-front-camera-75mm-woofer/)
- A Reddit post asks whether anyone has taken the Hub Max apart and notes only non-Max teardowns exist; mentions display assemblies sold as spares — [r/googlehome, 2021](https://www.reddit.com/r/googlehome/comments/qos819/has_anyone_taken_apart_the_nest_hub_max_found/) (snippet only; page could not be fetched)
- Complete replacement "LCD touch display screen digitizer" assemblies with front camera window are sold on eBay — [eBay listing](https://www.ebay.com/itm/157228724198)

### Inferences
- Replacement display assemblies being sold as one bonded LCD + digitizer + bezel unit suggests the display stack is glued and the practical service boundary is the whole front assembly.
- The base (speaker enclosure) and the display head are separate volumes; where the mainboard sits (behind the display vs in the base) was not confirmed.

### Gaps
- Disassembly procedure, screw locations, adhesive, mainboard outline/dimensions, mounting holes, heatsink and shield arrangement: none documented in text sources. The FCC internal photos PDF should be opened and examined by a human; it was not visually inspected here.
- Inter-board cables and connector families (FPC/ZIF vs board-to-board): not documented.

## Stock compute and what is lost with the mainboard (reference only)

### Takeaway
Stock board: Amlogic T931 SoC, LPDDR4, eMMC, Broadcom BCM43458 Wi-Fi/BT, Nordic nRF52840 Thread radio. Definitely lost with the board: SoC/RAM/eMMC, both radios, the three TAS5720 amps, camera regulators; uncertain: LP8556, LCD bias IC, ALS, mics, buttons, DC jack.

### Cited Findings
- SoC Amlogic T931 (4x Cortex-A73 + 2x Cortex-A53, Mali-G52, NPU); Wi-Fi/BT Broadcom BCM43458 (abgn+ac, 1x1, BT 5.0), antenna connector MHF4 — [WikiDevi, dated 2020-10-04](https://wikidevi.wi-cat.ru/Google_Nest_Hub_Max_(H2A))
- WikiDevi lists flash Samsung KLM4G1FETE-B041 (4 GiB eMMC) and RAM "1 GiB" Micron MT53E512M32D2NP-053 WT:E — [WikiDevi](https://wikidevi.wi-cat.ru/Google_Nest_Hub_Max_(H2A)). The RAM line is internally inconsistent: a 512M x 32 LPDDR4 part is 2 GB, not 1 GiB. Treat both memory figures as unverified.
- Thread/802.15.4 radio: Nordic nRF52840 on SPICC0 (MOSI GPIOC_0, MISO GPIOC_1, SS GPIOC_2, SCLK GPIOC_3, ~1.3 MHz), with reset GPIOA_2, interrupt GPIOA_4, bootloader GPIOA_5 — [sherlock-spi.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-spi.cc), [sherlock-ot-radio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-ot-radio.cc)
- Wi-Fi on SDIO, Bluetooth with PCM audio on TDM_A (GPIOX_8..11), BT_REG_ON GPIOX_17, 32.768 kHz LPO on GPIOX_16 — [sherlock-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc), [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- USB PHY is configured with one peripheral-mode, OTG-capable port — [sherlock-usb.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-usb.cc)
- Five board-ID strap GPIOs (HW_ID0..4) encode board build and option — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- FCC grants cover 2.4 GHz BT/BLE, 2.4/5 GHz WLAN (20/40/80 MHz) and 802.15.4 (2405-2475 MHz) — [fccid.io A4RH2A](https://fccid.io/A4RH2A)

### Inferences
- MHF4 antenna connectors imply the Wi-Fi/BT (and probably Thread) antennas are separate parts in the enclosure on coax pigtails and can be reused with a new radio module.
- A USB device port exists electrically; whether it is exposed as a hidden service connector on the Hub Max was not confirmed.

### Gaps
- Exact RAM/eMMC capacity, PMIC and regulator parts: not reliably sourced.
- Definitive list of which peripherals are soldered to the mainboard requires inspecting the FCC photos or a real unit.

## Public driver sources summary (bus map)

### Takeaway
The Fuchsia sherlock board driver gives a complete logical bus map of the product; it is the single best public reference and should be read alongside a real board.

### Cited Findings
- I2C "AO" bus (SDA/SCL GPIOAO_2/3): 0x6C TAS5720 tweeter L, 0x6D TAS5720 tweeter R, 0x6F TAS5720 woofer, 0x39 TCS3400 ALS. I2C2 (GPIOZ_14/15): 0x38 FT5726 touch. I2C3 (GPIOA_14/15): 0x2C LP8556 backlight, 0x36 IMX227, 0x3E LCD bias — [sherlock-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc)
- Full signal-name-to-pin list — [sherlock-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h)
- Board driver directory (audio, buttons, camera, light, thermistor, spi, usb, post-init/display, backlight, touch) — [sherlock board driver](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/)
- Panel timing/init: [amlogic-display driver](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/); camera: [imx227 driver](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/); backlight: [ti-lp8556 driver](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/ui/backlight/drivers/ti-lp8556/ti-lp8556.cc)

### Inferences
- Minimum host interface set for a drop-in board: 4-lane DSI; 2-lane CSI + 24 MHz MCLK; 2-3 I2C buses; 2 I2S data outputs (or one TDM) + MCLK; 1 PDM clock + 1 PDM data; about 12 GPIOs (LCD reset, backlight enable, 2 panel ID, touch INT/RST, cam reset + 2 regulator enables, amp enable/fault, ALS INT, 4 buttons, 2 LED PWM); 3 ADC inputs for NTCs if retained.

### Gaps
- No mainline Linux device tree for sherlock was found in this session (no search hit); Linux driver availability for each part is from memory and unverified.
- No GitHub/Hackaday hardware-reuse project specific to the Hub Max was found. Hackaday's 2023 teardown covers the 7" Nest Hub 2nd gen, not the Max.
