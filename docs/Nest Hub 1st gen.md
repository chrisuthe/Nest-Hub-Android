# Nest Hub 1st gen

The Nest Hub 1st gen (originally Google Home Hub) is a 7-inch smart display built on an Amlogic S905D2G. Its electrical design is well documented from the SoC side: Google's open-source Fuchsia board driver ("astro") and Google's GPL Cast kernel ("estelle") name almost every peripheral, its bus, I2C address and control lines, and publish the full MIPI DSI timings and init sequences for both panel variants. What is missing is physical: no public source gives a pin-by-pin assignment for any connector, the mainboard display socket's contact count could not be read from photos, logic and supply voltages on each flex are unmeasured, and the speaker's impedance and the mainboard's exact outline are undocumented. The only known source of a real display pin table is a paid BOE datasheet; everything else at connector level must be measured on a unit.

Evidence levels used throughout:

- **Confirmed**: stated for the exact part by a source that had the datasheet, read directly from a photo, or taken from Google's own board support code (Fuchsia board driver or GPL kernel device tree).
- **Sibling**: from a related part or another Nest Hub model. Not safe to wire from.
- **Inferred**: reasoning from other facts. Must be measured.

Standing caveats:

- The Fuchsia sources and the kernel device tree give the **SoC-side** view only (which S905D2 pin or bus each peripheral uses). GPIO names are Amlogic SoC pins; they matter as a list of signals the new board must provide, not as numbers to copy. Neither source gives connector part numbers, pin counts or pinouts.
- The GPL kernel tree contains only **prototype revisions (`p0`–`p2`)** of the 1st gen board; `estelle-p2` was the one read. Its details may differ from retail units. Other revisions were not compared. Vendor driver source in that tree was listed, not read. Nothing from it was checked against hardware.
- Distributor "spec page" values for the BOE panel are listing summaries of the datasheet, not the datasheet itself.
- Photo-derived dimensions are scaled estimates (about ±5–10%); verify with calipers before committing a PCB.
- No drivers have been built or tested.

## Identification

| Item | Value | Source |
|---|---|---|
| Product names | Google Nest Hub (1st gen); originally Google Home Hub | EDN, Google |
| Model | H1A | FCC |
| FCC ID | A4RH1A (EDN's text gives "AR4F1A", which appears to be a typo) | fccid.io, EDN |
| Fuchsia board name | astro | Fuchsia tree |
| Cast kernel codename | estelle | GPL kernel tree |
| SoC | Amlogic S905D2G (quad Cortex-A53) | EDN |
| Reference platform | "S905D2 u200" reference board cited as the CPU platform by a community Reddit thread (2021) | Reddit |
| Mainboard silkscreen | `G651-01770-05 REV01`, blue PCB (seen, FCC p5/p6; this is the FCC sample) | FCC photos |
| Mainboard size | Rectangle about 121 x 46 mm with a narrow tab of about 27 x 15 mm extending the top-left corner (seen, FCC p6 inch ruler; medium confidence) | FCC photos |
| External size | Depth 67.3 mm, width 178.5 mm, height 118 mm; 480 g | Google spec page |
| Display (marketing) | 7" LCD, 1024x600, 169.5 ppi | EDN |
| Board revision straps | HW_ID0 = GPIOZ_7, HW_ID1 = GPIOZ_8, HW_ID2 = GPIOZ_3; Fuchsia supports only DVT and PVT revisions | Fuchsia astro.h, post-init.cc |

### Telling it apart from the other models

| | Nest Hub 1st gen (H1A, astro / estelle) | Nest Hub 2nd gen (GUIK2, nelson / elaine) | Nest Hub Max (H2A, sherlock / newman) |
|---|---|---|---|
| FCC ID | A4RH1A | A4RGUIK2 | A4RH2A |
| SoC | Amlogic S905D2G | Amlogic S905D3G | Amlogic T931 |
| Screen | 7", 600x1024 | 7", 600x1024 | 10.1", 800x1280 |
| Camera | none | none | yes |
| Radar | none | Soli radar | none |
| Microphones | 2 | 3 | 2 |
| Mainboard | `G651-01770-05`, blue, about 121 x 46 mm with corner tab | `G651-04810-06 REV B`, about 101 x 46 mm | `G651-01768-05`, about 117 x 62 mm |
| Power | 14 V / 1.1 A barrel | 14 V barrel | 24 V / 1.25 A barrel |

- The 1st and 2nd gen screens are sold as separate, non-interchangeable replacement parts; one 2nd-gen listing states the 2nd-gen screen does not fit the 1st gen.
- Hackaday's "Google Nest Hub Teardown" (2023-05-29) covers the 2nd gen (S905D3G, Soli radar), not this model. A spec-page snippet giving a "1.7 in (43.5 mm) driver" and "three far-field microphones" also belongs to the 2nd gen. Do not use either for the 1st gen.

## Internal layout and connectors

### Boards and cables

Four interconnects leave the mainboard (EDN teardown, Confirmed by text and photos):

1. One flex to the glued display assembly (display and touch).
2. One flex to a top daughterboard carrying both MEMS microphones, the ambient light sensor and the mic-mute switch.
3. One flex to a base board holding the DC barrel jack and a hidden micro-USB port.
4. A two-wire cable to the speaker.

Details:

- **Display link.** The display assembly is glued to the body. The link is a separate, removable orange jumper FPC with gold fingers at both ends; the display rear shows a single ZIF on a small green board (seen, EDN `flex_PCB_cable` and `screen_back` photos). The mainboard therefore does not take the panel tail directly. iFixit describes a single screen ribbon into a flip-lock ZIF socket, with pin 1 marked by a triangle and "1"; no pin count or pitch is stated.
- **Display and touch share that one flex.** The photo-evidence notes record this as seen (one jumper, one ZIF on the display rear). The 7-inch pinout notes record it as not documented and Inferred (iFixit describes one ribbon, and 39 pins on an on-cell-touch module is consistent with touch I2C/INT/RST sharing the FPC). The earlier hardware notes likewise inferred that DSI, panel power, backlight LED strings and touch I2C/INT/RST very likely share the single FPC, not confirmed at pin level.
- **Top daughterboard.** Holds the two MEMS microphones, the ambient light sensor between the two mic openings at the top front, and the mic-mute switch cluster. It connects by a white FFC with a blue stiffener.
- **Base (I/O) board.** Holds the barrel jack and a micro-USB connector hidden under the glued rubber foot, held by a plastic reinforcement bracket. It "contains no active circuitry, only signal routing traces (and the connectors themselves)". It connects by a white FFC; the I/O board end is a flip-lock ZIF (iFixit describes flipping the clip up 90 degrees).
- **Mainboard.** All connectors are on the SoC side; the back has only test pads, gold ground pads and EMI gaskets (seen, EDN). The back also has conductive pads that press against the back of the screen (EDN's author guesses grounding). Two shield cans (Wi-Fi on the left; a large can over SoC/DDR/NAND) with a heat-sink plate and thermal pad over the large can.
- **Mounting.** Seven screws. Hole positions in mm from the top-left of the bounding box: (54, 3), (91, 4), (118, 8), (30, 43), (65, 43), plus gold-pad holes at (23, 12) and (118, 31) (seen, FCC p6 and EDN; low-medium confidence).
- **Antennas.** An orange flex antenna and a second antenna structure sit beside the mainboard near the Wi-Fi/BT chip (EDN). They are separate parts, so they may be reusable with a replacement Wi-Fi module if the connector type matches (Inferred); the antenna connector type (U.FL/MHF versus spring contacts) is not documented.
- **Speaker.** Sits in a heavy two-piece black plastic enclosure in the base with a strong magnet; the cone faces into the device and fires up through vents. It uses the chassis as its acoustic enclosure and port path, venting at the top.

### Mainboard connectors

No J-designators were legible in any photo.

| Position | Goes to | Style | Contacts (confidence) | Evidence |
|---|---|---|---|---|
| Right edge, middle; white housing | Display + touch via orange jumper FPC | ZIF FPC; same look and same length as the 2nd gen's J12 (housing about 10.6–11.1 mm) | **Not countable** at available resolution. Probably the same 0.3 mm class, about 33 (low confidence, Inferred from size match only) | Seen: FCC p5/p6, EDN `PCB_under_heat_sink` |
| Top-right; black | Mic / ambient-light / mute-switch board, white FFC with blue stiffener | ZIF FFC | about 10–12 (low) | Seen: FCC p5, EDN; destination from EDN text |
| Bottom-right; black | I/O board (DC jack + micro-USB), white FFC | ZIF FFC | about 10–12 (low) | Seen: FCC p5; destination from EDN and iFixit text |
| Left edge; black | Speaker, 2-wire | Wire-to-board | 2 (medium); family and pitch unknown | Seen: FCC p6, EDN; EDN text "two-wire cable" |

Sibling reference (2nd gen, not safe to wire from): the 2nd gen's display socket J12 is a white back-flip ZIF with staggered contacts, counted at 33 (±1) at 0.3 mm pitch, housing about 10.3–10.9 mm, FPC neck about 10 mm wide; a candidate family is a 33-position 0.3 mm back-flip ZIF such as the Hirose FH35C-33S-0.3SHW class (Inferred, not read from any marking). Its jumper flex is marked `G653-01131-02` with ends printed MAINBOARD and DISPLAY.

### Disassembly

Order per EDN:

1. Separate the adhesive-bonded display assembly from the body. This is the risky step: EDN's author cracked the glass prying with a thin screwdriver.
2. Disconnect the single display flex.
3. Remove "several dozen" screws, mostly T4 or T8 Torx plus one Phillips near the antennas.
4. Lift out the PCB assembly (mainboard + heat sink + top daughterboard), leaving the speaker and power-jack assembly.
5. The rubber foot is only glued and peels off to expose the micro-USB port.

iFixit has a screen-removal guide and an I/O daughter board replacement guide for this model (an earlier note that no iFixit teardown existed, only a device page and Q&A, is superseded for these two guides). A YouTube screen-replacement video exists (not reviewed).

### What leaves with the mainboard

| Lost with the stock mainboard | Evidence |
|---|---|
| Amlogic S905D2G SoC | EDN |
| RAM: 2x Nanya NT5CC256M16EP-EK (4 Gbit DDR3 each, top side) + 2x Nanya NT5CC128M16JR-EK (2 Gbit each, bottom side) = 12 Gbit (1.5 GB) | EDN |
| Storage: Kioxia/Toshiba TC58NVG2S0HBAI6, 4 Gbit (512 MB) SLC raw NAND | EDN; Fuchsia `astro-rawnand.cc` confirms raw NAND |
| Broadcom BCM4345 dual-band Wi-Fi (802.11b/g/n/ac) + Bluetooth 5.0, in its own shield can; on SDIO with a 32.768 kHz LPO clock from GPIOX_16 (PWM_E), BT_REG_ON GPIOX_17, SDIO reset GPIOX_6 | EDN; Fuchsia astro-gpios.h |
| All regulators (PMIC/regulator part numbers not identified by EDN or Fuchsia) | EDN (base board is passive) |
| TAS2770 speaker amplifier | Inferred from bus and layout (IC near the two output inductors) |
| LP8556 backlight driver | Inferred (see Backlight) |
| Thermistors, and the TMP112 and BMA456 parts if fitted on retail units | Inferred |
| Other visible parts: 220 µF can, two 6R8/100 inductors (amp output), a 4R7 inductor | EDN photos |

| Stays in the enclosure (reusable) | Evidence |
|---|---|
| Display + touch module, including the touch controller | Confirmed for BOE units (datasheet: controller on the module) |
| Jumper flex between panel and mainboard | Seen |
| Mic / light sensor / mute switch daughterboard and its FFC | EDN |
| Speaker and its enclosure | EDN |
| DC jack + micro-USB base board and its FFC | EDN |
| Antennas | EDN; reuse depends on undocumented connector type |
| Enclosure, heat sink | EDN |

What does not transfer with the hardware: Google's amplifier EQ/limiter tuning and far-field voice processing. Each must be rebuilt, substituted or dropped.

## Display

### Panel variants

Two interchangeable 7.0" panels were shipped. Both are 600x1024 **portrait-native**, mounted landscape, so the new system must rotate the framebuffer 90 degrees; "1024x600" describes use, not scan order.

| | Variant A | Variant B |
|---|---|---|
| Panel | BOE TV070WSM-TG1 (possibly TG0 on early units, see below) | Innolux P070ACB-DB0 |
| Driver IC | Fitipower JD9364 | Fitipower JD9364 |
| Touch controller | FocalTech FT3x27 at 0x38 | Goodix GT9293 at 0x5D |
| Strap `disp-soc-vid` | 0 | 1 |

Source: Fuchsia `display-panel.h` and astro `post-init.cc` (Confirmed).

The stock kernel's `estelle-panel.dtsi` lists entries `TV070WSM`, `TV070WSM_FT`, `P070ACB`, `P070ACB_FT`, plus an unrelated `B080XAN01` reference entry. The meaning of the `_FT` suffix is not explained for these entries, and the 1st gen panel file's power steps were only skimmed.

BOE TV070WSM-TG1 data, from Panelook and distributor pages (listing summaries of BOE datasheet Rev.P4 dated 2020-07-10; Confirmed for this panel at listing level):

| Item | Value |
|---|---|
| Type | 7.0" a-Si TFT-LCD "Assembly", 600(RGB)x1024 WSVGA, RGB vertical stripe |
| Interface | MIPI, 4 data lanes |
| Outline | 114.28 x 177.37 x 3.76 mm |
| Active area | 89.244 x 152.267 mm |
| Mass | 130 g maximum |
| Luminance / contrast | 450 nit / 1000:1 (eechain) |
| Supply | 3.3 V VCC at 35 mA typical; 3.3 V VDDI at 19 mA typical |
| Power | Panel logic 0.25 W typical; 1.55 W typical total |
| Backlight | WLED 7S3P (21 LEDs), bottom edge-lit, 15,000 h minimum life, no driver on the module |
| Touch | On-cell, 5-point, controller on the module, I2C |
| Mass production | 2020 Q4 (eechain) |

Brightness figures differ by source: eechain lists 450 nit for TG1, Panelook lists 440 nits for TG0, and Fuchsia sets `backlight_max_brightness = 400.0` nits for the product.

**TG0 versus TG1 (open).** The TG1 datasheet (2020-07-10) and eechain's mass-production date (2020 Q4) postdate the 2018 Home Hub by two years. Either the distributor date is a re-listing date or TG1 is not the only or original 1st-gen BOE suffix. Panelook lists a TV070WSM-TG0 (600x1024, 440 nits) on the same I-PEX connector, which may be the original panel. Fuchsia's comments cite the "BOE TV070WSM-TG1 spec". Other TV070WSM suffixes that exist: NM0, TG0, TG1, TG2, TH0, TL0, TL1, TL2, TU0. Read the label on a real unit.

**Innolux P070ACB-DB0.** No data at all was found for the exact part (zero search hits; no indexed page on Panelook, eechain or twscreen). Sibling data only, from P070ACB-DB1 / -DB2 (same family, 600x1024, not confirmed identical to DB0, not safe to wire from):

- MIPI 4 lanes, 39-pin connector, 3.3 V typical, WLED 6S3P (18 LEDs), 12,000 h, no driver, on-cell touch with I2C (one listing says I2C/SPI).
- DB2 outline 192 x 106.6 x 5.64 mm, active area 152.37 x 89.28 mm, 350 nit, 800:1. The outline differs from the BOE TG1, so DB2 is a different mechanical module with cover lens; do not assume the same FPC.
- None of the listings gives connector part number, pitch or pin table.
- Another suffix, P070ACB-DB6, appears in an FCC test report component list alongside BOE "TV070WSM-TL1-38P0" as alternate LCDs in one unrelated product (report not opened).

### Timings

From Fuchsia `panel-config.cc` (Confirmed, as used by Google):

| | BOE TV070WSM | Innolux P070ACB-DB0 |
|---|---|---|
| H active | 600 | 600 |
| H front porch | 40 (datasheet typical 20) | 80 |
| Hsync width | 24 | 10 |
| H back porch | 36 | 80 |
| V active | 1024 | 1024 |
| V front porch | 19 (datasheet typical 6) | 20 |
| Vsync width | 2 | 6 |
| V back porch | 8 | 20 |
| Pixel clock | 44.226 MHz (datasheet typical 42.4 MHz) | 49.434 MHz |
| Refresh | 60 Hz | 60 Hz |
| Sync polarity | hsync and vsync negative | hsync and vsync negative |
| Values are | Google's, deviating from datasheet typicals as noted | datasheet typical values |

Sibling cross-check (2nd gen kernel device tree, same panel families): BOE `TV070WSM_*` total 700 x 1053, hsync 24 / back porch 36, vsync 2 / back porch 8, 44.226 MHz; Innolux / K&D total 770 x 1070, hsync 10 / back porch 80, vsync 6 / back porch 20, 49.434 MHz. The 1st gen kernel panel file's timings were not extracted.

### DSI link

- MIPI DSI, `dphy_data_lane_count = 4` for both panels.
- BOE config caps the D-PHY clock lane at 180 MHz (panel spec allows 145–250 MHz). Innolux config caps it at 200 MHz (panel spec 125–250 MHz).
- JD9364 allows 80–500 Mbps per lane with 4-lane 24-bit pixel data.
- By arithmetic from the published timings (not a stated figure), the 7-inch panels need roughly 265 to 300 Mbps per lane.
- Sibling (2nd gen device tree): 4 lanes, 400 Mbit/s max per lane, burst video mode.
- Reference documents Google used, per the Fuchsia display driver README: Fitipower/Jadard JD9364 Data Sheet v1.07 (2021-02-26) and JD9364 User Guide v1.01 (2018-05-31). The BOE and Innolux datasheets are cited in Fuchsia comments but were not located publicly.

### Supplies

- BOE TG1: 3.3 V VCC and 3.3 V VDDI (Confirmed at listing level; currents above).
- Both rails at 3.3 V with no AVDD/AVEE listed suggests bias rails are generated on the module (Inferred; not stated by any source).
- Innolux DB0: not found. Sibling DB1/DB2 list 3.3 V.
- Actual rail voltages at the connector have not been measured.

### Panel identification

- One strap pin on the display module, read at boot as GPIO "disp-soc-vid": 0 = BOE, 1 = Innolux (Confirmed, Fuchsia).
- `GPIO_PANEL_DETECT` = GPIOH_5 is probably that strap (Inferred; the name mapping lives in `astro-gpio.cc`, which was not read line by line).
- The variant can also be identified by which touch address answers: 0x38 = BOE unit, 0x5D = Innolux unit.
- An iFixit answer thread quotes a panel label string "inxg86000087011088w02334183008530ge"; the "inx" prefix suggests an Innolux unit.
- A replacement board must support both variants, or the builder must identify the variant in hand.

### Init sequence source

- BOE: `lcd_init_sequence_BOE_TV070WSM_FITIPOWER_JD9364_ASTRO`, a roughly 1,480-line annotated header, `panel/boe-tv070wsm-fitipower-jd9364-astro.h`.
- Innolux: `panel/innolux-p070acb-fitipower-jd9364.h`, roughly 250 lines.
- The Fuchsia sequences use a simple opcode stream (sleep / GPIO / DCS write / read-reg) that can be transcribed into a Linux JD9365-family or `panel-simple-dsi`-style init table (Inferred).
- The stock kernel sends the same data as the Fuchsia tables (stated for the 2nd gen entries).
- These files should be archived at a pinned commit; nothing guarantees they stay on the Fuchsia `main` branch.

### Power and reset sequence

- SoC pins: `GPIO_LCD_RESET` = GPIOH_6, `GPIO_BACKLIGHT_ENABLE` = GPIOA_10 (Fuchsia astro-gpios.h).
- BOE init header start (Confirmed, Fuchsia): wait 10 ms after VCC; reset high 30 ms, low 10 ms, high 30 ms; read DCS 0x04 (display ID, 3 bytes); then JD9364 page-select (0xE0) register writes.
- BOE spec limits cited in the Fuchsia comments: VCC-ready to reset > 10 ms; reset low pulse > 10 µs; reset to MIPI LP mode > 5 ms.
- Fuchsia references a shared power sequence table, `kLcdPowerOnSequenceForAstroSherlockNelson` (and a matching power-off table), from `panel-config.cc`; one note places it in `initcodes-inl.h`. It was not transcribed.
- Previously open: whether there is multi-rail sequencing. The stock kernel device trees show no multi-rail sequence for any model; it is a reset pulse followed by the init table. For the 1st gen this rests on a skim of the panel file only.
- Sibling (2nd gen kernel, all Fitipower entries; the 1st gen steps were not extracted): drive panel GPIO index 1 low and wait 200 ms ("panel power on"; what index 1 drives is not shown); start the DSI signal; wait 10 ms; reset high 30 ms, low 10 ms, high 30 ms; read panel register 0x04 (up to 3 tries); send the init table for the detected panel. Power-off: stop DSI, wait 5 ms, reset low, wait 20 ms, GPIO index 1 high, wait 100 ms.

## Display connector and pinout

**No pin-by-pin assignment is public for this model.** Nothing below is a usable pinout; do not design a board from it.

### What is known

| Item | Value | Level |
|---|---|---|
| Panel interface (BOE TG1) | 39 pins, 0.3 mm pitch | Confirmed (BOE, listing level; the 0.3 mm figure was first seen only as a Panelook search snippet, later read from the saved Panelook page) |
| Datasheet connector (BOE TG1) | I-PEX 20660-039E-01, MINIFLEX 3-BFNH L-HD series: 39 pins, 0.3 mm pitch, single row, side entry, back-lock, SMT, for 0.20 mm FPC, 0.9 mm mated height | Confirmed (BOE) |
| Carried on that interface | DSI, supplies, backlight LED strings (no driver on module), on-cell touch I2C | Confirmed for backlight; touch sharing Inferred |
| Mainboard socket | White flip-lock ZIF, housing about 10.6–11.1 mm; pin 1 marked with a triangle and "1" | Seen |
| Mainboard socket contact count | Unreadable from FCC and EDN photos; about 33 at 0.3 mm guessed from size match with 2nd gen J12 | Inferred, low confidence |
| Link | Removable jumper flex, gold fingers at both ends | Seen |
| Innolux DB0 connector | Not found. Sibling DB1/DB2: 39 pins | Sibling |
| Pin Configuration field on Panelook | Blank | – |
| Pin table | In the paid 29-page BOE datasheet (Rev.P4) only | – |

The earlier lookup did not find a mating connector part number; the later Panelook read supplied the I-PEX part above.

### Evidence

- Panelook pages for TV070WSM-TG1 and the I-PEX connector (saved by hand; Panelook's entries are taken from BOE datasheet Rev.P4, 2020-07-10). The page sits behind a login and slider CAPTCHA, which was not bypassed by tooling. Panelook lists "datasheet 1 files" for TG1.
- twscreen and eechain spec pages for TG1. twscreen's page metadata references a file "TV070WSM-TG1.pdf" but exposes no download link.
- FCC internal photos (6 pages; p5 = board with shields and bare back, p6 = shields removed with inch ruler; p2–p4 seen only as thumbnails). The exhibit is listed as dated 2019-03-19 on the index page. FCC schematics and block diagrams are confidential.
- EDN photos (about 700–900 px wide; they will not resolve pin counts). EDN photos other than the five listed under Sources were not opened (`PCB_removed`, `PCB_side1`, `PCB_MEMS`, `PCB_antenna`, `PCB_SoC`, `power_flex`, `power_module`).
- iFixit screen-removal guide; its two step images were not inspected at full resolution.

### Open discrepancies

- **33 versus 39.** The mainboard socket looks like the 2nd gen's J12, which was counted at 33 contacts (±1), while the BOE datasheet gives 39 pins at the panel. Either the jumper flex has different pin counts at each end, or one count is wrong. The jumper flex is the thing to measure, at both ends.
- **Which side the I-PEX part is on.** Because the mainboard uses a removable jumper flex with fingers at both ends, the datasheet connector may be on the panel's flex assembly and not on the mainboard.
- **TG0 versus TG1** (see Display).
- **Innolux units.** Nothing confirms that the Innolux module presents the same connector or pinout as the BOE one.
- **TV070WSM family is not uniform.** The sibling TV070WSM-NM0 (1st-gen Amazon Echo Show panel) is 40-pin 0.5 mm, mating connector Kyocera 04-6299-640-020-846+, supplies 1.8/3.3/9/19/-7 V. It is electrically different; do not use it. It shows the family does not share one pinout or even one pin count.

### Searches that found nothing

- No hobbyist project reusing the 1st gen display or documenting its FPC pinout was found (Hackaday, EEVblog, GitHub, Reddit, blogs).
- A Reddit thread asks how to drive the screen from another board; it could not be read, and the snippet shows a question only.
- "P070ACB-DB0" exact searches (alone and with datasheet/pinout terms, English and Chinese): 0 results. "P070ACB" PDF search: only FCC component lists.
- "TV070WSM-TG1"/"TG0"/"TG2" + 39 pins / on-cell / pin assignment: only distributor summaries. TV070WSM datasheet PDF search: only FCC component lists and an Echo Show teardown.
- Generic "7 inch 600x1024 JD9365/JD9364 39pin 0.3mm on-cell" datasheet: 0 results.
- beyondinfinite.com: no P070ACB or TV070WSM files indexed.
- Replacement-screen listings give no FPC markings or pin counts.
- Not searched: YouTube, Japanese/Chinese teardown blogs, TechInsights, fcc.report mirrors.

## Backlight

### LED arrangement

| Item | Value | Level |
|---|---|---|
| BOE TG1 LEDs | WLED 7S3P, 21 LEDs, bottom edge-lit, 15,000 h minimum, no driver on the module, carried on the 39-pin interface | Confirmed (BOE) |
| String count | 3; the LP8556 is configured for "3 phase drivers" in the stock kernel, matching 7S3P | Confirmed (kernel, prototype revision) |
| Maximum current | 20 mA, per the kernel LP8556 comment "20mA MAX_CURRENT". Previously an inference from generic white-LED values | Confirmed (kernel, prototype revision) |
| String voltage | About 7 x ~3 V = ~21 V | Inferred; measure |
| Backlight power | Roughly 1.3 W across three strings of seven LEDs | Inferred |
| Innolux DB0 LEDs | Not found. Sibling DB1/DB2: 6S3P, 18 LEDs (about 18–19 V) | Sibling |
| LED Vf / If from datasheet | Not found | – |

The two panel sources may have different series counts (7S versus 6S). A constant-current boost driver such as the stock LP8556 handles both; a fixed-voltage design would not.

### Driver location

- TI LP8556 at I2C 0x2C, on Fuchsia bus "I2C_3" (SoC pins GPIOA_14 SDA / GPIOA_15 SCL), shared with the audio amplifier. Enable line `GPIO_BACKLIGHT_ENABLE` = GPIOA_10.
- The BOE module has no backlight driver (Confirmed, datasheet), so only LED strings cross the display flex on BOE units.
- That the LP8556 itself sits on the mainboard beside the display connector is **Inferred**: FCC p5 shows a QFN plus a "100" inductor and four capacitors right beside the display connector, the same arrangement as the 2nd gen, where the LP8556 (U1, marked `L8556E9`) is seen there. The chip marking was not read on the 1st gen.
- Consequence: the new board needs its own LED boost driver.

### LP8556 configuration

The two Google sources give **different register sets** for this model. Both are recorded; neither has been checked against a retail unit.

| Register | Fuchsia astro `backlight.cc` | Kernel `estelle-p2` (prototype revision) | Meaning, per kernel source comments |
|---|---|---|---|
| Device control (Fuchsia writes register 0x01; treated here as the same register, which neither source states) | 0x85 | 0x05 | Brightness by I2C register only, backlight enabled |
| 0xA0 | – | 0xFF | Current LSB |
| 0xA1 | – | 0x37 | 20 mA maximum current |
| 0xA2 | 0x30 | 0x20 | Undervoltage protection (0x20 = 2.5 V; 0x30 = 5.2 V per the 2nd gen column) |
| 0xA3 | 0x32 | – | 50 ms slope, 12-bit resolution |
| 0xA5 | 0x54 | 0x54 | 9.6 kHz PWM; 3 phases |
| 0xA7 | 0xF4 | – | Boost inductor current limit 1.6 A |
| 0xA9 | 0x60 | – | Maximum boost voltage 25 V |
| 0xAE | 0x09 | – | LED fault threshold 3 V |

- The Fuchsia astro values (seven writes) are identical to the kernel's 2nd gen (`elaine-b4`) values. The meanings in the right-hand column for 0xA3, 0xA7, 0xA9 and 0xAE come from the kernel's comments on those 2nd gen values.
- Initial brightness in the kernel is 0x800 of 0xFFF (half).
- Fuchsia: `backlight_max_brightness = 400.0` nits.
- The earlier note that the register values had not been decoded against the TI datasheet is partly superseded by the kernel comments above; no independent decode against the datasheet was done.
- Reusing the LP8556 on the new board means copying these values, and keeps the part on a mainline Linux driver.

## Touch

| | BOE units | Innolux units |
|---|---|---|
| Controller | FocalTech FT3x27 (Fuchsia `device_id = kFt3X27`); the GPL kernel's vendor `focaltech_touch` driver carries an FT3427 config | Goodix GT9293 (named in the Fuchsia panel-type comment) |
| I2C address | 0x38 | 0x5D |
| Kernel node | FocalTech `fts`, 600 x 1023, 10 touches | Goodix `gt9xx`; the device tree carries a full Goodix config blob |
| Interrupt | GPIOZ_4; no SoC pull ("driven by the touch controller") | GPIOZ_4; driver expects interrupt pulled up (SoC internal pull) |
| Reset | GPIOZ_9 | GPIOZ_9; driver expects reset pulled down (SoC internal pull) |
| Firmware | `needs_firmware = false`: no firmware download needed on this model | – |

- Bus: Fuchsia "I2C_2" (S905D2 I2C2), SoC pins `GPIO_SOC_TOUCH_I2C_SDA` = GPIOZ_14, `GPIO_SOC_TOUCH_I2C_SCL` = GPIOZ_15, 400 kHz. Fuchsia and the kernel agree on interrupt and reset pins.
- In the stock kernel both controllers are declared and the one that answers is used, so a new board can do the same.
- **Exact FocalTech part.** Fuchsia says only "FT3x27" (FT3427 versus FT3327 etc. was open). The kernel tree's vendor driver has an FT3427 config, and the device tree findings refer to the part as FT3427. The chip marking has not been read.
- **Touch point count conflict.** The kernel node declares 10 touches; the BOE datasheet summary says 5-point. Not resolved.
- **Controller location.** Previously unstated by any source and only inferred (controller type follows the panel vendor and needs no firmware). The BOE datasheet summary states on-cell touch with the controller on the module and an I2C interface (Confirmed, BOE). For Innolux units this remains Sibling/Inferred (DB1/DB2 list on-cell touch with I2C). Not confirmed visually on either.
- Sibling (2nd gen): the panel's own flex assembly carries the Goodix controller, silk `TP_VDD`, `LEDPWM`, `VDDIN`, and a small QFN inferred to be the panel bias supply.
- Not documented: touch I/O voltage (1.8 V versus 3.3 V), supply rail, and which FPC pins carry SDA/SCL/INT/RST.
- Google's spec page lists "Capacitive touch controls".

Vendor drivers in the GPL tree: `focaltech_touch` (configs for FT3427 and FT5726) and `goodix_touch` (GT9xx).

## Audio

### Microphones

- Two far-field MEMS microphones with PDM output, on the top daughterboard, one opening each side of the ambient light sensor on the front (Google spec page; EDN).
- SoC pins: GPIOA_7 = PDM_DCLK, GPIOA_8 = PDM_DIN0. PDM config `number_of_channels = 2`, `sysClockDivFactor = 4`, `dClockDivFactor = 250`; audio HIFI PLL set to 768 MHz (Fuchsia `astro-audio.cc`).
- Two channels on one data pin implies the two mics share one data line with opposite L/R select (standard stereo PDM) (Inferred). A replacement board needs one PDM clock output and one PDM data input.
- PDM clock: 768 MHz / 250 = 3.072 MHz (48 kHz x 64), if the divider is applied directly to the HIFI PLL. This is a calculation from the config values, not a stated figure.
- Microphone manufacturer and part number: not found.
- Not documented: daughterboard flex connector type, pin count, pinout and supply voltage.
- Not documented: whether the mute switch electrically cuts mic power or clock, or is only a GPIO read by software (Fuchsia reads it as a GPIO).
- Google lists "Ultrasound sensing" for this model; no dedicated ultrasound hardware was identified (probably uses the speaker and mics; unconfirmed).

### Amplifier

- TI TAS2770 at I2C 0x48. Fuchsia binds a TAS2770 driver (`BIND_PLATFORM_DEV_DID_TAS2770`, codec type `Tas27xx`), device name "audio-codec-48", `I2C_AUDIO_CODEC_ADDR = 0x48`, on I2C_3 (GPIOA_14 SDA / GPIOA_15 SCL). The stock kernel also declares a TAS2770 at 0x48 in I2S format.
- EDN did not identify the amplifier chip marking, so the exact TAS27xx variant is not visually confirmed.
- Digital audio: S905D2 TDM_B in I2S mode, 16-bit samples in 32-bit slots. Pins GPIOA_1 = TDMB_SCLK, GPIOA_2 = TDMB_FS, GPIOA_3 = TDMB_D0 (out), GPIOA_6 = TDMB_DIN3 (in). The codec uses the right I2S slot; mono; 48 kHz and 96 kHz listed; delta gain -1.5 dB.
- TDMB_DIN3 is most likely the amplifier's I/V-sense feedback data returned to the SoC (Inferred; not stated in the source).
- Control pins: `GPIO_SOC_AUDIO_EN` = GPIOA_5 (pulsed low 1 ms then high as hardware reset; the kernel also lists reset on GPIOA_5), `GPIO_AUDIO_SOC_FAULT_L` = GPIOA_4.
- The amplifier is on the mainboard (Inferred: mainboard I2C bus, only a two-wire cable to the speaker, and an IC near two 6R8/100 output inductors in the photos). It is lost with the stock board.
- The 14 V input is consistent with feeding the class-D amplifier directly from the adapter rail (TAS2770 accepts up to 16 V PVDD per TI's public datasheet); the actual PVDD source was not confirmed.
- The TAS2770 init register sequence is compiled in from a non-public config header (`TAS2770_CONFIG_PATH`), so Google's DSP and limiter tuning is not public. Speaker protection must be re-derived.

### Speaker

- One full-range driver; "Full-range speaker" is the only official description.
- iFixit's device page lists "2 in Full-Range speaker" (search snippet; unsourced wiki text).
- Two-wire cable to the mainboard; connector family and pitch unknown.
- Impedance, rated power and exact driver diameter: not found in any reliable source.
- The speaker uses the chassis volume acoustically, so air leaks or blocking the internal vent path with a larger board would change the sound (Inferred).

## Sensors and other devices

### I2C map

| Address (7-bit) | Device | Bus | Source |
|---|---|---|---|
| 0x18 | Bosch BMA456 accelerometer | not stated | Kernel `estelle-p2` only; may be prototype-only |
| 0x2C | TI LP8556 backlight | I2C_3 (GPIOA_14 SDA / GPIOA_15 SCL) | Fuchsia, kernel |
| 0x38 | FocalTech touch | I2C_2 (GPIOZ_14 SDA / GPIOZ_15 SCL) | Fuchsia, kernel |
| 0x39 | ams TCS3400 light sensor | I2C_AO_0 (GPIOAO_2 SDA / GPIOAO_3 SCL) | Fuchsia, kernel |
| 0x48 | TI TAS2770 amplifier | I2C_3 | Fuchsia, kernel |
| 0x48–0x4B | TI TMP112 temperature sensors (four) | not stated | Kernel only |
| 0x5D | Goodix touch | I2C_2 | Fuchsia, kernel |

- The earlier note that there are no address clashes among 0x2C, 0x38, 0x39, 0x48 and 0x5D, so fewer buses could be used (subject to unknown I/O voltage domains), predates the TMP112 finding. The kernel findings list TMP112 at 0x48–0x4B on all models and the TAS2770 at 0x48 on this one; the buses for the TMP112 parts are not given in what was read, so this apparent overlap is unresolved.
- Three separate I2C buses on the stock board is a layout convenience (Inferred).

### Ambient light sensor ("Ambient EQ")

- ams TCS3400 RGB + clear colour sensor, device "tcs3400-light-39", `I2C_AMBIENTLIGHT_ADDR = 0x39`, on the always-on I2C bus (I2C_AO_0).
- SoC pins: `GPIO_SOC_SENSORS_I2C_SDA` = GPIOAO_2, `GPIO_SOC_SENSORS_I2C_SCL` = GPIOAO_3, interrupt `GPIO_LIGHT_INTERRUPT` = GPIOAO_5.
- Fuchsia driver config: gain 64, integration time 711.68 ms, polling 700 ms.
- In the stock kernel it is declared as `tcs3472`, with `CONFIG_USE_TCS3400` set.
- Located on the top daughterboard between the two mic openings (EDN), so it is reusable; its I2C and interrupt lines travel over the same daughterboard flex as the PDM mic signals (Inferred).
- The sensor sits behind the display cover glass ink, so lux and colour calibration is specific to this design; Google's calibration coefficients were not found.
- Supply voltage and daughterboard pinout: not found.

### Other devices

| Device | Detail | Level |
|---|---|---|
| NTC thermistors | See Buttons, LEDs, thermal | – |
| TI TMP112 temperature sensors | Four per board at I2C 0x48–0x4B, driven as LM75 in the stock kernel. Not in the Fuchsia board driver findings | Confirmed in kernel (prototype revision); presence on retail units unverified |
| Bosch BMA456 accelerometer | I2C 0x18 in `estelle-p2`. Disabled in the Max's later `b4` revision; may be prototype-only. The earlier Fuchsia review found no accelerometer, proximity or temperature/humidity sensor in the board driver (absence in Fuchsia is good but not absolute proof) | Kernel prototype revision only |
| Micro-USB port | On the passive base board under the rubber foot; Fuchsia has `astro-usb.cc` | EDN, Fuchsia file list |
| Wi-Fi / Bluetooth | BCM4345 on the mainboard (lost) | EDN |

Official sensor list (Google): "Ambient EQ light sensor", "Ultrasound sensing", "Capacitive touch controls"; no camera.

## Buttons, LEDs, thermal

### Buttons and switch

| Signal | SoC pin | Electrical | Notes |
|---|---|---|---|
| Volume up | GPIOZ_5 | SoC internal pull-up, inverted (active low) | |
| Volume down | GPIOZ_6 | SoC internal pull-up, inverted (active low) | |
| Volume both | GPIOAO_10 | No internal pull ("external pull-ups used"), inverted | Used as factory-data-reset input |
| Mic privacy (mute switch) | GPIOZ_2 | No internal pull ("external pull-ups used"), not inverted | Kernel key code 248 |

- Placement: mic-mute switch at the top of the back, in the cluster on the top daughterboard; volume up/down along the left edge of the back (EDN).
- "Volume both" on a separate always-on-domain pin implies a small diode or logic AND of the two volume buttons on the stock board, so that holding both wakes or reset-triggers the SoC; a replacement board can ignore it and read the two buttons separately (Inferred).
- Where the volume switches physically sit and how they connect is not documented. They are probably on the mainboard edge or a small flex behind the rear housing (Inferred); EDN does not say.
- Whether the mute switch has a hardware cut of the mic supply or clock in addition to the GPIO is not documented.

### LED

- One amber LED, "AMBER_LED", on GPIOAO_11, PWM-dimmed (PWM_AO_A) (Fuchsia `astro-light.cc`; the kernel also lists an amber PWM LED).
- Probably the mic-mute indicator; its physical location (daughterboard versus mainboard) is not documented (Inferred).

### Thermal

- NTC thermistors read through the SoC SAR ADC. Fuchsia gives the part string "ncpXXwf104" with a 100 kΩ @ 25 °C table; the count and placement were not extracted from Fuchsia (the summary table listed a single NTC thermistor).
- The stock kernel resolves the part and count: Murata NCP03WF104, 47 kΩ pull-up to 1.8 V, **four** on the 7-inch models (prototype revision caveat applies).
- The thermistors are almost certainly on the mainboard and would be lost (Inferred); any 100 k NTC can replace them if thermal monitoring is wanted.
- Four TMP112 sensors are also declared in the kernel (see Sensors).
- Stock thermal path: a metal heat sink covers two shield cans, with thermal tape between heat sink and can, and between can and SoC (EDN).
- The stock design is passive, for an S905D2-class SoC dissipating a few watts in a sealed plastic shell within a 15 W total budget. A replacement SoC of similar or lower power with a comparable heat spreader should be fine; a hotter SoC would need thermal validation. This is engineering judgement, not a sourced figure.

## Power

| Item | Value | Source / level |
|---|---|---|
| Adapter output | 14 V / 1.1 A (~15 W) | EDN (adapter label) |
| Adapter input | 100–240 V, 50–60 Hz, 0.4 A | EDN |
| Cord | 1.5 m | EDN |
| Official description | "15 W power adapter", "DC power jack" | Google spec page |
| Adapter model | W18-015N1A | Third-party seller listings, low reliability |
| Barrel plug | 3.5 x 1.35 mm | Unverified seller claim (retail listing snippets; one is a Q&A on a 2nd-gen product page) |
| Polarity | Centre-positive is typical for this adapter family | Not confirmed by any source; check the adapter label |
| Third-party replacements | "14V 1.1A 15W" adapters sold as compatible with "Google Home Hub / Nest Hub / Nest Mini (2nd)" | Marketplace listings, low reliability |

- The jack is not on the mainboard. It sits with the micro-USB port on the passive base board, so 14 V and the USB signals arrive at the mainboard over a flex, and all regulation is on the board being discarded.
- A replacement mainboard can take 14 V in over the stock base flex once its pinout is traced, and must provide its own buck converters (for example 14 V to 5 V / 3.3 V / 1.8 V / core rails).
- The micro-USB port gives a ready-made route for a USB data, service or debug port on the new board (USB D+/D-/VBUS/ID run up the same flex), subject to tracing (Inferred).
- Not documented: base flex connector type, pin count and pinout (14 V, GND, USB signals); input protection components (fuse, TVS, reverse-polarity) on the stock board; PMIC and regulator part numbers.
- Sibling (2nd gen): the kernel's INA231 monitor labelled `ina231_14V` has an undervoltage alert at 11 V. Earlier guidance for the 2nd gen was to design the input stage for at least 12–16 V tolerance until measured.
- Bus I/O voltages are unknown: S905D2 GPIOZ, GPIOA and GPIOAO banks can be 1.8 V or 3.3 V and this is not documented for this board. Measure before connecting.

## Driver coverage

Android runs on the Linux kernel, so "driver" here means a Linux kernel driver, plus an Android HAL where needed. No drivers have been built or tested.

How this was checked: **Mainline** was read directly from `torvalds/linux` master (driver source and compatible strings) or confirmed absent by a code search returning zero hits. **Vendor GPL** is Google's GPL kernel release as mirrored at github.com/frederic/elaine-linux (Linux 4.9.244, last pushed 2022-06), a third-party mirror; the same code should be obtainable from Google's open-source release page for the product. Only directory listings, the device tree files named above and one other file were inspected.

Status key: **Mainline** = in current upstream kernel. **Vendor** = GPL source exists outside mainline and needs porting to a modern kernel. **Unlisted** = a mainline driver for the family exists but has no entry for this part. **Platform** = depends on the compute module.

### Parts that stay in the enclosure

| Part | Status | Driver / source | Work needed |
|---|---|---|---|
| JD9364 panel driver IC (both panels) | Mainline | `drivers/gpu/drm/panel/panel-jadard-jd9365da-h3.c` | Add a panel entry with our timings and the init sequence from the Fuchsia tables. The driver already carries a K&D panel (`kingdisplay,kd101ne3-40ti`) and Radxa displays as models to copy. Earlier notes had this only as a plausible, unverified starting point. |
| FocalTech FT3x27 touch (BOE units) | Unlisted | `edt-ft5x06.c` has `focaltech,ft3518` and a generic FocalTech path, but no FT3x27 entry | Test with an existing compatible; may need an ID added. Fallback: vendor `focaltech_touch` driver in the GPL tree, with an FT3427 config. |
| Goodix GT9293 touch (Innolux units) | Unlisted | `goodix.c` has no `gt9293` compatible, but falls back to its GT9xx chip data for unknown IDs | Likely works with a neighbouring compatible such as `goodix,gt928`; untested. Vendor `goodix_touch` (GT9xx) is in the GPL tree. |
| ams TCS3400 light sensor | Vendor | Mainline `tcs3472.c` does not support it. Google's GPL tree patches `drivers/iio/light/tcs3472.c` with `CONFIG_USE_TCS3400` | Forward-port Google's patch to the mainline TCS3472 driver. Zephyr also has a TCS3400 driver, useful as a register reference. |
| PDM microphones | Platform | Generic `dmic` codec is mainline; the PDM receiver is the compute module's | Depends on module choice. |
| Buttons, mute switch | Mainline | `gpio-keys` | Device-tree node. |
| Amber status LED | Mainline | `pwm-leds` | Device-tree node. |

### Parts on the stock mainboard (lost; listed to guide part selection)

| Part | Status | Driver / source | Work needed |
|---|---|---|---|
| TI LP8556 backlight | Mainline | `drivers/video/backlight/lp855x_bl.c` | Refit the part; copy Google's register values. |
| TI TAS2770 amplifier | Mainline | `sound/soc/codecs/tas2770.c` | Refit the part or choose another; tuning is ours to derive. |
| Maxim MAX98357A amplifier (candidate substitute) | Mainline | `sound/soc/codecs/max98357a.c` | Configuration-free I2S amplifier. |
| NTC thermistors | Mainline | `ntc_thermistor` via an ADC | Needs an ADC on the new board. |
| TI TMP112 | Mainline (as used in the stock kernel: standard `lm75` driver) | `lm75` | Only if refitted. |
| Bosch BMA456 | Vendor; mainline support not checked | Vendor `bma4xy` driver in the GPL tree | Only if present and wanted. |

Reusing the LP8556 and the stock amplifier part number on the new board keeps both on mainline drivers, with Google's backlight register values already published.

Earlier notes assumed both touch controllers were covered by mainline (`edt-ft5x06` family, `goodix`) without verifying the exact parts; the table above is the verified position. No mainline Linux device tree for "astro" was confirmed.

### Android layer above the kernel

Not verified; from general knowledge of AOSP.

- **Display, touch, buttons**: no HAL work. DRM and evdev are picked up directly.
- **Backlight**: needs the lights HAL pointed at the backlight sysfs node.
- **Light sensor**: needs a sensors HAL that reads IIO devices.
- **Audio**: needs an audio HAL configuration (tinyalsa / mixer paths) for the amplifier and PDM capture.

## What a replacement board must provide

| Function | Requirement |
|---|---|
| Display | 4-lane MIPI DSI output; LCD reset GPIO; one panel-ID strap input; panel supplies (3.3 V VCC and VDDI on BOE units, to be measured) |
| Display variants | Support both BOE and Innolux panels, selected by the strap or by which touch address answers; framebuffer rotated 90 degrees |
| Backlight | Its own LED boost driver with enable and dimming: three strings, 20 mA maximum, about 21 V (7S) and possibly about 18–19 V (6S, Innolux, Sibling). Reusing an LP8556 with Google's register values is the low-risk path |
| Touch | I2C at 400 kHz with interrupt and reset, answering at 0x38 (BOE) or 0x5D (Innolux); probe both |
| Light sensor | I2C plus interrupt for the TCS3400 at 0x39 |
| Microphones | One PDM clock output and one PDM data input carrying two channels |
| Speaker | A mono class-D amplifier fed by I2S, replacing the TAS2770; conservative speaker protection without Google's tuning |
| Buttons | GPIO inputs for volume up, volume down and the mute switch (3–4 GPIO inputs; stock reads them as plain levels with pull-ups) |
| LED | Optional PWM for the amber LED |
| USB | USB 2.0 to the base board's micro-USB port (optional service/debug route) |
| Power | 14 V input over the base flex, converted to all logic rails; about 15 W total budget |
| Thermal | Optional ADC inputs for NTC thermistors; heat spreading comparable to the stock heat sink |
| Radio | Wi-Fi/Bluetooth on the new board; stock antennas possibly reusable depending on connector type |
| Mechanical | Fit the stock outline (about 121 x 46 mm with a 27 x 15 mm corner tab), seven screw positions, connector positions on one side, and not block the speaker's internal vent path |

Design choices specific to this model:

- This model is the best documented and simplest of the three (best teardown, simplest audio, no radar or camera, known 14 V supply), making it the natural first target. A shared schematic core with the 2nd gen is plausible; a shared outline is unknowable until both boards are measured.
- Amplifier: like-for-like TAS2770 or a simpler common part.
- Backlight: reuse the LP8556 with Google's values, or design a different driver once the strings are characterised.
- Radio and antennas: on-module Wi-Fi with its own antenna, or a module wired to the enclosure's stock antennas.
- Recommended staging: prove panel, touch and audio on an off-the-shelf module I/O board through small flex adapter boards before committing to a full form-fit carrier. Panel bring-up failures present as a silent black screen, so this turns the largest single risk into a cheap experiment.
- A sacrificial unit, ideally one still booting stock firmware for passive rail and bus measurements, is the most valuable purchase before design work.

## Open questions and measurements to take

### Desk tasks

1. Buy the TV070WSM-TG1 datasheet from Panelook (29 pages, Rev.P4), and the TG0 one if listed. This is the only known source of a real pin table. Check Panelook for P070ACB-DB0 while logged in.
2. Request datasheets from twscreen, eechain or enlcd (they offer them on inquiry).
3. Transcribe Fuchsia's `kLcdPowerOnSequenceForAstroSherlockNelson` table and read the 1st gen kernel panel file's power steps in full.
4. Read `astro-gpio.cc` to confirm that `GPIO_PANEL_DETECT` (GPIOH_5) is the `disp-soc-vid` strap.
5. Compare the other `estelle` board revisions (`p0`, `p1`) and establish which buses carry the TMP112 and BMA456 parts.
6. Reconcile the two LP8556 register sets (Fuchsia versus kernel `estelle-p2`), ideally by capturing the stock I2C writes on a running unit.
7. Archive the Fuchsia astro and display driver files at a pinned commit.
8. Bench-test the FT3x27 and GT9293 against the mainline drivers before porting anything.

### On a unit still running stock firmware (passive measurement only)

- Rail voltages and backlight string voltage at the display connector, before disassembly goes further.
- Boost output voltage and LED current.
- Logic level of each bus (1.8 V or 3.3 V).
- I2C traffic for touch, backlight and amplifier.

### On an opened unit

- Panel label (TG0 or TG1; BOE or Innolux) and flex markings.
- Contact count, pitch and contact side at **both ends** of the display jumper flex; whether the I-PEX 20660-039E-01 is on the panel flex assembly or the mainboard.
- Display socket pinout, by probing a stock mainboard with the display unplugged: grounds by continuity, LED anode and cathodes back to the LP8556 pins, I2C and reset lines back to known test points, DSI pairs by their matched routing (or by scope), touch I2C by address scan, rails by multimeter.
- Whether the LP8556 is on the mainboard (read the QFN marking beside the display connector) and whether the touch controller is on the panel flex.
- Whether touch shares the display flex or has a separate tail.
- Panel supply rails; whether bias rails are generated on the module.
- Mic/sensor daughterboard flex: type, pin count, pinout, supply voltage.
- Base flex: type, pin count, pinout (14 V, GND, USB signals).
- Speaker: DC resistance, markings, connector family and pitch.
- Adapter label, barrel dimensions and polarity.
- Whether the mute switch cuts mic power or clock in hardware.
- Location of the volume switches and of the amber LED.
- Thermistor, TMP112 and BMA456 presence and placement on a retail board.
- Amplifier chip marking (exact TAS27xx variant); FocalTech chip marking (FT3427?).
- Antenna connector type.
- Input protection components.
- Mechanical envelope: mainboard outline, mounting holes, connector positions, height limits, heat-spreader contact areas, positions of any board-mounted switches that poke through the enclosure, antenna arrangement.

### Unresolved conflicts between sources

- LP8556 register values: Fuchsia versus kernel prototype device tree.
- Display interface count: 39 pins at the BOE panel versus about 33 guessed at the mainboard socket.
- Touch points: 10 (kernel) versus 5 (BOE datasheet summary).
- I2C address 0x48: TAS2770 and one TMP112, buses not established.
- Panel brightness: 400 nits (Fuchsia), 450 nit (TG1 listing), 440 nits (TG0 listing).
- BOE panel suffix: TG1 (Fuchsia comments, 2020 datasheet) versus a possible earlier TG0.

## Sources

### Google source code

- Fuchsia astro board driver directory (read from the `main` branch): https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/
  - astro.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro.h
  - astro-gpios.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-gpios.h
  - astro-i2c.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-i2c.cc
  - astro-audio.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-audio.cc
  - astro-light.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-light.cc
  - astro-buttons.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-buttons.cc
  - astro-thermistor.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/astro-thermistor.cc
  - post-init/post-init.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/post-init.cc
  - post-init/touch.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/touch.cc
  - post-init/backlight.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/backlight.cc
  - Other files in the directory (listed, not all read): astro-pwm.cc, astro-sdio.cc, astro-bluetooth.cc, astro-rawnand.cc, astro-usb.cc, post-init/display.cc
- Fuchsia amlogic-display driver: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display
  - panel-config.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc
  - BOE init sequence: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv070wsm-fitipower-jd9364-astro.h
  - Innolux init sequence: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/innolux-p070acb-fitipower-jd9364.h
  - README.md: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/README.md
- Fuchsia display-panel.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h
- Google GPL kernel (third-party mirror, Linux 4.9; files read: `estelle-p2.dts`, `estelle-panel.dtsi`, `estelle-p2_defconfig` under `arch/arm64/boot/dts/amlogic/` and `arch/arm64/configs/`): https://github.com/frederic/elaine-linux

### Teardowns and photos

- EDN teardown, Brian Dipert, 2020-10-14: https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/
  - Board in chassis: https://www.edn.com/wp-content/uploads/Google_Home_Hub_PCB_under_heat_sink.jpg
  - Back of board: https://www.edn.com/wp-content/uploads/Google_Home_Hub_PCB_back_no_Faraday_cages.jpg
  - Display jumper flex: https://www.edn.com/wp-content/uploads/Google_Home_Hub_flex_PCB_cable.jpg
  - Display rear: https://www.edn.com/wp-content/uploads/Google_Home_Hub_screen_back.jpg
- FCC filing A4RH1A: https://fccid.io/A4RH1A
  - Internal photos: https://fccid.io/A4RH1A/Internal-Photos/Internal-Photos-v1-3987263
  - PDF: https://fccid.io/m/d0ffc974e815edd84cf0220df452538aadc1f4816f616b415c17a4db32294a93.pdf
- iFixit screen removal guide: https://www.ifixit.com/Guide/How+to+Remove+Screen+Google+Nest+Hub/139253
  - Step images: https://guide-images.cdn.ifixit.com/igi/IQEnESt6vdP1nWcD.large , https://guide-images.cdn.ifixit.com/igi/UIT1hptFjbIFTCsM.large
- iFixit I/O daughter board replacement: https://www.ifixit.com/Guide/Google+Home+Hub+1st+Generation+I-O+Daughter+Board+Replacement/149013
  - I/O board ribbon image: https://guide-images.cdn.ifixit.com/igi/cGjpTa6QnAILqSFp.huge
- iFixit device page: https://www.ifixit.com/Device/Google_Home_Hub_1st_Generation
- iFixit answer thread (panel label string): https://www.ifixit.com/Answers/View/600556/
- YouTube screen replacement (not reviewed): https://www.youtube.com/watch?v=AE75AVNT12w
- Sibling 2nd gen teardown used for comparison: https://hackaday.io/project/190147-google-nest-hub-2nd-gen-teardown ; J12 close-up https://cdn.hackaday.io/images/5872331679376498675.jpg ; connector side https://cdn.hackaday.io/images/6772551679376436399.jpg
- Hackaday article on the 2nd gen (not this model): https://hackaday.com/2023/05/29/google-nest-hub-teardown/

### Panel data

- Panelook TV070WSM-TG1 parameters: https://www.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_parameter_51122.html
- Panelook TV070WSM-TG1 overview (mobile): https://m.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_overview_51122.html
- twscreen TV070WSM-TG1: https://www.twscreen.com/lcdpanel/24300
- eechain TV070WSM-TG1: https://eechain.com/en/product/detail/TV070WSM-TG1
- Sibling Innolux P070ACB-DB1/DB2: https://mohotec.com/product/7-0-innolux-p070acb-db2/ ; https://www.enlcd.com/products/19772 ; https://www.yinjidz.com/products-detail.php?ProId=1893 ; https://www.alibaba.com/product-detail/P070ACB-DB2-7-Inch-LCD-Touch_1601222037979.html
- FCC test report listing P070ACB-DB6 and TV070WSM-TL1-38P0 (not opened): https://fccid.io/m/a914b77207fa202ac79b31af529dd6eb6f20992ac6d662ad1fe66f74fda5286f.pdf
- Sibling TV070WSM-NM0 (electrically different): https://m.panelook.cn/TV070WSM-NM0_BOE_7.0_LCM_parameter_cn_32259.html ; https://ronboelectronics.en.made-in-china.com/product/QZzJFhVdbaWm/China-7-0-Inch-Tablet-LCD-Screen-Resolution-1024-600-Boe-TV070wsm-Nm0-Mipi-40-Pins.html

### Product, power and replacement parts

- Google support spec page: https://support.google.com/googlehome/answer/7072284?hl=en
- Adapter listings (low reliability): https://www.aliexpress.com/i/1005008505390379.html ; https://www.target.com/p/google-nest-hub-with-sleepsense-sand/-/A-80209779?showOnlyQuestions=true ; https://www.amazon.com/Dysead-Replacement-Supply-Adapter-Compatible/dp/B0D6LYXJ17 ; https://shopee.sg/14V-1.1A-15W-AC-DC-Power-Supply-Charger-For-Google-Home-Nest-Hub-Mini-Gen-2-2nd-W18-015N1A-G1015-G1030-Smart-speaker-i.860304842.29710786136
- Replacement screen listings (no FPC markings or pin counts): https://www.aliexpress.com/item/4000400520350.html (1st gen; page text not retrievable) ; https://he.aliexpress.com/item/1005013448231437.html (1st / 2nd gen) ; https://www.amazon.com/Home-Nest-Hub-Replacement-Generation/dp/B0CZ9G4HGD (2nd gen; states it does not fit the 1st gen)

### Community

- Reddit, S905D2 u200 reference and USB port under the foot (2021): https://www.reddit.com/r/googlehome/comments/puru6r/hacking_the_google_nest_hub_firmware/
- Reddit, question about driving the screen from another board (not readable): https://www.reddit.com/r/googlehome/comments/11m628w/google_nest_hub_screen_driver/
- Raspberry Pi forum, on why an unknown DSI panel is hard to bring up without a trusted init sequence: https://forums.raspberrypi.com/viewtopic.php?t=392157

### Repository source documents

- `reports/Nest Hub Android hardware research.md`
- `reports/Display connector pinout findings.md`
- `reports/Driver coverage matrix.md`
- `reports/Stock kernel device tree findings.md`
- `research_notes/Nest Hub Android hardware research/nest_hub_gen1_hardware.md`
- `research_notes/Display connector pinouts/pinout_7inch.md`
- `research_notes/Display connector pinouts/connector_photo_evidence.md`
