# Google Nest Hub 2nd gen (2021, GUIK2, Fuchsia board name "nelson") — internal hardware for peripheral reuse

Research date: 2026-11-17. Source legend (all fetched this session unless marked otherwise):

- **Fuchsia** = Fuchsia open-source tree, `refs/heads/main` as of Nov 2026, board driver `src/devices/board/drivers/nelson/` (BSD licence). Primary source: this is Google's own board support for this product. Signal names in it (e.g. `GPIO_SOC_TOUCH_RST_L`) appear to be schematic net names.
- **Hackaday.io** = txyz.info teardown project (logs dated March 2023; a datasheet file added 2025-11-16). Hobbyist teardown with logic-analyser captures; part numbers read off chips.
- **E360/TechInsights** = Electronics360 article dated 2021-08-11 summarising a TechInsights teardown. Secondary summary; contains at least one apparent error (see conflicts).
- **FCC** = fccid.io listing for A4RGUIK2 (grant 2021-01-04; photos released 2021-07-03).

IMPORTANT caveat for the report writer: I did NOT open the FCC internal-photo PDF, the teardown video, or the Hackaday pinout photos (images only, no text). Connector pin counts, FPC pitches, connector part numbers and pin-by-pin pinouts were not found in any text source. Everything about SoC-side GPIO numbers below tells you WHICH SIGNALS EXIST on each cable, not which connector pin they are on.

---

## 1. Display panel (7", 1024x600): part, interface, backlight, init sequence, connector

### Takeaway
The panel is a 4-lane MIPI DSI video-mode LCD, natively portrait 600x1024, with two possible vendors (BOE TV070WSM or K&D KD070D82) and two possible driver ICs (Fitipower JD9364 or JD9365), identified at boot by two ID pins on the display module. Timings, lane count and init sequences are published in Fuchsia; backlight is a TI LP8556 on I2C 0x2C. Connector type, pitch, pin count and pinout are not publicly documented in text form.

### Cited Findings
- Fuchsia selects the Nelson panel from two GPIO ID pins on the display module: ID1 = vendor (0 = K&D, 1 = BOE), ID0 = DDIC (0 = Fitipower JD9364, 1 = Fitipower JD9365). Resulting panel types: `KD_KD070D82_FITIPOWER_JD9364`, `KD_KD070D82_FITIPOWER_JD9365`, `BOE_TV070WSM_FITIPOWER_JD9364_NELSON`, `BOE_TV070WSM_FITIPOWER_JD9365`. — [Fuchsia post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/post-init/post-init.cc)
- SoC pins for those ID lines: `GPIO_DISP_SOC_ID0` = GPIOZ(11), `GPIO_DISP_SOC_ID1` = GPIOZ(12); panel reset `GPIO_SOC_DISP_RST_L` = GPIOZ(13) (active low by name); backlight enable `GPIO_SOC_BKL_EN` = GPIOA(10). — [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h)
- All Nelson panel configs set `dphy_data_lane_count = 4`. — [Fuchsia panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- BOE TV070WSM timings used by Fuchsia: H active 600, HFP 40, HSYNC 24, HBP 36; V active 1024, VFP 19, VSYNC 2, VBP 8; pixel clock 44.226 MHz (60 Hz); hsync/vsync negative; max D-PHY clock-lane frequency set to 180 MHz. Source comments cite the "BOE TV070WSM-TG1 spec" (panel supports MIPI clock 145-250 MHz; datasheet typical pixel clock 42.4 MHz, typical HFP 20, VFP 6). — [Fuchsia panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- K&D KD070D82 timings ("K&D KD070D82-39TI-A010" datasheet typicals): H active 600, HFP 80, HSYNC 10, HBP 80; V active 1024, VFP 20, VSYNC 6, VBP 20; pixel clock 49.434 MHz (60 Hz); max D-PHY clock-lane 200 MHz. — [Fuchsia panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- JD9364/JD9365 DDIC: per-lane data rate 80-500 Mbps for 4-lane 24-bit pixels (D-PHY clock 40-250 MHz), per datasheet references quoted in the Fuchsia comments. — [Fuchsia panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- Init sequences ARE published: DSI-on tables are named `lcd_init_sequence_BOE_TV070WSM_FITIPOWER_JD9364_NELSON`, `..._BOE_TV070WSM_FITIPOWER_JD9365`, `..._KD_KD070D82_FITIPOWER_JD9364`, `..._KD_KD070D82_FITIPOWER_JD9365`, in headers under `src/graphics/display/drivers/amlogic-display/panel/` (e.g. `boe-tv070wsm-fitipower-jd9364-nelson.h`, `kd-kd070d82-fitipower-jd9365.h`); power sequencing is `kLcdPowerOnSequenceForAstroSherlockNelson` in `initcodes-inl.h`. I read only the index file, not the byte tables. — [Fuchsia panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc)
- A Dec-2021 snapshot of the Fuchsia touch driver lists six bootloader panel IDs for Nelson: 1 `kd_fiti9364`, 2 `boe_fiti9364`, 3 `inx_fiti9364`, 4 `kd_fiti9365`, 5 `boe_fiti9365`, 6 `boe_sit7703` — i.e. an Innolux variant and a BOE panel with a Sitronix ST7703-family DDIC were at least planned. — [Fuchsia gt6853-nelson.cc, 2021-12 snapshot](https://go-mod-viewer.appspot.com/go.fuchsia.dev/fuchsia@v0.0.0-20211206140801-bbc9ce36f629/src/ui/input/drivers/gt6853/gt6853-nelson.cc); current main only maps the four KD/BOE x JD9364/JD9365 combinations — [Fuchsia post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/post-init/post-init.cc)
- Backlight driver IC: TI LP8556SQ-E09 ("LED Driver IC 6 Output with Step-Up DC DC Regulator"). — [Hackaday.io components](https://hackaday.io/project/190147/components)
- Backlight is controlled over I2C at address 0x2C on the "AV" I2C bus (S905D3 I2C3; SDA = GPIOA(14), SCL = GPIOA(15)). — [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h); [Fuchsia nelson-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc)
- TechInsights attributes the "60 Hz display/touchscreen subsystem" in their unit to BOE (est. cost $29.06), 1024x600. — [E360/TechInsights, 2021-08-11](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)

### Inferences
- A replacement board must (a) read the two ID pins (or try each init table) because any given unit may have a BOE or K&D glass with either DDIC, and (b) drive the display as 600x1024 portrait and rotate in software/GPU — the "1024x600" marketing resolution is landscape use of a portrait panel.
- 4 lanes at ~44-49 MHz pixel clock, 24 bpp implies roughly 265-300 Mbps per lane, comfortably within common SoC/bridge DSI capabilities (computed from the Fuchsia timings; not stated in a source).
- LP8556 is listed among mainboard-side parts in the Hackaday list and is addressed on the same I2C bus as the audio amp (which is on the mainboard), so the boost LED driver is most likely ON THE MAINBOARD and the display FPC carries raw LED anode/cathode strings. This is inference; it means a replacement board needs its own LED boost driver (re-using LP8556 with the same config is the low-risk choice).
- The "-E09" suffix denotes a specific factory EPROM preset of the LP8556 (boost voltage/current defaults); stock firmware may also override registers over I2C.

### Gaps
- Display FPC connector type, pitch, pin count and pinout: not found in any text source. Would need FCC internal photos (not opened) plus continuity measurement on a real unit.
- LED string count, forward voltage and current: not found. Measure boost output voltage and the LP8556 current-set resistor/registers on a working unit, or look for the BOE TV070WSM-TG1 / K&D KD070D82-39TI-A010 datasheets (cited by Fuchsia but I did not locate public copies; panelook was not checked).
- Panel supply rails (VDD, VGH/VGL generation on glass vs on board) and the power-on sequence timings: in `initcodes-inl.h` (not read).
- Whether the BOE SIT7703 / Innolux variants ever shipped at retail: unknown.
- No mainline Linux panel driver naming these exact modules was searched for or found. (Mainline has a generic `panel-jadard-jd9365da-h3` driver for the JD9365 family; I did not verify its applicability — treat as a lead, not a fact.)

---

## 2. Touch panel: controller, location, bus/address, connector

### Takeaway
Touch controller is a Goodix GT6853 on a dedicated I2C bus at address 0x5D with interrupt and reset lines; it needs a panel-specific config blob that differs per DDIC type. Fuchsia has an open driver; a Focaltech controller at 0x38 is also declared (earlier builds/alternate).

### Cited Findings
- Touch controller: "Goodix GT6853 Touch Screen Controller". — [Hackaday.io components](https://hackaday.io/project/190147/components)
- I2C addresses: Goodix 0x5D, Focaltech 0x38, both declared on the touch I2C bus (S905D3 I2C2; SDA = GPIOZ(14), SCL = GPIOZ(15)). — [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h); [Fuchsia nelson-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc)
- Touch interrupt `GPIO_TOUCH_SOC_INT_L` = GPIOZ(4); touch reset `GPIO_SOC_TOUCH_RST_L` = GPIOZ(9) (both active-low by name). — [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h)
- The GT6853 driver loads one of three config files chosen by panel type (9364, 9365, 7703 DDIC families); "a config file may contain multiple configs; the correct one is chosen based on the sensor ID reported by the touch controller". The Goodix driver "isn't used on boards earlier than P2". — [Fuchsia gt6853-nelson.cc, 2021-12 snapshot](https://go-mod-viewer.appspot.com/go.fuchsia.dev/fuchsia@v0.0.0-20211206140801-bbc9ce36f629/src/ui/input/drivers/gt6853/gt6853-nelson.cc)
- Open driver source lives at `src/ui/input/drivers/gt6853/` in Fuchsia (BSD). — [Fuchsia gt6853-nelson.cc snapshot](https://go-mod-viewer.appspot.com/go.fuchsia.dev/fuchsia@v0.0.0-20211206140801-bbc9ce36f629/src/ui/input/drivers/gt6853/gt6853-nelson.cc)
- Goodix's developer forum has a thread about porting a "goodix_gt6853" Linux driver to an NXP i.MX 5.4 kernel using Goodix's porting guide, i.e. a vendor Linux driver exists outside mainline. (Search-result snippet only; page not read.) — [Goodix developer forum](https://developers.goodix.com/en/bbs/detail/19f255b9cee04631a2b8c480f918b9f7)

### Inferences
- Retail units use the Goodix GT6853; the Focaltech entry is most likely a holdover from pre-P2 prototypes (the Goodix driver comment says it is only for P2+).
- The controller IC is probably on the touch/display flex (chip-on-flex), since the board driver only exposes I2C + INT + RST to it and TechInsights treats display+touch as one BOE subsystem. Not confirmed by any source.
- The controller may already hold a valid config in its flash from the factory (Fuchsia comment: "There's a chance we can proceed without a config"), so a replacement board might get working touch without the proprietary config blob — needs testing.
- GT6853 is NOT handled by the mainline Linux `goodix.c` (GT9xx) driver; it belongs to Goodix's newer "Berlin"/Brl protocol family. Whether mainline `goodix_berlin` works with it is unverified — a lead only.

### Gaps
- Touch connector (separate FPC vs combined with display FPC), pin count, pitch, pinout: not documented.
- I/O voltage for touch I2C/INT/RST (1.8 V vs 3.3 V) and touch supply rails: not documented; measure.
- The config blobs (`GT6853_CONFIG_9364/9365/7703_PATH`) are firmware files not present in the open tree as far as I could see; not verified.

---

## 3. Soli radar: chip, location, interface, public docs/drivers

### Takeaway
Soli is an Infineon BGT60TR13C on its own small "gesture board" with an 80 MHz oscillator and 1.8 V LDO, connected to the mainboard via SPI (stock clocked at ~30 MHz) plus IRQ and reset. It is the best-documented reusable peripheral: Infineon publishes an MIT-style driver library on GitHub, the stock register configuration has been captured and published, and the datasheet is obtainable via Infineon's Radar Development Kit.

### Cited Findings
- Chip: BGT60TR13C, "60GHz Radar sensor with integrated one transmitting and three receiving antennas". Supporting parts listed with it: LP5907SNX-1.8 (1.8 V low-noise LDO, 250 mA), NSC5103A-80M (1.8 V CMOS 80 MHz oscillator), TPS7A2501DRVR (adjustable 300 mA LDO). — [Hackaday.io components](https://hackaday.io/project/190147/components)
- It sits on a separate "gesture sensor board" that "includes a 60 GHz radar module from Infineon and a 1.9 V LDO regulator from TI". — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Fuchsia declares it as SPI device `PDEV_PID_INFINEON_BGT60TR13C` ("Radar sensor head") on SPICC1, CS index 0, with client-managed CS; SCLK = core clock / 22 = 30.3 MHz; MISO sampled with a 3-cycle input delay. — [Fuchsia nelson-spi.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-spi.cc)
- Radar signal set at the SoC ("SELINA" is the internal name): `SPI_B_MOSI` GPIOH(4), `SPI_B_MISO` GPIOH(5), `SPI_B_SS0` GPIOH(6), `SPI_B_SCLK` GPIOH(7), `SELINA_RESET` GPIOH(2), `SELINA_IRQ_OUT` GPIOH(3), `SELINA_OSC_EN` GPIOH(8). — [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h); IRQ/reset/CS binding confirmed in [Fuchsia post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/post-init/post-init.cc)
- `SELINA_OSC_EN` is only actively handled on board build P1 (prototype), where it is set to a no-pull input. — [Fuchsia post-init.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/post-init/post-init.cc)
- Operating behaviour captured from a real unit: 80 MHz clock; chip fills its FIFO with raw ADC data, raises an interrupt, and the SoC reads it with the SPI "burst mode" command; ~30 bursts per second; each burst returns a ~180 kbit frame of 24-bit blocks; `FIFO_CREF` = 7739; `PREFIX_EN` = 1. The full list of ~38 configuration register writes (addresses 0x00-0x56 with 24-bit values) is published in the log, plus DSLogic capture files `Config_register.dsl` and `FIFO_read_loop.dsl`. — [Hackaday.io project, log dated 2023-03-26](https://hackaday.io/project/190147-google-nest-hub-2nd-gen-teardown)
- A "BGT60TR13C Radar PCB pinout" log exists (2023-03-21) but consists only of two photos with no text; the pinout is in the images: `https://cdn.hackaday.io/images/6514751679376803135.jpg` and `https://cdn.hackaday.io/images/1670461679376915403.jpg` (not viewed by me). — [Hackaday.io pinout log](https://hackaday.io/project/190147/log/216912-bgt60tr13c-radar-pcb-pinout)
- The Hackaday author notes most of the written registers are not described in the datasheet he had (only 0x06 SFCTL was). Two datasheet PDFs are attached to the project (v01 uploaded 2023-03-27; V2.4.9 uploaded 2025-11-16). — [Hackaday.io project](https://hackaday.io/project/190147-google-nest-hub-2nd-gen-teardown)
- Infineon publishes an open driver library, `sensor-xensiv-bgt60trxx`, usable with "user-provided communication functions" (not tied to ModusToolbox HAL); it needs SPI (example uses mode 0, MSB first, 25 MHz), CS, an IRQ pin, a reset pin, and an LDO enable; init takes a register list generated by the `bgt60-configurator-cli` tool; the BGT60TR13C datasheet (V2.4.6) and the configurator ship in Infineon's Radar Development Kit via the Infineon Developer Center. — [Infineon/sensor-xensiv-bgt60trxx](https://github.com/Infineon/sensor-xensiv-bgt60trxx)
- Further open code: Infineon `xensiv-radar-presence` (presence detection library for BGT60TR13C) and `sensor-dsp`. — [Infineon/mtb-example-psoc6-radar-presence README](https://github.com/Infineon/mtb-example-psoc6-radar-presence); a third-party async no_std Rust crate `bgt60trxx` also supports BGT60TR13C (snippet only) — [docs.rs/bgt60trxx](https://docs.rs/bgt60trxx)
- Hobbyists have driven BGT60TR13C from nRF52833 and ESP32-C3 custom boards (forum thread titles/snippets only). — [Infineon community 1](https://community.infineon.com/t5/Radar-sensors/BGT60TR13C/td-p/883939); [Infineon community 2](https://community.infineon.com/t5/Radar-sensors/BGT60TR13C-minimum-register-list/td-p/999623)
- FCC grant covers 58-63.5 GHz operation "under waiver of section 15.255(c)(3), DA 18-1308". — [fccid.io A4RGUIK2](https://fccid.io/A4RGUIK2)

### Inferences
- The radar board is electrically simple for a replacement mainboard: 4-wire SPI + IRQ + RESET (+ possibly an oscillator/LDO enable), 1.8 V logic (the oscillator and LDO are 1.8 V parts; BGT60TR13C is a 1.8 V device). Level-shift if the new SoC uses 3.3 V I/O.
- Whether the radar board is powered from a mainboard rail that feeds the on-board LDOs (likely 3.3 V in) or receives 1.8 V directly is not documented.
- Raw FMCW data is easy to obtain; Google's gesture/sleep-sensing algorithms are proprietary, so a custom board gets presence/motion/range via Infineon's libraries, not "Quick Gestures" or Sleep Sensing.
- Operating the radar with a non-stock configuration may fall outside the FCC waiver conditions the product was certified under; a regulatory consideration, not a technical blocker.

### Gaps
- Radar flex connector type/pin count/pinout in text form: not available (only the Hackaday photos).
- I did not confirm whether Fuchsia's actual radar driver (the thing that binds to "selina-composite") is in the open tree; I only saw the board-level binding.

---

## 4. Microphones

### Takeaway
Three PDM MEMS microphones share one PDM clock; two are multiplexed on data line DIN0 and the third is on DIN1. At least two are on a separate sensor board, so they are reusable with any SoC/codec that has two PDM data inputs.

### Cited Findings
- Fuchsia PDM config: `number_of_channels = 3`; pins `MIC_DCLK` GPIOA(7), `MICLR_DIN0` GPIOA(8) ("First 2 MICs"), `MICLR_DIN1` GPIOA(9) ("Third MIC"); `sysClockDivFactor = 4`, `dClockDivFactor = 250`, with HIFI PLL set to 768 MHz. — [Fuchsia nelson-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc); [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h)
- Part: 3x "SD18OB371-077 PDM Mems Microphone". — [Hackaday.io components](https://hackaday.io/project/190147/components)
- The separate sensor board carries "Two MEMS microphones from Goertek" plus the Sensirion humidity/temperature sensor. — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Official spec: "3 far-field microphones", "Mic off switch". — [Google support specs](https://support.google.com/googlehome/answer/7072284?hl=en)

### Inferences
- PDM clock is probably 768 MHz / 250 = 3.072 MHz (my arithmetic from the Fuchsia divider; not stated explicitly).
- "MICLR" + "first 2 mics on DIN0" implies the standard L/R-select scheme: two mics on opposite clock edges of DIN0, the third alone on DIN1.
- TechInsights counting only two mics on the sensor board, against three in total, suggests the third mic is elsewhere (mainboard or another flex). If it is soldered to the mainboard it is lost on board replacement — needs physical confirmation.
- The hardware mic-mute switch likely cuts mic power or clock in hardware in addition to signalling the SoC (typical Google design, and `GPIO_MUTE_SOC` is only a sense input). Not confirmed; if true, the mute circuit's location (mainboard vs sensor board) matters for a redesign.

### Gaps
- Physical location of the third microphone; mic supply voltage; sensor-board connector/pinout: not documented.
- No public datasheet located for "SD18OB371-077" (not searched specifically).

---

## 5. Speaker and amplifier

### Takeaway
Single full-range 43.5 mm (1.7") driver, driven mono by a TI TAS5805M class-D amp in bridged (PBTL) mode over I2S, amp at I2C 0x2D. Speaker impedance, rated power and connector are not publicly documented.

### Cited Findings
- "Full-range speaker with 1.7 in (43.5 mm) driver." — [Google support specs](https://support.google.com/googlehome/answer/7072284?hl=en)
- Amplifier: TAS5805M ("23-W, Inductor-Less, Digital Input, Stereo, Closed-Loop Class-D"). — [Hackaday.io components](https://hackaday.io/project/190147/components); TechInsights also lists a TI audio power amplifier on the main board — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen); datasheet — [TI TAS5805M](https://www.ti.com/lit/ds/symlink/tas5805m.pdf)
- Fuchsia config: codec type Tas58xx with metadata `.bridged = true`; I2S on TDM-B (SCLK GPIOA(1), FS GPIOA(2), DO0 GPIOA(3)); 16-bit samples in 32-bit slots; single mono channel placed in the left slot; amp `PDN_N` = `GPIO_SOC_AUDIO_EN` GPIOA(5); fault = `GPIO_AUDIO_SOC_FAULT_L` GPIOA(12); a boost/inrush enable `GPIO_INRUSH_EN_SOC` GPIOA(0) (commented "BOOST_EN_SOC") is driven high before amp power-up. — [Fuchsia nelson-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc)
- Amp I2C address 0x2D on I2C3 (shared with backlight). — [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h)
- The stock DSP/EQ tuning is an init register sequence pulled from a non-public header (`TAS5805M_CONFIG_PATH`, `tas5805m_init_sequence1/2`). — [Fuchsia nelson-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc)
- Two INA231 current/power monitors: one on the main-logic-board supply (0x49; 0x46 on prototypes) and one on the speaker/audio supply (0x40), with an alert line `GPIO_ALERT_PWR_L` GPIOZ(10). — [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h); [Fuchsia nelson-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc); [Hackaday.io components](https://hackaday.io/project/190147/components) (2x INA231)
- Speaker estimated at $2.56 in the BOM, quantity 1. — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)

### Inferences
- The speaker is wired as a single 2-wire load across a PBTL bridge; any mono class-D amp of suitable power can replace the TAS5805M. Reusing a TAS5805M in PBTL is the closest match, but the stock EQ/DRC tuning (which protects a small driver) is not public — start with conservative gain and a high-pass filter.
- The amp is likely fed from the ~14 V input rail (TAS5805M PVDD range covers it), with the "inrush/boost enable" gating that rail. Not confirmed.

### Gaps
- Speaker impedance, power rating, connector type: not documented; measure DC resistance and read the driver label.
- The stock speaker protection/EQ coefficients are not public.

---

## 6. Sensors (ambient light/colour, temperature, others)

### Takeaway
Ambient light/colour is an AMS TCS3400 (I2C 0x39) and temperature/humidity is a Sensirion SHTC3-class part (I2C 0x70), both on the always-on "sensors" I2C bus; the Sensirion part is on the separate sensor board. There are also board thermistors on the SoC ADC.

### Cited Findings
- "TCS3400 Color Light-to-Digital Converter. Red, Green, Blue, Clear and IR light sensing" and "SHTC3 Humidity and Temperature Sensor". — [Hackaday.io components](https://hackaday.io/project/190147/components)
- Fuchsia: ambient light sensor at 0x39 (`tcs3400_light`, gain 64, integration time 711.68 ms, polled every 700 ms) with interrupt `GPIO_RGB_SOC_INT_L` = GPIOAO(5); Sensirion "SHTV3" temperature device at 0x70; both on I2C_AO_0 (SCL GPIOAO(2), SDA GPIOAO(3)). — [Fuchsia nelson-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-light.cc); [Fuchsia nelson-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc); [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h)
- Sensor board contains "Humidity and temperature sensor from Sensirion" and two MEMS mics. — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Official specs list "Temperature sensor", "Ultrasound sensing", Ambient EQ light sensor and Soli. — [Google support specs](https://support.google.com/googlehome/answer/7072284?hl=en); [Android Authority review, updated May 2023](https://www.androidauthority.com/google-nest-hub-second-generation-review-1210216/)
- Board driver includes `nelson-thermistor.cc`, `nelson-adc.cc` and `nelson-thermal.cc` (thermistors via SoC SAR ADC; files not read). — [Fuchsia nelson directory](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/)

### Inferences
- Hackaday (SHTC3) vs Fuchsia ("SHTV3", DID `SENSIRION_SHTV3`): same I2C address 0x70 and same vendor; likely the same or a register-compatible part. Treat as SHTC3-compatible but verify the ID register.
- "Ultrasound sensing" uses the existing speaker and mics — no extra hardware.
- TCS3400 physical location (sensor board vs display bezel flex) is not stated in any source read; in the first-gen Hub the ALS was on the mic/sensor board (EDN first-gen teardown, different model), so the same is plausible here.

### Gaps
- Exact location of the TCS3400 and thermistors; sensor-board connector pinout and supply voltage (likely 1.8 V on the AO bus, unverified).

---

## 7. Buttons / switches

### Takeaway
Four simple GPIO inputs: volume up, volume down (active low, external pull-ups), a mic-mute slide switch sense line (active high, polled), and a factory-reset line; plus one PWM-driven amber LED.

### Cited Findings
- `GPIO_VOL_UP_L` = GPIOZ(5), `GPIO_VOL_DN_L` = GPIOZ(6) (interrupt-driven, inverted/active-low); `GPIO_MUTE_SOC` = GPIOZ(2) (polled every 20 ms, not inverted, no pull); `GPIO_FDR_L` = GPIOAO(10) (polled, inverted, no pull; bound under the name "volume-both"). Comment: "No need for internal pull, external pull-ups used." — [Fuchsia nelson-buttons.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-buttons.cc); [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h)
- Amber LED: `GPIO_AMBER_LED_PWM` = GPIOAO(11), driven by PWM_AO_A, brightness-controllable, single colour. — [Fuchsia nelson-light.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-light.cc)
- Physical mute switch and volume keys are on the back of the device. — [Android Authority review](https://www.androidauthority.com/google-nest-hub-second-generation-review-1210216/)

### Inferences
- `FDR_L` named "volume-both" suggests a hardware AND of both volume keys generates the factory-data-reset signal; a replacement board can ignore it.
- The amber LED is the mic-muted indicator (standard Nest behaviour); not stated in the source.

### Gaps
- Whether the buttons/switch are on the mainboard itself or on a daughter flex: not documented. If they are board-mounted tact switches, the replacement PCB must place equivalents at identical positions — check FCC photos / a real unit.

---

## 8. Power input

### Takeaway
Official rating is a 15 W adapter with a DC barrel jack; the exact voltage/current and plug dimensions for the 2nd-gen were not confirmed in a primary source during this research.

### Cited Findings
- "15 W power adapter"; ports: "DC power jack"; power cable 1.5 m. — [Google support specs](https://support.google.com/googlehome/answer/7072284?hl=en); [Android Authority review](https://www.androidauthority.com/google-nest-hub-second-generation-review-1210216/)
- The FIRST-gen Home Hub adapter was 14 V / 1.1 A (~15 W) with a barrel connector (different model, for context only). — [EDN first-gen Home Hub teardown](https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/)
- Third-party replacement listings sell "14V 1.1A 15W" adapters described as compatible with "Google Home Hub / Nest Hub / Nest Mini (2nd)". Low-reliability marketplace sources; none specifically names the 2nd-gen Hub's own adapter model. — [Amazon listing](https://www.amazon.com/Dysead-Replacement-Supply-Adapter-Compatible/dp/B0D6LYXJ17); [Shopee listing](https://shopee.sg/14V-1.1A-15W-AC-DC-Power-Supply-Charger-For-Google-Home-Nest-Hub-Mini-Gen-2-2nd-W18-015N1A-G1015-G1030-Smart-speaker-i.860304842.29710786136)
- On-board power: "Slap-down DC-DC converters from Silergy"; 2x SSM6J507NU P-channel MOSFETs (30 V/10 A). — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen); [Hackaday.io components](https://hackaday.io/project/190147/components)
- SoC core rails are PWM-controlled regulators (`SOC_VDDEE_PWM` GPIOE(0), `SOC_VDDCPU_PWM` GPIOE(1)); a brownout-protection init exists. — [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h); [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h)

### Inferences
- 14 V nominal at roughly 1.1 A is the most likely input (15 W rating + same connector family as first gen), so a replacement board should budget ~15 W total and accept ~14 V in; design the input stage for at least 12-16 V tolerance until measured.
- The barrel jack is very likely soldered to the mainboard (or a small jack board) — its footprint/position is a mechanical constraint for a drop-in board.

### Gaps
- Exact adapter label (V/A, model number) for GUIK2 and barrel plug OD/ID/polarity: NOT confirmed. Read the adapter label and measure the plug with calipers. (Commonly quoted third-party figure for Google 14 V plugs is about 3.5 x 1.35 mm, centre positive — unsourced here; verify.)
- Whether there is a hidden USB/service port on the 2nd gen (first gen had micro-USB under the foot per EDN): Fuchsia has `nelson-usb.cc`, so USB exists at SoC level, but physical access was not confirmed.

---

## 9. Mechanical: disassembly, board outline, mounting, thermal

### Takeaway
Outer dimensions and weight are published; internal modules connect by flat-flex and unplug cleanly per teardown commentary. No published mainboard outline, hole pattern or step-by-step iFixit guide for the 2nd gen was found; the FCC internal photos and the txyz.info teardown video are the best visual references.

### Cited Findings
- Dimensions 177.4 x 120.4 x 69.5 mm; weight 558 g. — [Android Authority review](https://www.androidauthority.com/google-nest-hub-second-generation-review-1210216/)
- FCC exhibits include "Internal Photos" (PDF 1.2 MB, released 2021-07-03) and External Photos; Schematics, Block Diagram and Theory of Operation are under long-term confidentiality (metadata only). — [fccid.io A4RGUIK2](https://fccid.io/A4RGUIK2); internal photos at [fccid.io internal photos](https://fccid.io/A4RGUIK2/Internal-Photos/Internal-Photos-20200702-v1-Internal-Photos-5035982) (not opened)
- A video teardown exists: "Google Nest Hub (2nd Gen) Teardown" by txyzinfo. — [YouTube](https://www.youtube.com/watch?v=CcPzFpDd2_Q); covered by [Hackaday, 2023-05-29](https://hackaday.com/2023/05/29/google-nest-hub-teardown/)
- A Hackaday commenter (not the author) observes the unit "is easily separated into its component parts by simply unplugging all the flat-flex"; another notes opening risks broken clips. Reader comments, low authority. — [Hackaday, 2023-05-29](https://hackaday.com/2023/05/29/google-nest-hub-teardown/)
- TechInsights identifies separate assemblies: main board, gesture (radar) board, sensor board, display/touch subsystem, speaker, an "internal frame" ($3.52) and back enclosure. — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Debug access at the SoC: UART-AO TX/RX on GPIOAO(0)/(1), JTAG on GPIOAO(6-9). The Hackaday author used "debug cables" to capture the radar SPI. — [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h); [Hackaday, 2023-05-29](https://hackaday.com/2023/05/29/google-nest-hub-teardown/)

### Inferences
- The modular flex-connected construction (radar board, sensor board, display, speaker lead) is favourable for a replacement mainboard: at minimum four cable interfaces (display+backlight, touch, radar, sensor/mic board) plus speaker wires and DC input.
- Thermal: stock SoC is a ~12 nm quad A55 dissipating a few watts inside a sealed fabric/plastic shell with a metal frame/heat spreader (first gen used a heatsink + thermal tape per EDN). A hotter replacement SoC (e.g. a Pi CM-class module) would need the frame coupled as a heat spreader; total input budget is 15 W shared with backlight and amp.

### Gaps
- No iFixit guide/teardown for the 2nd-gen Hub was found in search (iFixit result returned was for Nest Mini). Disassembly steps (screen adhesive vs clips, screw types) not documented in text — watch the video.
- Mainboard outline, dimensions, mounting-hole positions, connector positions, component height limits: not published. Must be measured/scanned from a real board.

---

## 10. Reference only: stock compute and what is lost with the mainboard

### Takeaway
Stock platform is Amlogic S905D3G (quad Cortex-A55, with NPU), 2 GB DDR3, 4 GB eMMC, Broadcom BCM43458 Wi-Fi/BT over SDIO/UART, and an nRF52811 + SKY66408 802.15.4/Thread radio. All of these, plus the audio amp, backlight driver, power monitors and (probably) antennas' feed circuitry, are on the mainboard and go away with it.

### Cited Findings
- SoC: Amlogic S905D3G, 4x Cortex-A55, 1.9 GHz. — [Hackaday.io components](https://hackaday.io/project/190147/components); [Hackaday, 2023-05-29](https://hackaday.com/2023/05/29/google-nest-hub-teardown/); Fuchsia uses `AmlVersion::kS905D3G` and has `nelson-nna.cc` (neural accelerator) — [Fuchsia nelson-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc). Contradicted in naming by TechInsights' summary, which says "Amlogic S905D" (while still describing Cortex-A55) — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen). S905D3G is the better-supported identification.
- RAM: 4x Samsung K4B4G1646E-BYMA DDR3 4 Gbit (= 2 GB total). — [Hackaday.io components](https://hackaday.io/project/190147/components); "2 GB of DDR3 SDRAM" from Samsung — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Storage: Samsung KLM4G1FETE-B041 eMMC 5.1. Hackaday's list describes it as "4Gbit"; TechInsights says "4 GB MLC NAND flash, memory controller – Samsung". 4 GB is correct for that part family; the Hackaday "Gbit" is evidently a typo. — [Hackaday.io components](https://hackaday.io/project/190147/components); [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Wi-Fi/BT: Broadcom BCM43458 (802.11ac + BT) on SDIO (GPIOX0-5) with BT on UART (GPIOX12-15) and PCM (GPIOX8-11). — [Hackaday.io components](https://hackaday.io/project/190147/components); [Fuchsia nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h); TechInsights: "Wi-Fi 802.11 a/b/g/n/ac, Bluetooth 5.0 – Broadcom" — [E360/TechInsights](https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen)
- Thread/802.15.4: Nordic nRF52811 on SPICC0 at 1 MHz (GPIOC0-3) with reset/boot/interrupt lines, plus Skyworks SKY66408-11 2.4 GHz front end. — [Fuchsia nelson-spi.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-spi.cc); [Hackaday.io components](https://hackaday.io/project/190147/components)
- FCC-certified radios: 2.4 GHz and 5 GHz WLAN, BT/BLE, 802.15.4 (Thread/Zigbee reports), and 58-63.5 GHz radar. — [fccid.io A4RGUIK2](https://fccid.io/A4RGUIK2)
- Board revisions known to Fuchsia: P1, P2, P2_DOE, PRE_EVT, EVT, DVT, DVT2, read from 5 HW_ID strap pins. — [Fuchsia nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h)

### Inferences
- Lost when the mainboard is replaced (must be re-implemented): SoC/RAM/eMMC, Wi-Fi/BT module and Thread radio, TAS5805M amp, LP8556 backlight boost, INA231 monitors, DC input and regulators, and any board-mounted buttons/LED/third mic.
- Retained (off-board): LCD + touch assembly, radar board, sensor board (2 mics + temp/humidity, possibly ALS), speaker.
- Antennas: whether Wi-Fi/BT/Thread antennas are PCB traces on the mainboard or separate flex/frame antennas with coax was not determined; this strongly affects radio design on a replacement board.

### Gaps
- Antenna construction and connectors; exact mainboard-vs-daughterboard placement of ALS, third mic, buttons, LED: need the FCC internal photos or a physical unit.

---

## 11. Public driver sources and bus map (consolidated)

### Takeaway
The Fuchsia "nelson" board driver is a near-complete public map of buses, addresses and control lines for every peripheral; it is the single most valuable source for a replacement-board design. No mainline Linux device tree for this product was found.

### Cited Findings
Consolidated bus map (all from the Fuchsia files cited above: [nelson.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h), [nelson-i2c.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc), [nelson-gpios.h](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h), [nelson-spi.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-spi.cc), [nelson-audio.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc)):

| Peripheral | Part | Bus (stock SoC pins) | Address / rate | Sideband signals |
|---|---|---|---|---|
| Ambient light/colour | AMS TCS3400 | I2C_AO_0 "SENSORS" (SCL AO2, SDA AO3) | 0x39 | INT_L = AO5 |
| Temp/humidity | Sensirion SHTC3 / "SHTV3" | I2C_AO_0 | 0x70 | — |
| Touch | Goodix GT6853 (alt. Focaltech) | I2C2 "TOUCH" (SDA Z14, SCL Z15) | 0x5D (alt. 0x38) | INT_L = Z4, RST_L = Z9 |
| Backlight | TI LP8556 | I2C3 "AV" (SDA A14, SCL A15) | 0x2C | BKL_EN = A10 |
| Audio amp | TI TAS5805M (PBTL mono) | I2C3 + I2S TDM-B (SCLK A1, FS A2, DO A3) | 0x2D | PDN_N = A5, FAULT_L = A12, boost/inrush EN = A0 |
| Power monitors | 2x TI INA231 | I2C3 | 0x49 (board), 0x40 (speaker) | ALERT_L = Z10 |
| Display | BOE TV070WSM / K&D KD070D82, JD9364/JD9365 | MIPI DSI, 4 data lanes | 600x1024@60 | RST_L = Z13, ID0 = Z11, ID1 = Z12 |
| Radar | Infineon BGT60TR13C | SPICC1 (MOSI H4, MISO H5, CS H6, SCLK H7) | 30.3 MHz | IRQ = H3, RESET = H2, OSC_EN = H8 |
| Microphones | 3x PDM MEMS | PDM (DCLK A7, DIN0 A8 = 2 mics, DIN1 A9 = 1 mic) | — | mute sense = Z2 |
| Buttons | — | GPIO | — | VOL_UP_L Z5, VOL_DN_L Z6, FDR_L AO10 |
| LED | amber | PWM_AO_A on AO11 | — | — |
| Thread radio (on mainboard) | nRF52811 | SPICC0 (C0-C3) | 1 MHz | INT C5/C7, RST_L A13, BOOT_MODE_L Z1 |

- Other open code: Fuchsia `src/ui/input/drivers/gt6853/` (touch) — [snapshot](https://go-mod-viewer.appspot.com/go.fuchsia.dev/fuchsia@v0.0.0-20211206140801-bbc9ce36f629/src/ui/input/drivers/gt6853/gt6853-nelson.cc); Fuchsia `src/graphics/display/drivers/amlogic-display/panel/*.h` (panel init tables) — [panel-config.cc](https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc); Infineon radar driver — [sensor-xensiv-bgt60trxx](https://github.com/Infineon/sensor-xensiv-bgt60trxx).

### Inferences
- I2C bus voltages are not stated; the AO-domain sensor bus on Amlogic SM1 designs is commonly 1.8 V and GPIOZ/GPIOA banks are commonly 1.8 V or 3.3 V depending on the VDDIO strap — every interface needs its logic level measured before connecting a new SoC.
- Because each off-board peripheral is on standard interfaces (DSI, I2C, SPI, PDM, I2S-fed amp), the functional design of a replacement board is straightforward; the unknowns are entirely physical (connectors, pinouts, voltages, outline).

### Gaps
- No Linux device tree (mainline or vendor) for "nelson"/GUIK2 was found; Google's open-source release tarballs for Nest Hub firmware (which could contain a kernel DTS) were not searched.
- Not found/verified this session: iFixit teardown for this model, panelook entries, GitHub hardware-reuse projects, Reddit threads with pinouts. The one Reddit thread surfaced (r/hardwarehacking, "Nest Hub (2nd Gen) Teardown and Radar SPI decoding") is the same author as the Hackaday project and was not opened.

---

## Summary of what must be measured on a real unit (nothing public found)

1. Every FPC/FFC connector: type, pitch, pin count, pinout (display, touch, radar board, sensor board), and the speaker and DC connectors.
2. Logic levels and supply rails for each daughterboard (radar board input rail; sensor board rail; touch and panel rails).
3. Backlight LED string configuration, voltage and current.
4. Speaker impedance and safe power.
5. Adapter voltage/current and barrel dimensions/polarity.
6. Mainboard outline, mounting holes, connector positions, height limits, heat-spreader contact areas, antenna arrangement.
7. Physical location of: third microphone, TCS3400, buttons/mute switch, amber LED, LP8556.
