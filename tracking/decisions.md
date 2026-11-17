# Decisions log

One entry per design decision. Change the status and fill in the outcome when a decision is made; do not delete the reasoning. Tracker rows refer to these by ID in their `blocked_by` column. Background for each is in `docs/Compute modules and board design.md`.

| ID | Decision | Status | Outcome | Date |
|---|---|---|---|---|
| D1 | Target model, and one board or several | Decided | 1st gen first. One superset schematic, one bench board for all three models, three form-fit layouts, one software build. | 2026-11-17 |
| D2 | Compute module | Open | | |
| D3 | Microphone path | Open | | |
| D4 | Amplifier | Open | | |
| D5 | Backlight driver (and panel bias supply on the Max) | Open | | |
| D6 | Radio and antennas | Open | | |
| D7 | Optional peripherals and thermal (radar, camera) | Open | | |
| D8 | Operating system and product scope | Decided | Linux running Hearth, with Home Assistant's Linux Voice Assistant for voice. No Raspberry Pi silicon. Sold as retrofit kits. | 2026-11-17 |

## D1. Target model

- **Decided:** start with the 1st gen. It is the best documented and simplest: no radar, no camera, one amplifier, known 14 V supply.
- **Decided: board strategy.**
  - **One superset schematic.** Covers all three models, with parts fitted or left off per model: 14-24 V input, LP8556 set for 3 or 6 strings, panel bias supply (Max only), one or three amplifier channels, two PDM data inputs, radar SPI (2nd gen only), camera CSI-2 (Max only).
  - **Board ID straps.** A few resistor straps on each board tell software which model it is in.
  - **One bench board.** Drives all three models from outside the enclosure, through small per-model adapter flexes. Used for bring-up before any form-fit board.
  - **Three form-fit layouts.** One per model, from the same schematic, 1st gen first. One drop-in board for all three is ruled out: the stock outlines (about 121 x 46, 101 x 46 and 117 x 62 mm), mounting holes, connector positions, heat paths and input voltages (14 V vs 24 V) all differ.
  - **One software build.** One kernel and Linux image (Android until D8), with a device-tree overlay per model chosen at boot from the board ID straps.
- **Open:** whether the two 7-inch models can share one form-fit board. Depends on whether the module plus the superset parts fit the 2nd gen's 101 x 46 mm outline and height; needs both boards measured.
- **Open:** whether one amplifier part suits both the 14 V and the 24 V models (see D4).
- **2nd gen:** shares the panel family and bus layout with the 1st gen, and adds the radar, the GT6853 touch risk and a third microphone. Its stock board is about 101 x 46 mm against about 121 x 46 mm, so a shared outline is unknowable until both are measured.
- **Max:** gets its own board: 24 V, 10.1-inch panel with external bias rails, three amplifiers and a camera. It shares the electrical architecture (portrait DSI panel, I2C touch, PDM mics, TCS3400, LP8556) with the 7-inch models.

## D2. Compute module

- **Current shortlist (Linux, per D8):** RK3576 (Boardcon LGA3576 module) and i.MX 8M Plus (Ka-Ro QSXP module); fallback RK3566/RK3568. Comparison and reasoning in "Linux stack and chip selection" in `docs/Compute modules and board design.md`.
- **Bench board:** Radxa ROCK 4D (RK3576) ordered 2026-11-17, for the bench test plan in that doc. No i.MX 8M Plus board ordered yet (recommended: NXP FRDM-IMX8MPLUS).
- **Earlier options (Android plan, superseded by D8):** Raspberry Pi CM5 (newest community Android, panel needs a C driver entry, non-commercial licence); Radxa CM4 / CM5 (vendor Android 14, panel described in device tree); NXP i.MX 8M (current vendor Android, heaviest integration).
- **Blocks:** almost every driver row, the power rail list, the microphone path and the board outline.
- **Needs before deciding:** whether the candidate has a PDM input usable under Android; its kernel version (sets how much porting the light-sensor patch and any vendor touch driver need); its power draw against the enclosure's thermal limit.
- **2nd gen also needs:** two PDM data inputs (three channels); a spare SPI at 25-30 MHz plus two GPIOs for the radar (1.8 V logic, or level shifting); a kernel the out-of-tree Goodix GTX8 touch driver can be ported to.
- **Max also needs:** I2S with two data outputs (or TDM) for three amplifiers; and, if the camera is attempted, 2-lane CSI-2 at 1000 Mbit/s per lane with an ISP and dewarping usable under Android.

## D3. Microphone path

- **Options:** stock PDM microphones straight into the module with software echo cancellation; through an XMOS-class voice processor; replaced by an added array.
- **Depends on:** D2.
- **2nd gen:** three microphones (two on PDM data 0, one on data 1), but the count is disputed: TechInsights says two, and the stock kernel enables only two data lines. Whether a four-mic XVF3800-class front end suits the stock three-mic geometry was not researched.
- **Max:** same two-mic PDM arrangement as the 1st gen, but on two separate small boards beside the camera hole; the privacy switch covers microphones and camera together.

## D4. Amplifier

- **Options:** refit a TAS2770 (mainline driver, like-for-like); a simpler part such as a MAX98357A.
- **Needs before deciding:** speaker DC resistance; the stock chip marking; how speaker protection is done without Google's tuning.
- **2nd gen:** stock is a TAS5805M or TAS5825M (sources conflict) in bridged mono at 0x2D. The TAS5805M has a mainline driver, the TAS5825M only a vendor one. Also needs: the chip marking, and what the third wire at J7 carries.
- **Max:** three channels (woofer + two tweeters) on 24 V with a digital crossover at about 1.2 kHz, tweeters 6.4 dB below the woofer, woofer fed an L+R mono mix. Three TAS5720 is like-for-like and mainline. Full-range signal must never reach the tweeters. Also decide where the crossover runs (amplifier DSP or software).

## D5. Backlight driver

- **Options:** refit an LP8556 with Google's register values (proposed); a different driver once the LED strings are characterised.
- **Needs before deciding:** measured string voltage and current; which of the two published register sets retail units use.
- **2nd gen:** LP8556 confirmed on the mainboard beside J12, 3 strings. The `b4` register values are the same seven as the 1st gen Fuchsia set, but the string-current registers are not written, so the current is unknown until measured.
- **Max:** 6 strings (6S6P). Open whether 0xA3 = 0x32 is written (Fuchsia yes; kernel device tree has no entry). The panel also needs a ±5.5 V bias supply (`BIAS`; 0x3E on stock, TPS65132-class inferred), decided here with the backlight.

## D6. Radio and antennas

- **Options:** the module's own Wi-Fi with its own antenna; a radio wired to the stock antennas.
- **Needs before deciding:** antenna connector type.
- **2nd gen:** two U.FL-type coax jacks (J3 plus one), radios not mapped to jacks. The Thread radio (nRF52811) leaves with the mainboard; whether to restore it is part of this decision.
- **Max:** stock uses MHF4 antenna connectors. Also decide whether to restore Thread (stock Nordic radio on SPI with its own coax antenna).

## D7. Optional peripherals and thermal

- **For the 1st gen:** whether to fit temperature sensors, and whether the chosen module stays within what the stock heat sink can shed.
- **For the 2nd gen:** whether the radar ships in version one as a presence sensor (userspace driver; Google's gesture and sleep features do not transfer; the regulatory position on non-stock radar configuration is open); whether to recreate the INA231 power monitors and the temperature sensors.
- **For the Max:** whether to attempt the camera at all (deferring it is legitimate); whether to keep the three NTC thermistors. TMP112 and BMA456 are absent from the `b4` revision.

## D8. Operating system and product scope

- **Decided:** the product is a retrofit board sold to other people, running Hearth (the Flutter kiosk app in the Hearth-Home repo).
- **Decided:** Linux, not Android. Hearth already runs on Linux through flutter-pi, the board needs no Android hardware layers, and Linux images and update tools are fine to use commercially. Voice runs as Home Assistant's Linux Voice Assistant alongside Hearth, as on the Pi build today.
- **Decided:** no Raspberry Pi silicon.
- **Consequences:**
  - The driver tracker's Android-layer column and the "Works in Android" status step no longer apply; the last step becomes "Works in Hearth".
  - The project name still says Android.
  - The chip must have a mainline GPU and display stack that flutter-pi, its GStreamer video player and WPE WebKit work on; see "Linux stack and chip selection" in `docs/Compute modules and board design.md`.
