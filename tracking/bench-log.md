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
- [ ] Top daughterboard flex: count, pitch, pinout, supply (`MIC`, `ALS`, `MUTE`, `LED`)
- [ ] Base board flex: count, pitch, pinout (`PWR`, `USB`)
- [ ] Speaker DC resistance, markings, connector family and pitch (`SPK`)
- [ ] Amplifier and touch controller chip markings (`AMP`, `TOUCH`)
- [ ] Location of the volume switches and the amber LED (`BTN`, `LED`)
- [ ] Whether the mute switch cuts microphone power or clock (`MUTE`)
- [ ] Antenna connector type (`RADIO`)
- [ ] Temperature sensor parts present on a retail board (`THERM`)
- [ ] Board outline, mounting holes, connector positions, height limits (`MECH`)
