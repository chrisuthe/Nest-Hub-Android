# Nest Hub 2nd gen

The Nest Hub 2nd gen (2021) is a 7-inch smart display with a 600x1024 MIPI DSI panel, I2C touch, three PDM microphones, one speaker, a colour light sensor, a temperature/humidity sensor and a 60 GHz Soli radar on its own daughterboard. The logical design is almost fully known: Google's Fuchsia board driver ("nelson") and Google's GPL Linux 4.9 kernel device tree ("elaine") name every peripheral, its bus, address, control lines, panel timings, panel init tables and backlight register values, and teardown photos identify every mainboard connector by designator. The biggest unknowns are physical: no pin-by-pin assignment exists for any connector (including the J12 display socket, where a 33-contact photo count conflicts with a 39-pin panel figure), no supply or logic voltage has been measured, and speaker impedance, backlight LED series count and current, and the mainboard mechanical drawing are all undocumented. Nothing here has been checked against hardware, and no driver has been built or tested.

Evidence levels used throughout:

- **Confirmed**: stated for the exact part by a source that had the datasheet or is Google's own board support code, or read directly from a photo.
- **Sibling**: from a related part or another model. Not safe to wire from.
- **Inferred**: reasoning from other facts. Must be measured.
- Photo-derived pin counts and dimensions carry their own confidence note. Photo-scaled dimensions are estimates (about ±5–10%); verify with calipers.

GPIO names such as GPIOZ_13 are Amlogic SoC pins on the stock board. They show which signals exist and their polarity; they are not connector pin numbers and are not numbers to copy.

## Identification

| Item | Value | Source / level |
|---|---|---|
| Product | Google Nest Hub 2nd gen (2021) | |
| Model | GUIK2 | |
| FCC ID | A4RGUIK2 (grant 2021-01-04; internal photos released 2021-07-03) | fccid.io |
| Fuchsia board name | nelson | Fuchsia tree |
| Cast kernel codename | elaine (files `elaine-b4.dts`, `elaine-sm1-panel.dtsi`, `elaine-bx_defconfig`) | Google GPL kernel |
| Radar internal name | "SELINA" | Fuchsia signal names |
| Stock SoC | Amlogic S905D3G, 4x Cortex-A55, 1.9 GHz, with NPU | Hackaday.io, Fuchsia (`AmlVersion::kS905D3G`, `nelson-nna.cc`). The TechInsights summary says "Amlogic S905D" while still describing Cortex-A55; S905D3G is the better-supported identification. |
| Mainboard part number | `G651-04810-06 REV B` (retail unit); `G651-04810-04 REV03` (FCC sample) | Confirmed, read from photos |
| Mainboard size | About 101 x 46 mm (Hackaday photo scale); 101.5 mm wide by the FCC ruler. Wide rectangle with scalloped cut-outs on the top and bottom edges. | Photo estimate, medium confidence |
| Outer dimensions / weight | 177.4 x 120.4 x 69.5 mm, 558 g | Android Authority |
| Board revisions known to Fuchsia | P1, P2, P2_DOE, PRE_EVT, EVT, DVT, DVT2, read from 5 HW_ID strap pins | Fuchsia nelson.h |
| Kernel device tree revision read | `b4`, the latest 2nd-gen revision in the tree. Other revisions were not compared. | Stock kernel device tree |

How to tell it from the other models:

| | Nest Hub 1st gen | Nest Hub 2nd gen | Nest Hub Max |
|---|---|---|---|
| Model / FCC ID | H1A / A4RH1A | GUIK2 / A4RGUIK2 | H2A / A4RH2A |
| Fuchsia / Cast names | astro / estelle | nelson / elaine | sherlock / newman |
| SoC | S905D2G | S905D3G | T931 |
| Screen | 7", 600x1024 | 7", 600x1024 | 10.1", 800x1280 |
| Microphones | 2 | 3 | 2 |
| Radar | none | Soli (BGT60TR13C) | none |
| Camera | none | none | Sony IMX227 |
| Mainboard | `G651-01770-05`, about 121 x 46 mm | `G651-04810-06`, about 101 x 46 mm | `G651-01768-05`, about 117 x 62 mm |
| Extra sensors | NTC only | temp/humidity at 0x70, radar | camera |

Google's spec page lists the 2nd gen with a "1.7 in (43.5 mm) driver" and "3 far-field microphones"; the 1st gen has two microphones and no stated driver size. 1st- and 2nd-gen replacement screens are sold as separate, non-interchangeable parts (one Amazon listing states the 2nd-gen screen does not fit the 1st gen). Hackaday's "Google Nest Hub Teardown" (2023-05-29) covers this model, not the 1st gen or the Max.

## Internal layout and connectors

### Assemblies

TechInsights identifies separate assemblies: main board, gesture (radar) board, sensor board, display/touch subsystem, speaker, an "internal frame" (estimated $3.52) and back enclosure. A Hackaday commenter (not the teardown author; low authority) says the unit "is easily separated into its component parts by simply unplugging all the flat-flex"; another notes that opening risks broken clips.

No iFixit guide or step-by-step teardown for the 2nd gen was found. Disassembly steps (adhesive versus clips, screw types) are not documented in text; the references are the txyzinfo teardown video and the FCC internal photos (8 pages: p3 connector side with ruler, p4 SoC side with ruler, p7 radar board with ruler, p8 display back).

### Mainboard (Confirmed from photos unless noted)

- All FPC/FFC connectors and the speaker header are on the side opposite the SoC.
- SoC side: S905D3G, 2 of the 4 DDR3 chips under one shield frame, TAS5805M amplifier, a 2200 µF / 25 V can, inductors L6, L7, L12 (R68), L2 (4R7).
- Connector side: eMMC (Samsung KLM4G1FETE) or raw NAND (the FCC sample shows a different flash part), 2 DDR3 (U20, U21), BCM4345x Wi-Fi, nRF52811 + SKY66408 (Thread), LP8556 backlight driver (U1), shields SH2/SH3/SH4.
- Mounting: gold-ringed holes. Approximate positions on the SoC-side view, mm from the top-left of the bounding box: (11, 8) with a slot at (11, 11); (68, 6); (95, 8) with a second hole at (95, 11.5); (95, 27); (14, 41). FCC p3 shows 6 EMI spring-gasket pads plus about 5 screw holes. Low-medium confidence on coordinates.
- Two U.FL-style coax antenna jacks: J3 (top-left of connector side) and one at the opposite bottom corner. The earlier text research could not determine antenna construction; the photos show coax jacks, but the antennas themselves and which radio each serves were not identified.
- Scale references used for the Hackaday photos: nRF52811 QFN48 = 6.0 mm, S905D3G measured 14.1 mm, DDR3 96-FBGA 7.5 x 13.3 mm (all three agree). FCC photos use the ruler in frame, which lies in front of the board, so true sizes are equal to or slightly larger than computed.

### Connector table

No connector part number was read from any marking, and no pinout is known for any of these.

| Ref | Goes to | Style | Contacts (confidence) | Pitch | Evidence |
|---|---|---|---|---|---|
| J12 | Display + touch, via orange jumper FPC | ZIF FPC, white housing, dark back-flip actuator on the side away from the cable entry, staggered long/short contacts | 33 (medium-high, ±1; photo count: 19 + 16 − 2 across two overlapping zooms; cross-check on a second photo 33 ±2) | 0.3 mm (measured 0.28–0.29) | Housing about 10.3–10.9 mm long (FCC p3 ruler and Hackaday scale). FPC neck about 10 mm wide. |
| J15 | Soli radar board | ZIF FFC, white housing / black flip | 10 (high; 10 tails at the radar-board end, about 10 at the mainboard end, 10 test pads on the radar board) | 0.5 mm (measured 0.52) | "J15" legible on FCC p3 |
| J6 | Top mic / sensor / mute-switch strip board (FFC printed "AWM 20861 105C 60V VW-1", stiffener marked "MLB") | ZIF FFC | 12 (medium, ±1) | 0.5 mm | Hackaday photo |
| J5 | Small board top-centre (silk `G651-04807-02`), single hole/sensor | ZIF FFC | 6 (medium) | 0.5 mm | Hackaday photos |
| J8 | Side board on the edge opposite the display connector (silk `...10994...`), 6-conductor FFC | ZIF FFC | 6 (high; 6 contacts and 6 conductors visible) | 0.5 mm | Hackaday photo |
| J4 | DC-jack / power board | ZIF FFC | 16 (medium-high, ±1) | 0.5 mm | Hackaday photos, FCC p3 |
| J7 | Speaker | Wire-to-board, black shrouded right-angle header | 5 positions, 3 wires (red, black/grey, green in positions 1/3/5) (high) | about 1.2–1.25 mm (measured 1.17 Hackaday, about 1.1 FCC) | Hackaday photo, FCC p3 |
| J3 + 1 more | Antennas | U.FL-type coax | – | – | Photos |

Notes:

- The J12 contact count is the photo researcher's. The report author viewed the J12 close-up and confirmed the J12 designator, the jumper flex printed MAINBOARD, and an LP8556-marked IC with an inductor beside the connector, but the image resolution available was too low to verify the contact count independently.
- Which of the J5/J8 boards hold the volume buttons, SHTC3 and TCS3400 was not determined from the photos.
- Candidate connector family for J12: 33-position, 0.3 mm, back-flip ZIF (for example Hirose FH35C-33S-0.3SHW class). Inferred; not read from any marking.
- The 1st gen's display socket has the same look and housing length as J12 (about 10.6–11.1 mm), but its count could not be read (Sibling observation only).

### What stays and what leaves with the mainboard

| Location | Parts | Level |
|---|---|---|
| Stays (off-board) | LCD + touch assembly including the Goodix touch controller on the panel's flex assembly; display/touch jumper flex `G653-01131-02`; radar board; mic/sensor/mute strip board (J6); J5 and J8 boards; DC-jack board (J4); speaker; antennas on coax | Confirmed from photos for touch controller, radar, strip board, DC-jack board |
| Leaves (on mainboard) | SoC, RAM, eMMC; Wi-Fi/BT (BCM43458) and Thread radio (nRF52811 + SKY66408); TAS5805M amplifier (SoC side); LP8556 backlight driver (connector side, beside J12); 2x INA231 power monitors; all regulators and the DC input stage; the 3V3 and 3V4 LDOs that feed the radar board; TMP112 temperature sensors and NTC thermistors (location not stated, presumed mainboard) | Amplifier and LP8556 location Confirmed from photos; others from part lists |

Stock compute and radios, for reference only:

- RAM: 4x Samsung K4B4G1646E-BYMA DDR3 4 Gbit = 2 GB (Hackaday.io; TechInsights: "2 GB of DDR3 SDRAM" from Samsung).
- Storage: Samsung KLM4G1FETE-B041 eMMC 5.1. Hackaday's list says "4Gbit"; TechInsights says "4 GB MLC NAND flash, memory controller – Samsung". 4 GB is correct for that part family; the Hackaday "Gbit" is evidently a typo.
- Wi-Fi/BT: Broadcom BCM43458 (802.11ac + BT) on SDIO (GPIOX0-5), BT on UART (GPIOX12-15) and PCM (GPIOX8-11). TechInsights: "Wi-Fi 802.11 a/b/g/n/ac, Bluetooth 5.0 – Broadcom". The photo reading is "BCM4345x".
- Thread/802.15.4: Nordic nRF52811 on SPICC0 (GPIOC0-3) with reset/boot/interrupt lines (INT C5/C7, RST_L A13, BOOT_MODE_L Z1), plus Skyworks SKY66408-11 2.4 GHz front end. Fuchsia gives the SPI rate as 1 MHz; the stock kernel device tree gives 100 kHz with chip select GPIOC_2.
- FCC-certified radios: 2.4 GHz and 5 GHz WLAN, BT/BLE, 802.15.4 (Thread/Zigbee reports) and the 58–63.5 GHz radar.
- Debug access at the SoC: UART-AO TX/RX on GPIOAO(0)/(1), JTAG on GPIOAO(6-9). The Hackaday author used "debug cables" to capture the radar SPI.
- USB exists at SoC level (Fuchsia has `nelson-usb.cc`). Whether a hidden USB/service port is physically accessible on the 2nd gen (the 1st gen has micro-USB under the foot) was not confirmed.

## Display

### Panel variants

The panel is a 7-inch, 4-lane MIPI DSI video-mode LCD, natively portrait 600x1024 at 60 Hz, mounted landscape. The marketing resolution 1024x600 describes use, not scan order, so the new system must rotate the framebuffer 90 degrees.

Sources disagree on the full variant list:

| Source | Variants listed |
|---|---|
| Fuchsia main (post-init.cc, display-panel.h) | `KD_KD070D82_FITIPOWER_JD9364`, `KD_KD070D82_FITIPOWER_JD9365`, `BOE_TV070WSM_FITIPOWER_JD9364_NELSON`, `BOE_TV070WSM_FITIPOWER_JD9365` |
| Fuchsia touch driver, December 2021 snapshot (bootloader panel IDs) | 1 `kd_fiti9364`, 2 `boe_fiti9364`, 3 `inx_fiti9364`, 4 `kd_fiti9365`, 5 `boe_fiti9365`, 6 `boe_sit7703` |
| Stock kernel `elaine-sm1-panel.dtsi` | `TV070WSM_FT9364`, `TV070WSM_FT9365`, `P070ACB_FT9364`, `KD070D82_FT9364`, `KD070D82_FT9365`, `TV070WSM_ST7703I` |

So the confirmed combinations are BOE TV070WSM or K&D KD070D82 glass with a Fitipower JD9364 or JD9365 driver IC; the kernel and the 2021 Fuchsia snapshot additionally carry an Innolux P070ACB/JD9364 entry and a BOE panel with a **Sitronix ST7703** driver IC. Whether the Innolux and ST7703 variants ever shipped at retail is unknown. A replacement board must handle the ST7703 variant or the panel must be identified first.

Physical evidence of which panels exist:

- FCC sample display back label: `KD070D82-39TI-A010 / 20200403 BB2 AB`; cover glass `103040000-00294 TG 2012` (Confirmed, FCC p8).
- Retail (Hackaday) unit: panel flex assembly printed `TV070W... FPCA`, date `20200923` (Confirmed, photo). The BOE suffix on a 2nd-gen unit is unknown.
- TechInsights attributes the "60 Hz display/touchscreen subsystem" in their unit to BOE (estimated cost $29.06), 1024x600.
- Fuchsia's nelson timing comments cite the "BOE TV070WSM-TG1 spec" and the "K&D KD070D82-39TI-A010" datasheet.

### Panel identification

Fuchsia selects the panel from two GPIO strap pins on the display module:

| Strap | SoC pin | Meaning |
|---|---|---|
| `GPIO_DISP_SOC_ID1` | GPIOZ(12) | Vendor: 0 = K&D, 1 = BOE |
| `GPIO_DISP_SOC_ID0` | GPIOZ(11) | Driver IC: 0 = JD9364, 1 = JD9365 |

How the stock kernel chooses among the six entries in its panel file was not read. Its power-on sequence does read a panel register over DSI to check that the panel responds (0x04 for the Fitipower entries; 0xDA, 0xDB, 0xDC for the ST7703 entry). A replacement board should read the straps (or try each init table), or the builder must identify the variant in hand.

### Timings

Fuchsia and the stock kernel agree.

| | BOE TV070WSM (`TV070WSM_*`) | K&D KD070D82 (kernel column: "Innolux / K&D") |
|---|---|---|
| Active | 600 x 1024 | 600 x 1024 |
| H front porch / sync / back porch | 40 / 24 / 36 | 80 / 10 / 80 |
| V front porch / sync / back porch | 19 / 2 / 8 | 20 / 6 / 20 |
| Total (h, v) | 700, 1053 | 770, 1070 |
| Pixel clock | 44.226 MHz (60 Hz) | 49.434 MHz (60 Hz) |
| Sync polarity | hsync and vsync negative | (datasheet typicals; polarity not separately stated for this model) |
| Fuchsia max D-PHY clock-lane frequency | 180 MHz | 200 MHz |
| Kernel DSI setting | 4 lanes, 400 Mbit/s max per lane, burst video mode | same |

- BOE datasheet figures quoted in Fuchsia comments: panel supports MIPI clock 145–250 MHz; datasheet typical pixel clock 42.4 MHz, typical HFP 20, VFP 6. Google's values differ from those typicals.
- The K&D values are the "K&D KD070D82-39TI-A010" datasheet typicals.
- All nelson panel configs set `dphy_data_lane_count = 4`.
- JD9364/JD9365: per-lane data rate 80–500 Mbps for 4-lane 24-bit pixels (D-PHY clock 40–250 MHz), per datasheet references quoted in the Fuchsia comments.
- Inferred (arithmetic from the timings, not stated in a source): about 265–300 Mbps per lane at 24 bpp. Display bandwidth does not discriminate between compute modules.

### Control signals

| Signal | Stock SoC pin | Notes |
|---|---|---|
| Panel reset `GPIO_SOC_DISP_RST_L` | GPIOZ(13) | Active low by name |
| Panel ID0 / ID1 | GPIOZ(11) / GPIOZ(12) | Inputs |
| Backlight enable `GPIO_SOC_BKL_EN` | GPIOA(10) | |
| Kernel "panel GPIO index 1" | not shown | Called "panel power on" in a comment; only panel GPIO index 0 is defined in the files read, so what index 1 drives is not shown |

### Supplies

Not found for the 2nd gen. Panel supply rails (VDD, and whether VGH/VGL or bias rails are generated on the glass/flex or on the mainboard) are undocumented and must be measured.

Related information, not safe to wire from:

- BOE TV070WSM-TG1 (the 1st gen's BOE panel, whose spec Fuchsia also cites for the 2nd gen timings; Confirmed for TG1 from Panelook entries based on BOE datasheet Rev.P4 dated 2020-07-10, applicability to the 2nd-gen panel not established): 3.3 V VCC at 35 mA typical, 3.3 V VDDI at 19 mA typical; panel logic 0.25 W typical, 1.55 W typical total; on-cell 5-point touch with controller on the module, I2C. Both rails at 3.3 V with no AVDD/AVEE listed suggests bias rails are generated on the module (Inferred).
- The 2nd-gen panel flex assembly carries a small QFN with many passives, read as a panel bias supply (Inferred), and silk labels `TP_VDD`, `LEDPWM`, `VDDIN` (Confirmed, photo).

### Init sequence source

- Fuchsia publishes the DSI-on tables as `lcd_init_sequence_BOE_TV070WSM_FITIPOWER_JD9364_NELSON`, `..._BOE_TV070WSM_FITIPOWER_JD9365`, `..._KD_KD070D82_FITIPOWER_JD9364`, `..._KD_KD070D82_FITIPOWER_JD9365`, in headers under `src/graphics/display/drivers/amlogic-display/panel/` (for example `boe-tv070wsm-fitipower-jd9364-nelson.h`, `kd-kd070d82-fitipower-jd9365.h`). Only the index file was read, not the byte tables.
- The stock kernel panel file sends the init table for the detected panel, described as the same data as the Fuchsia tables.
- The Fuchsia tables use a simple opcode stream (sleep / GPIO / DCS write / read-reg) that can be transcribed into a Linux panel driver init table (Inferred from the 1st-gen notes).
- Fuchsia's shared power sequence is `kLcdPowerOnSequenceForAstroSherlockNelson` in `initcodes-inl.h`; it was not read. This was previously an open desk task; the kernel device tree now supplies the sequence below.
- The Fuchsia display driver README lists the reference documents Google used: Fitipower/Jadard JD9364 Data Sheet v1.07 (2021-02-26), JD9364 User Guide v1.01 (2018-05-31).
- Raspberry Pi engineers describe a trustworthy init sequence, timings and D-PHY configuration as the hardest part of bringing up an unknown DSI panel, with failure showing only as a black screen and "almost no debug information". The published tables remove that problem.
- The Fuchsia files should be archived at a pinned commit, since nothing guarantees they stay on the main branch.

### Power and reset sequence (stock kernel, all Fitipower entries)

Power-on:

1. Drive panel GPIO index 1 low, wait 200 ms ("panel power on").
2. Start the DSI signal.
3. Wait 10 ms. Reset high 30 ms, low 10 ms, high 30 ms. Reset is SoC pin GPIOZ_13.
4. Read panel register 0x04 (up to 3 tries) to check the panel responds.
5. Send the init table for the detected panel.

Power-off: stop DSI, wait 5 ms, reset low, wait 20 ms, GPIO index 1 high, wait 100 ms.

The ST7703 entry uses longer reset delays (100 / 10 / 220 ms) and checks registers 0xDA, 0xDB, 0xDC.

No multi-rail sequence appears in the device tree; it is a reset pulse followed by the init table. BOE spec figures quoted in the 1st-gen Fuchsia header for the same panel family: VCC-ready to reset > 10 ms; reset low pulse > 10 µs; reset to MIPI LP mode > 5 ms (Sibling).

## Display connector and pinout

**No pin-by-pin assignment is public.** Nothing below is a usable pinout.

What is known:

| Item | Finding | Level |
|---|---|---|
| Mainboard socket | J12, ZIF FPC, back-flip, staggered, 33 contacts ±1 at 0.3 mm | Photo count; not independently verified |
| Jumper flex | Separate orange FPC with ZIF fingers at both ends, stiffeners printed `MAINBOARD` and `DISPLAY`, marked `G653-01131-02`, `FF-H 94V-0`, `Fw4320`, `E351586` | Confirmed, photo |
| Display + touch | One flex carries both | Confirmed, photo |
| Panel end | The DISPLAY end plugs into a same-size ZIF (CN1) on the panel's own orange flex assembly (`TV070W... FPCA`, date `20200923`) | Confirmed, photo |
| On the panel flex assembly | Goodix touch controller (package marking seen; listed as GT6853 in the Hackaday component list); a second small connector CN2 (touch-sensor tail, about 8–10 contacts, low confidence); a small QFN with many passives (panel bias supply, Inferred); silk `TP_VDD`, `LEDPWM`, `VDDIN`; four soldered backlight tail pads marked + and − | Confirmed, photo, except where marked |
| Backlight path | LP8556 (U1, marked `L8556E9` with TI logo) sits on the mainboard beside a "101"/"100" inductor and a Schottky, a few mm from J12, so the LED anode/cathode strings run through J12 | Driver location Confirmed; string routing Inferred |
| DSI group | Five pairs of small pads/vias beside the upper half of J12, consistent with a 4-lane + clock MIPI DSI group | Pads Confirmed; function Inferred |
| Panel interface pin count | 39 pins | Inferred from the "-39TI" in KD070D82-39TI part numbers only |
| Panel datasheet connector | Not found for either 2nd-gen panel | |
| Pin table | Not found | |

A Facebook group post asks for the "orange cable that connects the screen to the main board" on the 2nd gen, consistent with the separate jumper flex (snippet only).

Sibling data (not safe to wire from):

- BOE TV070WSM-TG1 (1st gen): 39 pins, 0.3 mm, connector I-PEX 20660-039E-01 (MINIFLEX 3-BFNH L-HD series; single row, side entry, back-lock, SMT, for 0.20 mm FPC, 0.9 mm mated height). Its pin table exists only in a paid 29-page datasheet (Rev.P4); Panelook's Pin Configuration field is blank.
- Other TV070WSM suffixes exist: NM0, TG0, TG1, TG2, TH0, TL0, TL1, TL2, TU0. TV070WSM-NM0 (1st-gen Amazon Echo Show panel) is electrically different: 40-pin 0.5 mm, mating connector Kyocera 04-6299-640-020-846+, supplies 1.8/3.3/9/19/−7 V. The TV070WSM family does not share one pinout or even one pin count.
- K&D KD070D82: no datasheet found. The only public variant is KD070D82-39TI-A00x, the Lenovo Tab M7 (TB-7305) LCD + touch assembly; Lenovo's FCC report lists "TV070WSM-TL2 BOE" and "KD070D82-39TI-A004 K&D" as its two LCD sources. A flex marking seen in a marketplace comment for a different tablet: "AS2538A S8C-KD070D82-39TI-A014-LCD-FPC-V1.0 240613". The earlier pinout notes recorded whether the Nest Hub 2 uses a -39TI variant as unknown; the FCC sample label `KD070D82-39TI-A010` and the Fuchsia datasheet citation show that it does.
- Innolux P070ACB-DB1/DB2 (siblings of the 1st-gen Innolux panel; relevant only if the kernel's `P070ACB_FT9364` entry corresponds to a shipped 2nd-gen panel): 39 pins, 3.3 V, 6S3P backlight, different outline.

### Open discrepancies

1. **33 versus 39.** The mainboard socket is counted at 33 contacts, while the K&D part-number suffix and the 1st-gen BOE datasheet both point to 39 pins at the panel. Either the jumper flex has different pin counts at each end, or one count is wrong. The photo evidence also describes CN1 on the panel flex assembly as "same-size" as the jumper's DISPLAY end. The jumper flex is the thing to measure: count contacts at both ends.
2. **Which connector part.** Neither J12 nor CN1 has an identified part number.
3. **Whether touch I2C/INT/RST, panel rails, ID straps and reset all pass through J12.** Expected, since only one flex links the halves, but no pin-level evidence exists.

## Backlight

| Item | Finding | Level |
|---|---|---|
| Driver IC | TI LP8556SQ-E09 ("LED Driver IC 6 Output with Step-Up DC DC Regulator"); marking `L8556E9` on U1 | Confirmed (Hackaday component list and photo) |
| Location | Mainboard, connector side, beside J12 with an inductor and a Schottky | Confirmed, photo. Previously only inferred. |
| Control | I2C address 0x2C on the "AV" bus (S905D3 I2C3; SDA = GPIOA(14), SCL = GPIOA(15)), shared with the audio amplifier | Confirmed, Fuchsia |
| Enable | `GPIO_SOC_BKL_EN` = GPIOA(10) | Confirmed, Fuchsia |
| LED strings | 3 strings: the LP8556 is configured for "3 phase drivers". Previously unknown. | Confirmed, stock kernel device tree |
| LEDs per string, forward voltage, string current | Not found for the 2nd gen | Must be measured |
| On-module driver | None; four soldered backlight tail pads marked + and − on the panel flex assembly | Confirmed, photo |

The "-E09" suffix denotes a specific factory EPROM preset of the LP8556 (boost voltage/current defaults); stock firmware may also override registers over I2C.

### LP8556 register values (stock kernel, 2nd gen `b4`)

| Register | Value | Meaning, per source comments |
|---|---|---|
| Device control (0x01) | 0x85 | Brightness by I2C register only, backlight enabled |
| 0xA2 | 0x30 | Undervoltage protection, 5.2 V on the 2nd gen |
| 0xA3 | 0x32 | 50 ms slope, 12-bit resolution |
| 0xA5 | 0x54 | 9.6 kHz PWM; 3 phases |
| 0xA7 | 0xF4 | Boost inductor current limit 1.6 A |
| 0xA9 | 0x60 | Maximum boost voltage 25 V |
| 0xAE | 0x09 | LED fault threshold 3 V |

- Registers 0xA0 (current LSB) and 0xA1 (maximum current) are not written on the 2nd gen `b4`. On the 1st gen `p2` they are 0xFF and 0x37 ("20mA MAX_CURRENT"); that is a 1st-gen value (Sibling), not a 2nd-gen one. The 2nd gen's string current is not known.
- The 2nd-gen values above are the same seven values Fuchsia writes on the 1st gen (astro `backlight.cc`: 0x01=0x85, 0xA2=0x30, 0xA3=0x32, 0xA5=0x54, 0xA7=0xF4, 0xA9=0x60, 0xAE=0x09). The Fuchsia nelson backlight values were not transcribed.
- Reusing the LP8556 on the new board means copying these values; it keeps the backlight on the mainline `lp855x_bl` driver.

Sibling and inferred figures (do not size anything from these without measuring):

- TV070WSM-TG1: WLED 7S3P (21 LEDs), bottom edge-lit, 15,000 h minimum life, no driver on the module, carried on the same 39-pin interface. The device-tree findings note that the 3-phase configuration matches this 7S3P figure.
- P070ACB-DB1/DB2: 6S3P (18 LEDs).
- Typical white-LED values give about 21 V for 7 in series (about 18–19 V for 6) at about 20 mA per string, roughly 1.3 W across three strings of seven (Inferred from generic values).
- Variants may differ in series count (7S versus 6S). A constant-current boost like the LP8556 handles both; a fixed-voltage design would not.

## Touch

| Item | Finding | Level |
|---|---|---|
| Controller | Goodix GT6853 ("Touch Screen Controller") | Confirmed (Hackaday component list); Goodix package marking seen in photo |
| Location | On the panel's own flex assembly, not the mainboard, so it survives the swap | Confirmed, photo. Previously only inferred. |
| Bus | Dedicated touch I2C (S905D3 I2C2; SDA = GPIOZ(14), SCL = GPIOZ(15)), 400 kHz | Confirmed, Fuchsia and kernel |
| Address | 0x5D | Confirmed |
| Interrupt | `GPIO_TOUCH_SOC_INT_L` = GPIOZ(4), active low by name, falling edge in the kernel | Confirmed |
| Reset | `GPIO_SOC_TOUCH_RST_L` = GPIOZ(9), active low by name | Confirmed |
| Resolution | 600 x 1024 | Kernel device tree |
| Connector | Shares the display jumper flex through J12; touch-sensor tail goes to CN2 on the panel flex assembly | Confirmed, photo (pin positions unknown) |

- The kernel device tree node is the Goodix GTX8 driver with compatible `goodix,gt9886`. The driver coverage matrix records the same vendor driver (`goodix_touch_gtx8`, GPL v2) with compatible `goodix,gt6853`, also seen in `Supernote-Ratta/kernel_a6x_a5x`. Both statements are kept; the driver source itself was only listed, not read.
- The 2nd-gen kernel config builds the GTX8 driver with firmware update support.
- Fuchsia also declares a FocalTech controller at 0x38 on the same bus. The Goodix driver "isn't used on boards earlier than P2", so the FocalTech entry is most likely a holdover from pre-P2 prototypes (Inferred); retail units use the GT6853.
- The Fuchsia GT6853 driver (`src/ui/input/drivers/gt6853/`, BSD) loads one of three config files chosen by panel type (9364, 9365, 7703 driver-IC families): "a config file may contain multiple configs; the correct one is chosen based on the sensor ID reported by the touch controller". The config blobs (`GT6853_CONFIG_9364/9365/7703_PATH`) appear not to be in the open tree (not verified).
- A Fuchsia comment says "There's a chance we can proceed without a config", so the controller's factory-flashed configuration likely suffices on a new board. Needs a test.
- GT6853 is not handled by the mainline `goodix.c` (GT9xx) driver. Mainline `goodix_berlin` only lists GT9916. Earlier notes treated Linux support as an unverified lead; the driver coverage check found the vendor driver above.
- Goodix's developer forum has a thread about porting a "goodix_gt6853" Linux driver to an NXP i.MX 5.4 kernel using Goodix's porting guide (search-result snippet only; page not read).
- Touch I/O voltage (1.8 V versus 3.3 V) and supply rails are not documented; measure. The panel flex assembly has a `TP_VDD` silk label.

Touch is the main software risk on this model.

## Audio

### Microphones

| Item | Finding | Level |
|---|---|---|
| Count | 3 PDM MEMS microphones ("3 far-field microphones", "Mic off switch" per Google) | See conflict below |
| Part | 3x "SD18OB371-077 PDM Mems Microphone" (no public datasheet located; not searched specifically) | Hackaday component list |
| PDM pins | `MIC_DCLK` GPIOA(7); `MICLR_DIN0` GPIOA(8) ("First 2 MICs"); `MICLR_DIN1` GPIOA(9) ("Third MIC") | Confirmed, Fuchsia |
| PDM config | `number_of_channels = 3`; `sysClockDivFactor = 4`, `dClockDivFactor = 250`, HIFI PLL 768 MHz | Confirmed, Fuchsia |
| PDM clock | Probably 768 MHz / 250 = 3.072 MHz | Inferred (arithmetic, not stated) |
| Location | On the strip board bonded to the display glass edge, with the mute switch, reached via J6 (12 contacts ±1, 0.5 mm) | Location Confirmed from photos (FCC p8, Hackaday); mic count on that board is from text |
| Mute sense | `GPIO_MUTE_SOC` = GPIOZ(2) | Confirmed, Fuchsia |

Conflicts on microphone count and location:

- Fuchsia configures three channels (two on DIN0, one on DIN1); the Hackaday list has three microphones; Google's spec says three.
- TechInsights (via Electronics360) counts "Two MEMS microphones from Goertek" on the separate sensor board alongside the Sensirion sensor. The text research inferred from this that the third microphone sits elsewhere, possibly on the mainboard, and would then be lost.
- The photo evidence places the microphones (3x PDM) and mute switch on the J6 strip board, but takes the count from text, not from the photo.
- The stock kernel device tree enables the PDM clock plus two PDM data lines; a third data line is commented out. The device-tree findings conclude that the microphone count should be checked on the board.

"MICLR" plus "first 2 mics on DIN0" implies the standard L/R-select scheme: two microphones on opposite clock edges of DIN0, the third alone on DIN1 (Inferred). A replacement needs one PDM clock output and two PDM data inputs.

The hardware mute switch likely cuts microphone power or clock in hardware in addition to signalling the SoC (typical Google design; `GPIO_MUTE_SOC` is only a sense input). Not confirmed; if true, the location of the mute circuit matters for a redesign.

Microphone supply voltage and the J6 pinout are not documented.

Microphone path options for the new board: stock PDM microphones straight into the SoC with software echo cancellation; through a voice front end such as the XMOS XVF3800 (takes four PDM microphones; echo cancellation, beamforming and noise suppression on-chip; output over USB or I2S); or abandoned in favour of an added array. Whether XMOS firmware suits the stock three-microphone geometry was not researched. The generic `dmic` codec is mainline; the PDM receiver depends on the compute module.

### Amplifier

| Item | Finding | Level |
|---|---|---|
| Part | TI TAS5805M ("23-W, Inductor-Less, Digital Input, Stereo, Closed-Loop Class-D"), wired bridged mono (PBTL) | See conflict below |
| Location | Mainboard, SoC side | Confirmed, photo |
| Control | I2C 0x2D on I2C3 (shared with backlight) | Confirmed, Fuchsia and kernel |
| Audio bus | I2S on TDM-B: SCLK GPIOA(1), FS GPIOA(2), DO0 GPIOA(3); 16-bit samples in 32-bit slots; single mono channel in the left slot | Confirmed, Fuchsia |
| Power-down | `PDN_N` = `GPIO_SOC_AUDIO_EN` GPIOA(5) (kernel: power-down pin GPIOA_5) | Confirmed |
| Fault | `GPIO_AUDIO_SOC_FAULT_L` GPIOA(12) | Confirmed, Fuchsia |
| Boost / inrush enable | `GPIO_INRUSH_EN_SOC` GPIOA(0) (commented "BOOST_EN_SOC"), driven high before amplifier power-up | Confirmed, Fuchsia |
| Fuchsia codec | Type Tas58xx with metadata `.bridged = true` | Confirmed, Fuchsia |

Conflict on the amplifier part: the Hackaday chip-marking list, the photo evidence and the Fuchsia config header names (`TAS5805M_CONFIG_PATH`) say TAS5805M. The stock kernel node is labelled `tas5805` but uses the `ti,tas5825m` driver, and the kernel config enables TAS5825M and not TAS5805. The two parts are closely related; which is fitted should be read from the chip. TechInsights lists only "a TI audio power amplifier" on the main board.

- The stock DSP/EQ and limiter tuning is an init register sequence from a non-public header (`TAS5805M_CONFIG_PATH`, `tas5805m_init_sequence1/2`). The stock speaker protection/EQ coefficients are not public, so protection must be re-derived conservatively: start with conservative gain and a high-pass filter.
- The amplifier is likely fed from the roughly 14 V input rail (the TAS5805M PVDD range covers it), with the inrush/boost enable gating that rail. Not confirmed.
- Any mono class-D amplifier of suitable power can replace it (Inferred). Options with known Linux support: TAS5805M (mainline `sound/soc/codecs/tas5805m.c`; an out-of-tree Sonocotta driver also exists), TAS5825M (not in mainline; a `tas5825m.c` is in Google's GPL tree), MAX98357A (mainline; configuration-free I2S amplifier with known start/stop pops).

### Speaker

| Item | Finding | Level |
|---|---|---|
| Driver | One full-range speaker, 1.7 in (43.5 mm) driver | Google spec |
| BOM estimate | $2.56, quantity 1 | TechInsights |
| Connector | J7, wire-to-board, 5 positions, 3 wires (red, black/grey, green in positions 1/3/5), about 1.2–1.25 mm pitch | Confirmed, photo |
| Impedance, power rating | Not documented | Measure DC resistance; read the driver label |

Conflict: the text research inferred a single 2-wire load across the PBTL bridge; the photos show three wires at J7. What the third wire carries is not established.

"Ultrasound sensing" in Google's spec uses the existing speaker and microphones; no extra hardware (Inferred).

## Sensors and other devices

### Summary

| Device | Bus / address | Sideband | Location | Level |
|---|---|---|---|---|
| ams TCS3400 colour light sensor ("Red, Green, Blue, Clear and IR light sensing") | I2C_AO_0 "SENSORS" (SCL GPIOAO(2), SDA GPIOAO(3)), 0x39 | `GPIO_RGB_SOC_INT_L` = GPIOAO(5) | Not stated; J5 or J8 board not determined. On the 1st gen it is on the mic/sensor board (Sibling). | Confirmed part and bus |
| Sensirion SHTC3 temperature/humidity | I2C_AO_0, 0x70 | – | On the separate sensor board per TechInsights | Confirmed part and bus |
| TMP112 temperature sensors | I2C 0x48–0x4B, four per board, driven as LM75 | – | Not stated | Stock kernel device tree |
| NTC thermistors | SoC SAR ADC | – | Not stated | Stock kernel device tree |
| INA231 power monitors (x2) | I2C3; 0x40 and 0x49 | `GPIO_ALERT_PWR_L` = GPIOZ(10) | Mainboard (lost) | Confirmed |
| Infineon BGT60TR13C radar | SPI | IRQ, reset | Own daughterboard via J15 | Confirmed |
| nRF52811 Thread radio | SPI | INT, reset, boot | Mainboard (lost) | Confirmed |

Details:

- **TCS3400.** Fuchsia driver config: `tcs3400_light`, gain 64, integration time 711.68 ms, polled every 700 ms. The stock kernel declares it as `tcs3472` with `CONFIG_USE_TCS3400` set.
- **SHTC3.** Hackaday reads the part as SHTC3; Fuchsia calls it "SHTV3" (DID `SENSIRION_SHTV3`). Same I2C address and vendor; likely the same or a register-compatible part (Inferred). Treat as SHTC3-compatible but verify the ID register. The stock kernel uses the `shtc1` driver.
- **TMP112.** Not present in the Fuchsia-derived notes. An address overlap exists on paper: the INA231 on I2C3 is at 0x49 and the TMP112 range is 0x48–0x4B; the bus carrying the TMP112 parts is not stated in the source. The stock kernel drives them with the standard `lm75` driver.
- **NTC thermistors.** Murata NCP03WF104, 47 kΩ pull-up to 1.8 V, four on the 7-inch models. Fuchsia has `nelson-thermistor.cc`, `nelson-adc.cc` and `nelson-thermal.cc` (files not read).
- **INA231.** Fuchsia: one on the main-logic-board supply at 0x49 (0x46 on prototypes) and one on the speaker/audio supply at 0x40. Stock kernel: 0x40 labelled 14 V (`ina231_14V`, undervoltage alert at 11 V) and 0x49 labelled 3.3 V, 10 mΩ shunts, driven by the standard `ina2xx` driver. Hackaday lists 2x INA231.
- **Sensor bus voltage.** Likely 1.8 V on the AO bus (unverified). The AO-domain sensor bus on Amlogic SM1 designs is commonly 1.8 V, and GPIOZ/GPIOA banks are commonly 1.8 V or 3.3 V depending on the VDDIO strap; every interface needs its logic level measured.
- Official specs list "Temperature sensor", "Ultrasound sensing", Ambient EQ light sensor and Soli.
- The BMA456 accelerometer found in the kernel tree is listed for the 1st gen `p2` and the Max only, not the 2nd gen.

### Soli radar

The radar is the best-documented reusable part in the family.

**Hardware**

| Item | Finding | Level |
|---|---|---|
| Chip | Infineon BGT60TR13C, "60GHz Radar sensor with integrated one transmitting and three receiving antennas" | Confirmed (Hackaday, Fuchsia `PDEV_PID_INFINEON_BGT60TR13C`, "Radar sensor head") |
| Board | Separate "gesture sensor board", silk `G651-05613-03 REV A` (retail), `G651-05613-01 REV02` (FCC sample); about 12.5 x 21–23 mm | Confirmed, photo; size is a photo estimate |
| On the radar board | BGT60TR13C; LP5907SNX-1.8 (1.8 V low-noise LDO, 250 mA); NSC5103A-80M (1.8 V CMOS 80 MHz oscillator) | Confirmed (photo annotations and component list) |
| On the mainboard near the radar connector corner | TPS7A2501 LDO (U13; TPS7A2501DRVR, adjustable 300 mA) makes 3V4; another LDO makes 3V3 | Confirmed, photo annotation |
| Connector | 10-pin 0.5 mm ZIF with an "ACES"-style logo on the actuator; mainboard end is J15 | Confirmed, photo |
| Test pads, top to bottom | `3V3` (LDO in), `1V8` (LDO out), `GND`, `3V4`, `CS`, `RST`, `MISO`, `MOSI`, `IRQ`, `CLK` | Confirmed (annotation text in photo) |

- **The photo labels test pads, not connector pin numbers.** The pad-to-FFC-pin order is not given and could not be traced.
- TechInsights describes the board as including "a 60 GHz radar module from Infineon and a 1.9 V LDO regulator from TI"; the component list and photo say 1.8 V (LP5907-1.8).
- The earlier text research could not say whether the board takes 3.3 V or 1.8 V; the test-pad labels show a 3V3 LDO input and a separate 3V4 rail supplied from the mainboard.
- Logic is 1.8 V (Inferred: the oscillator and LDO are 1.8 V parts and the BGT60TR13C is a 1.8 V device). Level-shift if the new SoC uses 3.3 V I/O.

**Signals on the stock board**

| Signal | Stock SoC pin |
|---|---|
| `SPI_B_MOSI` | GPIOH(4) |
| `SPI_B_MISO` | GPIOH(5) |
| `SPI_B_SS0` (chip select) | GPIOH(6) |
| `SPI_B_SCLK` | GPIOH(7) |
| `SELINA_RESET` | GPIOH(2) |
| `SELINA_IRQ_OUT` | GPIOH(3) |
| `SELINA_OSC_EN` | GPIOH(8); only actively handled on board build P1 (prototype), where it is set to a no-pull input |

- Fuchsia: SPICC1, CS index 0, client-managed CS; SCLK = core clock / 22 = 30.3 MHz; MISO sampled with a 3-cycle input delay.
- Stock kernel device tree: SPI at 25 MHz, chip select GPIOH_6, interrupt GPIOH_3, exposed to userspace as `spidev` plus a UIO interrupt. Google also ran the radar from userspace.
- The two sources differ on the SPI clock (30.3 MHz versus 25 MHz); Infineon's example also uses 25 MHz.
- Whether Fuchsia's actual radar driver (the thing that binds to "selina-composite") is in the open tree was not confirmed; only the board-level binding was seen.

**Operating behaviour captured from a real unit** (Hackaday.io project, log dated 2023-03-26)

- 80 MHz clock. The chip fills its FIFO with raw ADC data, raises an interrupt, and the SoC reads it with the SPI "burst mode" command.
- About 30 bursts per second; each burst returns a roughly 180 kbit frame of 24-bit blocks.
- `FIFO_CREF` = 7739; `PREFIX_EN` = 1.
- The full list of about 38 configuration register writes (addresses 0x00–0x56 with 24-bit values) is published in the log, plus DSLogic capture files `Config_register.dsl` and `FIFO_read_loop.dsl`.
- The author notes most of the written registers are not described in the datasheet he had (only 0x06 SFCTL was). Two datasheet PDFs are attached to the project (v01 uploaded 2023-03-27; V2.4.9 uploaded 2025-11-16).
- The "BGT60TR13C Radar PCB pinout" log (2023-03-21) consists only of two photos with no text.

**Software**

- Infineon `sensor-xensiv-bgt60trxx`: open, platform-independent C driver library (Apache-2.0 per the driver coverage check; described as MIT-style in the earlier notes), usable with "user-provided communication functions" and not tied to the ModusToolbox HAL. It needs SPI (example uses mode 0, MSB first, 25 MHz), CS, an IRQ pin, a reset pin and an LDO enable. Init takes a register list generated by the `bgt60-configurator-cli` tool. The BGT60TR13C datasheet (V2.4.6) and the configurator ship in Infineon's Radar Development Kit via the Infineon Developer Center.
- Infineon `xensiv-radar-presence` (presence detection library for the BGT60TR13C) and `sensor-dsp`.
- A third-party async no_std Rust crate `bgt60trxx` also supports the BGT60TR13C (snippet only).
- Hobbyists have driven the BGT60TR13C from nRF52833 and ESP32-C3 custom boards (forum thread titles/snippets only).
- There is no Linux kernel driver. Plan: run in userspace over `spidev` with GPIO for IRQ and reset. Presence detection logic is ours to build.
- Android has no framework concept for radar; expose it through a custom service or app (not verified; general AOSP knowledge).

**Limits**

- Raw FMCW data is easy to obtain; the realistic outcome is presence, motion and range sensing. Google's gesture ("Quick Gestures") and Sleep Sensing algorithms are proprietary and do not transfer.
- The FCC grant covers 58–63.5 GHz operation "under waiver of section 15.255(c)(3), DA 18-1308". Running the radar with a non-stock configuration may fall outside the waiver conditions the product was certified under: a regulatory consideration, not a technical blocker.

## Buttons, LEDs, thermal

### Buttons and switch

| Input | Stock SoC pin | Behaviour |
|---|---|---|
| Volume up `GPIO_VOL_UP_L` | GPIOZ(5) | Interrupt-driven, inverted (active low) |
| Volume down `GPIO_VOL_DN_L` | GPIOZ(6) | Interrupt-driven, inverted (active low) |
| Mic mute `GPIO_MUTE_SOC` | GPIOZ(2) | Slide switch sense; polled every 20 ms, not inverted (active high), no pull. Key code 248 in the stock kernel. |
| Factory reset `GPIO_FDR_L` | GPIOAO(10) | Polled, inverted, no pull; bound under the name "volume-both" |

- Fuchsia comment: "No need for internal pull, external pull-ups used."
- `FDR_L` named "volume-both" suggests a hardware AND of both volume keys generates the factory-data-reset signal; a replacement board can ignore it (Inferred).
- The physical mute switch and volume keys are on the back of the device. The mute switch is on the J6 strip board (Confirmed, photo). Where the volume switches sit (mainboard, J5 board or J8 board) was not determined. If they are board-mounted tact switches, the replacement PCB must place equivalents at identical positions.

### LED

- Amber LED: `GPIO_AMBER_LED_PWM` = GPIOAO(11), driven by PWM_AO_A, brightness-controllable, single colour. Only amber on this model.
- It is probably the mic-muted indicator (standard Nest behaviour; not stated in the source). Its physical location is not documented.

### Thermal

- Stock SoC is a roughly 12 nm quad A55 dissipating a few watts inside a sealed fabric/plastic shell with a metal frame/heat spreader (the 1st gen used a heat sink plus thermal tape per EDN; Sibling). A hotter replacement module would need the frame coupled as a heat spreader. Total input budget is 15 W shared with backlight and amplifier. This is engineering judgement, not a sourced figure, and needs validation with the chosen module.
- Temperature monitoring on the stock board: four NTC thermistors and four TMP112 sensors (see Sensors).
- Heat-spreader contact areas are not documented.

## Power

| Item | Finding | Level |
|---|---|---|
| Adapter rating | "15 W power adapter"; "DC power jack"; power cable 1.5 m | Google spec, Android Authority |
| Input voltage | **14 V.** The stock kernel device tree has a TI INA231 power monitor labelled `ina231_14V` with an undervoltage alert at 11 V. | Confirmed (resolved; previously open) |
| Input current | About 1.1 A by the 15 W rating and third-party "14V 1.1A 15W" listings | Low-reliability listings; adapter label not read |
| DC jack location | On a separate DC-jack / power board connected to J4 (16-contact FFC ±1, 0.5 mm) | Confirmed, photo |
| Barrel dimensions and polarity | Not confirmed. Commonly quoted third-party figure is about 3.5 x 1.35 mm, centre positive (unsourced; one mention is a Q&A on a 2nd-gen retail product page) | Unverified |
| Adapter model | Third-party listings name W18-015N1A (14 V 1.1 A) | Low reliability |

History of the input-voltage question: the earlier research could only state "15 W" and inferred 14 V at about 1.1 A by analogy with the 1st gen (whose adapter label reads 14 V / 1.1 A per EDN) and from marketplace listings, none of which specifically names the 2nd gen's own adapter. A Reddit snippet (not opened) claims most Google Nest products accept the same barrel connector and work with 14 V DC, with only the Max needing 24 V. The kernel device tree settles the voltage at 14 V. Until measured, the earlier advice to design the input stage for at least 12–16 V tolerance and a budget of about 15 W total still applies.

Also earlier inferred: the barrel jack is soldered to the mainboard or a small jack board. The photos show the separate jack board. Whether that board is passive (as on the 1st gen) and what the 16 conductors carry are not established.

On-board power (all lost with the mainboard):

- "Slap-down DC-DC converters from Silergy" (TechInsights).
- 2x SSM6J507NU P-channel MOSFETs (30 V / 10 A) (Hackaday).
- SoC core rails are PWM-controlled regulators (`SOC_VDDEE_PWM` GPIOE(0), `SOC_VDDCPU_PWM` GPIOE(1)); a brownout-protection init exists (Fuchsia).
- Two INA231 monitors with a shared alert line (see Sensors).
- A 2200 µF / 25 V electrolytic on the SoC side.

## Driver coverage

Android runs on the Linux kernel, so "driver" here means a Linux kernel driver, plus an Android HAL where one is needed. **No drivers have been built or tested.** Mainline status was read from `torvalds/linux` master or confirmed absent by a zero-hit code search. "Vendor" means Google's GPL kernel release for this product as mirrored at https://github.com/frederic/elaine-linux (Linux 4.9.244, last pushed 2022-06; a third-party mirror; the same code should be obtainable from Google's open-source release page for the product). Only directory listings, the device trees and one file were inspected; vendor driver source was listed, not read.

### Parts that stay in the enclosure

| Part | Status | Driver / source | Work needed |
|---|---|---|---|
| JD9364 / JD9365 panel driver IC | Mainline | `drivers/gpu/drm/panel/panel-jadard-jd9365da-h3.c` | Add a panel entry with our timings and init sequence from the Fuchsia tables. The driver already carries a K&D panel (`kingdisplay,kd101ne3-40ti`) and Radxa displays as models to copy. |
| Sitronix ST7703 panel driver IC (BOE variant) | Mainline | Mainline has an ST7703 panel driver | Needed only if the unit has the `TV070WSM_ST7703I` panel. Applicability of the mainline driver to this panel was not checked. |
| Goodix GT6853 touch | Vendor | Goodix GTX8 driver, GPL v2: `goodix_touch_gtx8` in the GPL tree; also seen in `Supernote-Ratta/kernel_a6x_a5x`. Fuchsia has an open BSD driver as a reference. | Port an out-of-tree driver to the target kernel. Mainline `goodix_berlin` only lists GT9916. Highest touch risk. |
| ams TCS3400 light sensor | Vendor | Mainline `tcs3472.c` does not support it. Google's GPL tree patches `drivers/iio/light/tcs3472.c` with `CONFIG_USE_TCS3400`. | Forward-port Google's patch to the mainline TCS3472 driver. Zephyr also has a TCS3400 driver, useful as a register reference. |
| Sensirion SHTC3 | Mainline | `drivers/hwmon/shtc1.c` | Device-tree node. |
| Infineon BGT60TR13C radar | None (kernel) | Infineon `sensor-xensiv-bgt60trxx`, Apache-2.0, platform-independent C | Run in userspace over `spidev` with GPIO for IRQ and reset. Presence detection logic is ours to build. |
| PDM microphones | Platform | Generic `dmic` codec is mainline; the PDM receiver is the compute module's | Depends on module choice. |
| Buttons, mute switch | Mainline | `gpio-keys` | Device-tree node. |
| Amber LED | Mainline | `pwm-leds` | Device-tree node. |

### Parts on the stock mainboard (lost; listed to guide part selection)

| Part | Status | Driver / source | Work needed |
|---|---|---|---|
| TI LP8556 backlight | Mainline | `drivers/video/backlight/lp855x_bl.c` | Reuse the part and copy the stock register values. |
| TI TAS5805M amplifier | Mainline | `sound/soc/codecs/tas5805m.c`; out-of-tree Sonocotta driver also exists | Stock tuning is not public; derive conservative EQ/limiter. |
| TI TAS5825M amplifier (the stock kernel's driver choice) | Vendor | Not in mainline; `tas5825m.c` is in Google's GPL tree | Port if this part is used. |
| Maxim MAX98357A amplifier (candidate) | Mainline | `sound/soc/codecs/max98357a.c` | None beyond a device-tree node; known start/stop pops. |
| NTC thermistors | Mainline | `ntc_thermistor` via an ADC | Needs an ADC on the new board. |
| TMP112 temperature sensors | Standard driver in the stock kernel; mainline not separately checked | `lm75` | Only if re-created on the new board. |
| TI INA231 power monitors | Standard driver in the stock kernel; mainline not separately checked | `ina2xx` | Only if re-created on the new board. |

Reusing the LP8556 and the stock amplifier part number on the new board keeps each on a mainline driver, with Google's backlight register values already published.

### Android layer above the kernel

Not verified; from general knowledge of AOSP.

- Display, touch, buttons: no HAL work. DRM and evdev are picked up directly.
- Backlight: needs the lights HAL pointed at the backlight sysfs node.
- Light and humidity sensors: need a sensors HAL that reads IIO/hwmon devices.
- Audio: needs an audio HAL configuration (tinyalsa/mixer paths) for the amplifier and PDM capture.
- Radar: no Android framework concept; expose it through a custom service or app.

The compute module's kernel version decides how much porting the vendor drivers need.

## What a replacement board must provide

Every off-board peripheral is on a standard interface (DSI, I2C, SPI, PDM, I2S-fed amplifier), so the functional design is straightforward; the unknowns are physical.

| Function | Requirement | Notes |
|---|---|---|
| Display | 4-lane MIPI DSI video mode, 600x1024 portrait at 44.2 or 49.4 MHz pixel clock; panel reset GPIO; two panel-ID strap inputs; the "panel power on" control if it proves to be a real signal | Software rotation by 90 degrees. Handle JD9364, JD9365 and possibly ST7703 variants. |
| Panel supplies | Whatever rails J12 carries | Unknown; measure. |
| Backlight | Boost LED driver for 3 strings, with enable. Reusing an LP8556 with the stock register values is the low-risk choice if the strings are exposed at J12. | String voltage/current unknown. |
| Touch | I2C at 400 kHz (device at 0x5D), interrupt input, reset output, touch supply | Logic level unknown. |
| Microphones | One PDM clock output, two PDM data inputs (3 channels) | Or a voice front end. Supply and mute-circuit behaviour unknown. |
| Speaker | Mono class-D amplifier fed by I2S, with enable and fault lines, and conservative protection | Speaker impedance and the third speaker wire unknown. |
| Light sensor | I2C (0x39) plus interrupt input | |
| Temp/humidity | I2C (0x70) | |
| Radar | 4-wire SPI at 25–30 MHz, interrupt input, reset output, 3V3 and 3V4 supply rails to the radar board, 1.8 V logic | FFC pin order unknown. |
| Buttons | GPIO inputs for volume up, volume down, mute sense (optionally the combined factory-reset line) | External pull-ups on stock board. |
| LED | One PWM output for the amber LED | Optional. |
| Power | Accept 14 V from the stock jack board via J4; generate all logic rails; about 15 W total budget | Input protection on the stock board is not documented. |
| Radios | Wi-Fi/BT on the module or a separate radio; two coax antenna jacks exist on the stock board | Thread (nRF52811 on the stock board) is lost unless deliberately restored. |
| Thermal monitoring | Optional NTC inputs via an ADC, or I2C temperature sensors | |
| Mechanical | Match the roughly 101 x 46 mm outline, mounting holes, connector positions and height limits | All must be measured. |

Buses: the stock board uses three I2C buses (sensors: 0x39, 0x70; touch: 0x5D; AV: 0x2C, 0x2D, 0x40, 0x49). Fewer could be used subject to I/O voltage domains, which are unknown.

What reuse delivers: the screen, touch, microphones, speaker, light sensor, temperature/humidity sensor, radar board and enclosure carry over. The amplifier tuning, far-field voice processing and radar algorithms do not; each must be rebuilt, substituted or dropped.

Design decisions specific to this model:

- Whether to reuse the LP8556 with Google's register values or design a different driver once the strings are characterised.
- Like-for-like amplifier (TAS5805M bridged) or a simpler common part, and how to protect the small driver without Google's tuning.
- Microphone path (direct PDM, voice front end, or replacement array).
- Radio and antenna handling, and whether to restore Thread.
- Whether the radar ships in version one as a presence sensor.
- Whether to share a schematic core with the 1st gen: the 2nd gen shares its panel family and bus layout and adds the radar, the GT6853 touch risk and the third microphone. A shared outline is unknowable until both boards are measured (the stock boards are about 101 x 46 mm and about 121 x 46 mm).

Recommended approach from the research: prove panel, touch and audio on an off-the-shelf module IO board through small flex adapter boards before committing to a full form-fit carrier, because panel bring-up failures present as a silent black screen.

## Open questions and measurements to take

No completed open project has swapped the mainboard of any smart display. The first milestone is a teardown-and-measure session on a real unit, ideally one still booting stock firmware so that rails and buses can be observed passively.

Connectors and pinouts:

1. J12: confirm the contact count (33 ±1 is a photo count) and pitch; count contacts at both ends of jumper flex `G653-01131-02` and at CN1 to resolve the 33-versus-39 discrepancy; read connector markings.
2. J12 pinout. Probe a stock mainboard with the display unplugged: grounds by continuity, LED anode and cathodes back to the LP8556 pins, I2C and reset lines back to known test points, DSI pairs by their matched routing (the five pad pairs beside J12), straps and rails.
3. J15 radar FFC pin order (test-pad labels are known, pin order is not).
4. J6, J5, J8 and J4 pinouts and exact contact counts; which board holds the volume buttons, SHTC3, TCS3400 and amber LED.
5. J7: what the three speaker wires are.
6. Exact connector part numbers for every connector.

Voltages and electrical:

7. Logic level of each bus (touch, sensors, AV, radar, PDM, buttons); banks can be 1.8 V or 3.3 V.
8. Panel supply rails and any bias rails; what kernel "panel GPIO index 1" drives.
9. Touch supply; strip-board (microphone) supply; sensor supply.
10. Radar board rails (3V3 and 3V4 expected from the pad labels).
11. Backlight: LEDs per string, string forward voltage and current. Measure the boost output and string voltage at the connector on a working unit before disassembly goes further (passive measurement only).
12. Speaker DC resistance and any markings.
13. Adapter label (voltage, current, model), barrel outer/inner diameter and polarity.
14. What the 16 conductors of J4 carry; whether the jack board is passive; whether a hidden USB/service port exists.

Component identity and location:

15. Panel variant in hand: read the panel label and flex markings (BOE suffix, K&D variant, driver IC); whether the Innolux and ST7703 variants shipped.
16. Amplifier chip marking: TAS5805M or TAS5825M.
17. Microphone count and positions (three versus two on the strip board; third PDM data line commented out in the kernel).
18. Whether the mute switch cuts microphone power or clock in hardware or only signals the SoC.
19. Location and bus of the four TMP112 sensors and four NTC thermistors.
20. SHTC3 versus "SHTV3": read the ID register.
21. Antenna construction and which coax jack serves which radio.

Mechanical:

22. Mainboard outline, mounting-hole positions, connector positions, component height limits, heat-spreader contact areas, positions of any board-mounted switches that poke through the enclosure.
23. Disassembly method (adhesive versus clips, screw types).

Software and desk tasks:

24. Whether the GT6853 works without the host loading a config blob.
25. Transcribe the panel init byte tables from Fuchsia (only the index file has been read) and archive the Fuchsia files at a pinned commit.
26. Compare other `elaine` device-tree board revisions; only `b4` was read.
27. Read the vendor driver sources (GTX8, TCS3400 patch, TAS5825M); they have only been listed.
28. Decode the LP8556 register values against TI's datasheet to recover current and boost settings (the source comments give meanings, but no independent decode was done).
29. Try to obtain panel datasheets: the paid TV070WSM-TG1 datasheet from Panelook (29 pages, Rev.P4) is the only known source of a real pin table for the BOE family; distributors (twscreen, eechain, enlcd) offer datasheets on inquiry.
30. Validate thermal headroom with the chosen module.
31. Regulatory position on operating the radar with a non-stock configuration.

Not searched or not viewed: YouTube (other than noting the teardown video exists), Reddit (the r/hardwarehacking thread "Nest Hub (2nd Gen) Teardown and Radar SPI decoding" is by the Hackaday author and was not opened), Japanese/Chinese teardown blogs, the full TechInsights report, fcc.report mirrors. FCC schematics, block diagram and theory of operation are under long-term confidentiality (metadata only).

## Sources

Google / Fuchsia (BSD-licensed board support; signal names appear to be schematic net names):

- nelson board driver directory: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/
- nelson.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson.h
- nelson-gpios.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-gpios.h
- nelson-i2c.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-i2c.cc
- nelson-spi.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-spi.cc
- nelson-audio.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-audio.cc
- nelson-light.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-light.cc
- nelson-buttons.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/nelson-buttons.cc
- nelson post-init.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/nelson/post-init/post-init.cc
- panel-config.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc
- display-panel.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h
- amlogic-display README: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/README.md
- gt6853-nelson.cc, December 2021 snapshot: https://go-mod-viewer.appspot.com/go.fuchsia.dev/fuchsia@v0.0.0-20211206140801-bbc9ce36f629/src/ui/input/drivers/gt6853/gt6853-nelson.cc
- astro backlight.cc (1st gen, register comparison): https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/astro/post-init/backlight.cc
- 1st-gen BOE init header (family reset timing reference): https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv070wsm-fitipower-jd9364-astro.h

Google GPL kernel (third-party mirror; Linux 4.9; `arch/arm64/boot/dts/amlogic/` and `arch/arm64/configs/`):

- https://github.com/frederic/elaine-linux

Teardowns and photos:

- Hackaday.io 2nd gen teardown project: https://hackaday.io/project/190147-google-nest-hub-2nd-gen-teardown
- Hackaday.io component list: https://hackaday.io/project/190147/components
- Hackaday.io radar PCB pinout log: https://hackaday.io/project/190147/log/216912-bgt60tr13c-radar-pcb-pinout
- Hackaday.io radar decoding log: https://hackaday.io/project/190147/log/217014-decoding-bgt60tr13c-project-soli
- Connector side of mainboard, all daughterboards attached: https://cdn.hackaday.io/images/6772551679376436399.jpg
- J12 close-up: https://cdn.hackaday.io/images/5872331679376498675.jpg
- SoC / heat-sink side: https://cdn.hackaday.io/images/9216291679376325164.jpg
- Radar board both sides, annotated pinout: https://cdn.hackaday.io/images/6514751679376803135.jpg
- Radar board + FFC + mainboard corner: https://cdn.hackaday.io/images/1670461679376915403.jpg
- Hackaday article, 2023-05-29: https://hackaday.com/2023/05/29/google-nest-hub-teardown/
- Teardown video (txyzinfo): https://www.youtube.com/watch?v=CcPzFpDd2_Q
- Electronics360 summary of the TechInsights teardown, 2021-08-11 (secondary; contains at least one apparent error): https://electronics360.globalspec.com/article/17053/teardown-google-nest-hub-2nd-gen
- FCC listing: https://fccid.io/A4RGUIK2
- FCC internal photos: https://fccid.io/A4RGUIK2/Internal-Photos/Internal-Photos-20200702-v1-Internal-Photos-5035982 (PDF: https://fccid.io/m/07ad8aaf6f2e26cc51c5cb66f053116c926b5cf32b7ccc54e8e6662bfa81cd47.pdf)
- EDN 1st-gen teardown (cross-reference only): https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/
- Facebook group post about the 2nd-gen screen cable (snippet only): https://www.facebook.com/groups/GoogleAssistant/posts/1429846610882402/

Product specifications and reviews:

- Google support spec page: https://support.google.com/googlehome/answer/7072284?hl=en
- Android Authority review (updated May 2023): https://www.androidauthority.com/google-nest-hub-second-generation-review-1210216/

Panel and connector data:

- Panelook TV070WSM-TG1: https://www.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_parameter_51122.html and https://m.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_overview_51122.html
- twscreen TV070WSM-TG1: https://www.twscreen.com/lcdpanel/24300
- eechain TV070WSM-TG1: https://eechain.com/en/product/detail/TV070WSM-TG1
- TV070WSM-NM0 (different sibling): https://m.panelook.cn/TV070WSM-NM0_BOE_7.0_LCM_parameter_cn_32259.html and https://ronboelectronics.en.made-in-china.com/product/QZzJFhVdbaWm/China-7-0-Inch-Tablet-LCD-Screen-Resolution-1024-600-Boe-TV070wsm-Nm0-Mipi-40-Pins.html
- Lenovo FCC report listing KD070D82-39TI-A004 and TV070WSM-TL2: https://fccid.io/m/0c706c2d3b04f1d71bb6fdeaca4258ebf14f0436a8d83122ba5047f2533b96a5.pdf
- KD070D82-39TI-A014 flex marking (marketplace comment): https://am.ozon.com/product/tachskrin-sensornoe-steklo-dlya-plansheta-atouch-q37-1529015875/
- Innolux P070ACB-DB1/DB2 siblings: https://mohotec.com/product/7-0-innolux-p070acb-db2/ , https://www.enlcd.com/products/19772 , https://www.yinjidz.com/products-detail.php?ProId=1893 , https://www.alibaba.com/product-detail/P070ACB-DB2-7-Inch-LCD-Touch_1601222037979.html
- Raspberry Pi forum on DSI panel bring-up: https://forums.raspberrypi.com/viewtopic.php?t=392157

Replacement-part listings (no FPC markings or pin counts in any):

- https://www.aliexpress.com/i/1005004924443271.html (2nd gen)
- https://he.aliexpress.com/item/1005013448231437.html (1st / 2nd gen)
- https://www.amazon.com/Home-Nest-Hub-Replacement-Generation/dp/B0CZ9G4HGD (2nd gen; states it does not fit the 1st gen)
- https://www.amazon.com/Replacement-Generation-Display-Digitize-Assembly/dp/B0D1GD6QRV (2nd gen)
- Reddit question about driving the screen from another board (could not be read): https://www.reddit.com/r/googlehome/comments/11m628w/google_nest_hub_screen_driver/

Radar:

- Infineon sensor-xensiv-bgt60trxx: https://github.com/Infineon/sensor-xensiv-bgt60trxx
- Infineon radar presence example: https://github.com/Infineon/mtb-example-psoc6-radar-presence
- Rust crate: https://docs.rs/bgt60trxx
- Infineon community threads: https://community.infineon.com/t5/Radar-sensors/BGT60TR13C/td-p/883939 and https://community.infineon.com/t5/Radar-sensors/BGT60TR13C-minimum-register-list/td-p/999623

Audio and touch:

- TI TAS5805M datasheet: https://www.ti.com/lit/ds/symlink/tas5805m.pdf
- Sonocotta TAS5805M driver: https://github.com/sonocotta/tas5805m-for-raspbian-paspberry-pi-zero
- Adafruit MAX98357A notes: https://learn.adafruit.com/adafruit-max98357-i2s-class-d-mono-amp/raspberry-pi-usage
- XMOS XVF3800 front end: https://www.seeed.cc/product/respeaker-mic-array-v2-0 and https://www.creatingsmarthome.com/index.php/2026/04/12/home-assistant-diy-voice-assistant-with-seeed-studio-respeaker-xvf3800/
- Goodix developer forum thread on a GT6853 Linux driver (snippet only): https://developers.goodix.com/en/bbs/detail/19f255b9cee04631a2b8c480f918b9f7

Power adapter (low reliability):

- https://www.amazon.com/Dysead-Replacement-Supply-Adapter-Compatible/dp/B0D6LYXJ17
- https://shopee.sg/14V-1.1A-15W-AC-DC-Power-Supply-Charger-For-Google-Home-Nest-Hub-Mini-Gen-2-2nd-W18-015N1A-G1015-G1030-Smart-speaker-i.860304842.29710786136
- https://www.aliexpress.com/i/1005008505390379.html
- Target Q&A on a 2nd-gen product page: https://www.target.com/p/google-nest-hub-with-sleepsense-sand/-/A-80209779?showOnlyQuestions=true
- Reddit snippet on shared barrel connector and 14 V: https://www.reddit.com/r/googlehome/comments/ep5ch8/how_on_earth_to_get_a_nest_hub_max_power_cable/

Project source documents consolidated here:

- `reports/Nest Hub Android hardware research.md`
- `reports/Display connector pinout findings.md`
- `reports/Driver coverage matrix.md`
- `reports/Stock kernel device tree findings.md`
- `research_notes/Nest Hub Android hardware research/nest_hub_gen2_hardware.md`
- `research_notes/Display connector pinouts/pinout_7inch.md`
- `research_notes/Display connector pinouts/connector_photo_evidence.md`
