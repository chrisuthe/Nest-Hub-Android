# Bench log

Every measurement taken on real hardware goes here, newest last. This log is the only thing that can change a tracker row's evidence to **Measured**. Measurements on a unit running stock firmware are passive observation only.

For each entry record: date, which unit (model, panel label, board marking), what was measured and how, the result, and which tracker rows were updated. Put photos in `tracking/photos/` and link them.

## Entries

| Date | Unit | Measurement | Method | Result | Rows updated |
|---|---|---|---|---|---|
| | | | | | |

## Planned measurements, 1st gen

Before opening (unit running stock firmware):

- [ ] Adapter label, barrel dimensions and polarity (`PWR`)

After the display is lifted but still connected and running:

- [ ] Panel rail voltages at the display connector (`DISP-PWR`)
- [ ] Backlight string voltage and current (`BL`)
- [ ] Logic level of the touch, light-sensor and amplifier buses (`TOUCH`, `ALS`, `AMP`)
- [ ] I2C traffic for backlight and touch, to settle which LP8556 register set is used (`BL`)

Opened and unpowered:

- [ ] Panel label: BOE or Innolux, TG0 or TG1 (`DISP`)
- [ ] Contact count and pitch at both ends of the display jumper flex; where the I-PEX connector sits (`DISP`)
- [ ] Display socket pinout by continuity (`DISP`, `DISP-ID`, `DISP-PWR`, `BL`, `TOUCH`)
- [ ] Marking of the chip beside the display connector, to confirm the LP8556 (`BL`)
- [ ] Mic board flex: count, pitch, pinout, supply (`MIC`, `ALS`, `MUTE`, `LED`)
- [ ] DC-jack board flex: count, pitch, pinout (`PWR`, `USB`)
- [ ] Speaker DC resistance, markings, connector family and pitch (`SPK`)
- [ ] Amplifier and touch controller chip markings (`AMP`, `TOUCH`)
- [ ] Location of the volume switches and the amber LED (`BTN`, `LED`)
- [ ] Whether the mute switch cuts microphone power or clock (`MUTE`)
- [ ] Antenna connector type (`RADIO`)
- [ ] Temperature sensor parts present on a retail board (`THERM`)
- [ ] Board outline, mounting holes, connector positions, height limits (`MECH`)

## Planned measurements, thermal (all models)

On the ROCK 4D (RK3576) running Hearth:

- [ ] Board input power at idle, on the photo screen, playing 1080p H.264, playing HEVC, and on a Home Assistant dashboard, with an inline USB-C power meter (`THERM`, `SOM`)
- [ ] SoC temperature over a 30-minute run of each, with no fan and the stock Radxa heat sink removed if practical (`THERM`)
- [ ] The same with the CPU's top frequencies capped, to see how much power a cap saves without visible lag (`THERM`)

On each opened Nest Hub:

- [ ] Stock heat sink or spreader: material, outline, thickness, where it touches the board, how it mounts, and whether it also touches the enclosure or display frame (`THERM`, `MECH`)
- [ ] Gap between the board and the heat sink at the SoC position, to size a thermal pad for a new board (`THERM`, `MECH`)

## Planned measurements, 2nd gen

Before opening (unit running stock firmware):

- [ ] Adapter label (voltage, current, model), barrel dimensions and polarity (`PWR`)

After the display is lifted but still connected and running:

- [ ] Panel rail voltages and any bias rails at J12; what the panel power-on line drives (`DISP-PWR`)
- [ ] Backlight boost output and string voltage and current at J12 (`BL`)
- [ ] Logic level of the touch, sensor, AV, radar, PDM and button lines (`TOUCH`, `ALS`, `HUM`, `AMP`, `RADAR`, `MIC`, `BTN`, `MUTE`)
- [ ] Touch supply, strip-board (microphone) supply and sensor supply (`TOUCH`, `MIC`, `ALS`, `HUM`)
- [ ] Radar board rails at J15: 3V3 and 3V4 expected (`RADAR`)

Opened and unpowered:

- [ ] Panel label and flex markings: BOE suffix or K&D variant; driver IC (JD9364 / JD9365 / ST7703) (`DISP`, `DISP-ID`)
- [ ] Contact count and pitch at J12, at both ends of jumper flex G653-01131-02 and at CN1 (33 versus 39); connector markings (`DISP`)
- [ ] J12 pinout by continuity: grounds, LED anode and cathodes back to the LP8556, I2C and reset lines, DSI pairs by matched routing, straps, rails (`DISP`, `DISP-ID`, `DISP-PWR`, `BL`, `TOUCH`)
- [ ] LEDs per string from the backlight tail pads or panel label (`BL`)
- [ ] J15 radar FFC pin order against the test-pad labels (`RADAR`)
- [ ] J6 strip board flex: count, pitch, pinout, supply; microphone count and positions (`MIC`, `MUTE`, `HUM`, `LED`)
- [ ] J5 and J8 boards: count, pitch, pinout; which holds the TCS3400, SHTC3 and volume switches (`ALS`, `HUM`, `BTN`)
- [ ] J4 DC-jack board flex: count, pitch, what the 16 conductors carry; whether the jack board is passive; whether a hidden USB port exists (`PWR`, `USB`)
- [ ] J7: what the three speaker wires are; speaker DC resistance and markings; connector family (`SPK`)
- [ ] Amplifier chip marking: TAS5805M or TAS5825M (`AMP`)
- [ ] Touch controller marking on the panel flex assembly (`TOUCH`)
- [ ] Whether the mute switch cuts microphone power or clock (`MUTE`, `MIC`)
- [ ] Location and bus of the four TMP112 sensors and four NTC thermistors (`THERM`)
- [ ] SHTC3 versus SHTV3: read the ID register (`HUM`)
- [ ] Antenna construction and which coax jack serves which radio (`RADIO`, `THREAD`)
- [ ] Location of the volume switches and the amber LED (`BTN`, `LED`)
- [ ] Board outline, mounting holes, connector positions, height limits, heat-spreader contact areas, board-mounted switch positions, disassembly method (`MECH`)

## Planned measurements, Max

Desk tasks first (no unit needed):

- [ ] Download the FCC internal-photo PDF and read pages 3 and 7-10 at full resolution: connector counts, chip markings, DC-jack board (`DISP`, `CAM`, `MIC`, `PWR`, `BIAS`, `BL`, `MECH`)
- [ ] Transcribe the Fuchsia `kLcdPowerOnSequenceForAstroSherlockNelson` table (`DISP-PWR`, `BIAS`)
- [ ] Read the Max backlight node for the second panel vendor in the kernel tree; settle 0xA3 (`BL`)

Before opening (unit running stock firmware):

- [ ] Adapter label and model, barrel dimensions and polarity (`PWR`)

After the display is lifted but still connected and running:

- [ ] VDDIO at the display connector: 1.8 V or 3.3 V (`DISP-PWR`)
- [ ] AVDD and AVEE values, and their order relative to reset and DSI start (`BIAS`)
- [ ] Backlight string voltage and current (`BL`)
- [ ] Logic level of the touch, light-sensor, amplifier and camera buses (`TOUCH`, `ALS`, `AMP`, `CAM`)
- [ ] I2C traffic on the backlight bus, to see whether 0xA3 is written (`BL`)
- [ ] Camera rail voltages (analog, digital, I/O) and MCLK at the camera flex (`CAM`)
- [ ] Whether the privacy switch removes camera power or MCLK, mic power or PDM clock (`MUTE`, `CAM`, `MIC`)

Opened and unpowered:

- [ ] Panel label and flex markings: BOE or Innolux, and any G101B158 marking (`DISP`, `DISP-ID`)
- [ ] Contact count and pitch at both ends of the display flex; whether touch has its own tail; what the second small orange flex patch is (`DISP`, `TOUCH`)
- [ ] Display socket pinout by continuity, with the ID strap pins (`DISP`, `DISP-ID`, `DISP-PWR`, `BIAS`, `BL`, `TOUCH`)
- [ ] Marking of the chip beside the display connector (LP8556?) and of the 0x3E bias IC (`BL`, `BIAS`)
- [ ] Touch controller location and marking; chip and firmware ID registers (`TOUCH`)
- [ ] Camera flex: connector part, count, pitch, pinout, reset polarity at the module, which mainboard connector (`CAM`)
- [ ] Mic board connectors: count, pitch, pinout, supply, which mainboard connector (`MIC`)
- [ ] Location and wiring of the light sensor, privacy switch, volume buttons and LEDs (`ALS`, `MUTE`, `BTN`, `LED`)
- [ ] DC-jack board cable: connector, count, pinout (`PWR`)
- [ ] Destination of the top-right, left-edge and four bottom-edge mainboard connectors (`CAM`, `MIC`, `PWR`, `ALS`, `MUTE`, `BTN`, `LED`)
- [ ] Speaker DC resistance and markings for each driver; pin assignment of the 8-position header; connector family; no passive crossover in the enclosure (`SPK`, `AMP`)
- [ ] Markings of the three QFNs near the speaker header (`AMP`)
- [ ] Antenna connector types and which coax goes to which radio, including the Thread jack (`RADIO`, `THREAD`)
- [ ] Whether any USB connector is exposed (`USB`)
- [ ] Thermistor locations; absence of TMP112 and BMA456 on a retail board (`THERM`, `ACCEL`)
- [ ] Board outline, mounting holes, connector positions, height limits, heat-spreader contact; where the board sits (`MECH`)
