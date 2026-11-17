# Decisions log

One entry per design decision. Change the status and fill in the outcome when a decision is made; do not delete the reasoning. Tracker rows refer to these by ID in their `blocked_by` column. Background for each is in `docs/Compute modules and board design.md`.

| ID | Decision | Status | Outcome | Date |
|---|---|---|---|---|
| D1 | Target model, and one board or several | Partly decided | 1st gen is the first target. Whether the 2nd gen shares a schematic core is open. | 2026-11-17 |
| D2 | Compute module | Open | | |
| D3 | Microphone path | Open | | |
| D4 | Amplifier | Open | | |
| D5 | Backlight driver | Open | | |
| D6 | Radio and antennas | Open | | |
| D7 | Optional peripherals and thermal | Open | | |

## D1. Target model

- **Decided:** start with the 1st gen. It is the best documented and simplest: no radar, no camera, one amplifier, known 14 V supply.
- **Open:** whether the 2nd gen shares a schematic core. Needs both boards measured.

## D2. Compute module

- **Options:** Raspberry Pi CM5 (newest community Android, panel needs a C driver entry, non-commercial licence); Radxa CM4 / CM5 (vendor Android 14, panel described in device tree); NXP i.MX 8M (current vendor Android, heaviest integration).
- **Blocks:** almost every driver row, the power rail list, the microphone path and the board outline.
- **Needs before deciding:** whether the candidate has a PDM input usable under Android; its kernel version (sets how much porting the light-sensor patch and any vendor touch driver need); its power draw against the enclosure's thermal limit.

## D3. Microphone path

- **Options:** stock PDM microphones straight into the module with software echo cancellation; through an XMOS-class voice processor; replaced by an added array.
- **Depends on:** D2.

## D4. Amplifier

- **Options:** refit a TAS2770 (mainline driver, like-for-like); a simpler part such as a MAX98357A.
- **Needs before deciding:** speaker DC resistance; the stock chip marking; how speaker protection is done without Google's tuning.

## D5. Backlight driver

- **Options:** refit an LP8556 with Google's register values (proposed); a different driver once the LED strings are characterised.
- **Needs before deciding:** measured string voltage and current; which of the two published register sets retail units use.

## D6. Radio and antennas

- **Options:** the module's own Wi-Fi with its own antenna; a radio wired to the stock antennas.
- **Needs before deciding:** antenna connector type.

## D7. Optional peripherals and thermal

- **For the 1st gen:** whether to fit temperature sensors, and whether the chosen module stays within what the stock heat sink can shed.
