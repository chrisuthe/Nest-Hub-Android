# Display connector pinout findings

Targeted follow-up to the hardware research report, 2026-11-17. Goal: find the ribbon (FPC) pinout for the stock display on each Nest Hub model.

**Result: no pin-by-pin assignment is public for any of the three models.** The connector format is now known for two of them, and the pin table for the 1st gen panel exists in a paid datasheet.

Detailed notes, with every claim tagged by evidence level, are in `research_notes/Display connector pinouts/`.

## Evidence levels

- **Confirmed**: stated for the exact part by a source that had the datasheet, or read directly from a photo.
- **Sibling**: from a related panel. Not safe to wire from.
- **Inferred**: reasoning from other facts. Must be measured.

## Summary by model

| | Nest Hub 1st gen | Nest Hub 2nd gen | Nest Hub Max |
|---|---|---|---|
| Panel | BOE TV070WSM-TG1 (or TG0) / Innolux P070ACB-DB0 | BOE TV070W… / K&D KD070D82-39TI-A010 | BOE TV101WXM-AG0 / Innolux P101DEZ-DD0 |
| Panel interface | 39 pins, 0.3 mm (confirmed, BOE) | 39 pins (inferred from "-39TI") | 51 pins (confirmed, BOE) |
| Datasheet connector | I-PEX 20660-039E-01 (confirmed, BOE) | not found | not found |
| Mainboard display socket | white ZIF, count unreadable | J12, ZIF, 33 contacts ±1 at 0.3 mm (photo count) | white ZIF, 16–18 mm long, count unresolved |
| Panel supplies | 3.3 V VCC + 3.3 V VDDI | not found | VDDIO + external AVDD/AVEE ±5.5 V |
| Backlight | 7S3P, 21 LEDs, no driver on module | not found | 6S6P, 36 LEDs, no driver on module |
| Touch | on-cell, controller on module, I2C | controller on panel flex assembly | PCAP assembly; tail arrangement unknown |
| Display/touch flex | one shared jumper flex | one shared jumper flex (G653-01131-02) | inferred shared |
| Pin table | in paid datasheet | not found | not found |

## Nest Hub 1st gen

Source: Panelook pages for TV070WSM-TG1 and the I-PEX connector, saved by hand 2026-11-17. Panelook's entries are taken from BOE datasheet Rev.P4 dated 2020-07-10.

- Connector: I-PEX 20660-039E-01, MINIFLEX 3-BFNH L-HD series. 39 pins, 0.3 mm pitch, single row, side entry, back-lock, SMT, for 0.20 mm FPC, 0.9 mm mated height.
- Supply: 3.3 V VCC at 35 mA typical, 3.3 V VDDI at 19 mA typical. Panel logic 0.25 W typical; 1.55 W typical total.
- Backlight: WLED 7S3P, bottom edge-lit, 15,000 h minimum life, no driver on the module, carried on the same 39-pin interface.
- Touch: on-cell, 5-point, controller on the module, I2C.
- Outline: 114.28 x 177.37 x 3.76 mm, active area 89.244 x 152.267 mm, 130 g maximum.
- Pin Configuration field on Panelook is blank. The 29-page datasheet is a paid download.

Inferred, to be measured:

- Backlight is about 20 mA per string at about 21 V: roughly 1.3 W of backlight power across three strings of seven LEDs.
- Both rails at 3.3 V with no AVDD/AVEE listed suggests bias rails are generated on the module.

Open points:

- **TG0 versus TG1.** The TG1 datasheet postdates the 2018 Home Hub by two years. Panelook lists a TV070WSM-TG0 (600x1024, 440 nits) on the same I-PEX connector, which may be the original panel. Read the label on a real unit.
- **Which side the I-PEX part is on.** The mainboard uses a removable jumper flex with fingers at both ends, so the datasheet connector may be on the panel's flex assembly and not on the mainboard.
- No data at all was found for the Innolux P070ACB-DB0. Sibling P070ACB-DB1/DB2 list 39 pins, 3.3 V and a 6S3P backlight, with a different outline.
- The mainboard socket's pin count could not be read from the FCC or EDN photos.

## Nest Hub 2nd gen

Source: Hackaday.io teardown photos and FCC internal photos. I viewed the J12 close-up myself and confirm the J12 designator, the jumper flex printed MAINBOARD, and an LP8556-marked IC with an inductor beside the connector. The image resolution available to me was too low to verify the contact count, which is the photo researcher's.

Mainboard `G651-04810-06 REV B`, about 101 x 46 mm.

| Ref | Goes to | Style | Contacts | Pitch |
|---|---|---|---|---|
| J12 | Display + touch | ZIF FPC, back-flip, staggered | 33 (±1) | 0.3 mm |
| J15 | Radar board | ZIF FFC | 10 | 0.5 mm |
| J6 | Mic / mute strip board | ZIF FFC | 12 | 0.5 mm |
| J5 | Small top-centre board | ZIF FFC | 6 | 0.5 mm |
| J8 | Side board | ZIF FFC | 6 | 0.5 mm |
| J4 | DC-jack board | ZIF FFC | 16 | 0.5 mm |
| J7 | Speaker | wire-to-board | 5 positions, 3 wires | ~1.25 mm |

- Display and touch share one jumper flex, `G653-01131-02`, ends printed MAINBOARD and DISPLAY. It plugs into the panel's own flex assembly, which carries the Goodix touch controller.
- The backlight driver (LP8556) is on the mainboard beside J12, so the LED strings run through J12.
- Radar board `G651-05613-03 REV A` has test pads labelled 3V3, 1V8, GND, 3V4, CS, RST, MISO, MOSI, IRQ, CLK. The FFC pin order is not shown.

**Unresolved discrepancy:** the mainboard socket is counted at 33 contacts, while the K&D part number suffix and the 1st gen BOE datasheet both point to 39 pins at the panel. Either the jumper flex has different pin counts at each end, or one count is wrong. The jumper flex is the thing to measure.

## Nest Hub Max

Source: distributor summaries of the BOE TV101WXM-AG0 datasheet Rev.P0, and partial viewing of the FCC internal photos.

- 51 pins, 4-lane MIPI. One listing gives the flex marking as "TV101WXM-AG0-P0".
- Supplies listed as 3.3 / 5.5 / -5.5 V (VDDIO / AVDD / AVEE). The 3.3 V logic level is unusual for a JD936x panel and should be measured.
- Backlight WLED 6S6P, no driver on the module.
- The unnamed "lcd_bias" I2C device at 0x3E is most likely a TPS65132-class dual ± supply on the mainboard, generating AVDD/AVEE (inferred from datasheet addresses; chip marking not identified).
- Mainboard `G651-01768-05`, about 117 x 62 mm. The display socket shows about 27 solder tails on one side, which fits either about 27–30 pins at 0.5 mm or about 51–55 at 0.3 mm staggered.
- Nothing was found for the Innolux P101DEZ-DD0.

A full pinout was found for the sibling BOE TV101WXM-NLB, but it has 39 pins, a Himax driver IC, on-panel bias and no touch. **Do not wire the Max panel from it.** It is recorded in the notes for reference only.

## How to get the actual pinouts

1. Buy the TV070WSM-TG1 datasheet from Panelook (29 pages, Rev.P4), and the TG0 one if listed. This is the only known source of a real pin table.
2. Ask a distributor for the TV101WXM-AG0 spec; twscreen's page metadata references a PDF.
3. On a physical unit: read the panel label and flex markings, and count contacts at both ends of the jumper flex.
4. Probe a stock mainboard with the display unplugged: grounds by continuity, LED anode and cathodes back to the LP8556 pins, bias rails back to the bias IC, I2C and reset lines back to known test points, DSI pairs by their matched routing.
5. With a unit still running stock firmware, measure rail voltages and backlight string voltage at the connector before disassembly goes further. This is passive measurement only.

## Sources

- Panelook TV070WSM-TG1: https://www.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_parameter_51122.html
- Hackaday.io 2nd gen teardown: https://hackaday.io/project/190147-google-nest-hub-2nd-gen-teardown
  - J12 close-up: https://cdn.hackaday.io/images/5872331679376498675.jpg
  - Connector side: https://cdn.hackaday.io/images/6772551679376436399.jpg
  - Radar pinout: https://cdn.hackaday.io/images/6514751679376803135.jpg
- FCC internal photos: https://fccid.io/A4RH1A/Internal-Photos/Internal-Photos-v1-3987263 , https://fccid.io/A4RGUIK2/Internal-Photos/Internal-Photos-20200702-v1-Internal-Photos-5035982 , https://fccid.io/A4RH2A/Internal-Photos/Internal-Photos-rev-4255949
- iFixit 1st gen screen removal: https://www.ifixit.com/Guide/How+to+Remove+Screen+Google+Nest+Hub/139253
- TV101WXM-AG0 summaries: https://panelcd.com/product/tv101wxm-ag0/
- TPS65132 datasheet: https://www.ti.com/lit/ds/symlink/tps65132.pdf
