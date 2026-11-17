# Nest Hub Max

The Nest Hub Max is Google's 10.1-inch smart display (2019) with a camera, two microphones and a three-driver, three-amplifier audio system running from 24 V. The logical design is well documented: Google's open-source Fuchsia board driver ("sherlock") and GPL Cast kernel device tree ("newman") name almost every peripheral, its bus, I2C address, control lines, display timings, DSI init sequences and backlight register values. The physical design is poorly documented: there is no component-level teardown, only a 10-page FCC internal-photo exhibit viewed at low resolution. The biggest unknowns are every connector pinout (including the 51-pin display flex), all supply and logic voltages, the ±5.5 V panel bias sequencing, speaker impedance, the destinations of most mainboard connectors, and a usable driver and image pipeline for the IMX227 camera.

Evidence levels used throughout:

- **Confirmed**: stated for the exact part by a source that had the datasheet, read from Google's source code, or read directly from a photo. Photo readings carry their own confidence (high / medium / low) where the source gave one.
- **Sibling**: from a related part. Not safe to wire from.
- **Inferred**: reasoning from other facts. Must be measured.

SoC pin names such as GPIOZ_1 are Amlogic T931 pins on the stock board. They matter only as a list of signals that exist and their polarity, not as numbers to copy.

No drivers have been built or tested, and nothing from the kernel device tree or Fuchsia source has been checked against hardware.

## Identification

| Item | Value | Source |
|---|---|---|
| Product | Google Nest Hub Max, 2019 | Fuchsia notes |
| Model number | H2A | FCC, WikiDevi |
| FCC ID | A4RH2A | fccid.io |
| Fuchsia board name | sherlock | Fuchsia tree |
| Cast kernel codename | newman | Google GPL kernel tree |
| Stock SoC | Amlogic T931 (4x Cortex-A73 + 2x Cortex-A53, Mali-G52, NPU) | WikiDevi; T931 also seen in FCC photos p4/5 |
| Mainboard silkscreen | `G651-01768-05` | Seen, FCC p6 |
| Mainboard size | About 117 x 62 mm by the photo ruler (likely a little larger in reality, because the ruler lies in front of the board); near-rectangular with a small step at the bottom-left. Low-medium confidence. A separate low-resolution viewing of FCC p4/5 gave "about 11 cm wide". | Seen, FCC p6 and p4/5 |
| External dimensions | 250.1 x 182.55 x 101.23 mm | GadgetGuy |
| Weight | 1.365 kg (GadgetGuy); 2.91 lb / 1.32 kg (Wccftech). Sources disagree. | |
| Other Google part numbers | AUX PCB `G651-01948-03`, webcam `G653-01357-02` (spare-parts listing; no LCD part number given) | joesge.com |

Codename cross-reference for the family:

| Model | Model number | FCC ID | Fuchsia board | Cast codename | Stock SoC |
|---|---|---|---|---|---|
| Nest Hub 1st gen | H1A | A4RH1A | astro | estelle | Amlogic S905D2G |
| Nest Hub 2nd gen | GUIK2 | A4RGUIK2 | nelson | elaine | Amlogic S905D3G |
| Nest Hub Max | H2A | A4RH2A | sherlock | newman | Amlogic T931 |

How to tell the Max from the other two models:

- 10.1-inch 800x1280 panel (marketed as 1280x800); the other two are 7-inch 600x1024.
- It is the only model with a camera (Sony IMX227) and a mic + camera privacy switch.
- 24 V / 1.25 A (30 W) adapter; the 7-inch models use 14 V. The barrel plug reportedly matches other Google Nest products physically while the voltage differs, so adapters must not be interchanged.
- Three speaker drivers and three TAS5720 amplifiers; the 7-inch models have one driver and one amplifier.
- Amber and green LEDs; the 7-inch models have amber only.
- No Soli radar (2nd gen only) and no temperature/humidity sensor (2nd gen only).
- Mainboard silk `G651-01768-05`; 1st gen is `G651-01770-05 REV01`, 2nd gen is `G651-04810-06 REV B`.

Kernel device tree files read for this model: `newman-b4.dts`, `newman-common.dtsi`, `newman-panel.dtsi`, `newman-bx_defconfig` (Linux 4.9, `arch/arm64/boot/dts/amlogic/` and `arch/arm64/configs/`). `b4` is the latest board revision in the tree. Other revisions were not compared.

## Internal layout and connectors

### State of physical evidence

- No component-level teardown (iFixit or otherwise) of the Max exists publicly. A Reddit post (r/googlehome, 2021, snippet only) asks whether anyone has taken the Max apart and notes that only non-Max teardowns exist.
- The only public internal imagery is the FCC "Internal Photos rev" exhibit: 10 pages, 1.4 MB PDF, public since 2019-11-03. FCC schematics, block diagram and theory of operation are under long-term confidentiality (metadata only).
- FCC page guide: p1 = removed display assembly and chassis; p4 bottom = SoC side with ruler; p5 bottom = connector side with shields; p6 bottom = connector side, shields off, ruler (best); p7 = close-ups; p8 = DC-jack board; p9 = camera module; p10 = display rear.
- The exhibit was viewed in a browser at limited resolution, not downloaded. One viewing covered pages 1, 2, 4, 5 and part of 6 (pages 3 and 7–10 not reviewed because the viewer stopped rendering); a second viewing produced the p6, p9 and p10 observations below. Resolution was too low to read connector designators or count pins reliably. Photo-scaled dimensions are estimates (about ±5–10%); verify with calipers before committing a PCB.
- Other internals sources, none of which documents the display connector:
  - mydigit.cn teardown thread, 2023-10-01, 22 numbered photos (login-gated; text only read): mainboard, T931, Micron 2 GB RAM, Samsung 4 GB eMMC, 30 W woofer.
  - TechInsights Deep Dive Teardown DDT-1909-802 (paywalled, 2019-11-26).
  - YouTube "Opening a Google Nest Hub Max to replace screen [FAILED ATTEMPT]" (David Mudge); not watched or transcribed.
  - Reddit r/fixit "Trying to locate Google Nest Hub Max display ribbon cable" (snippet only): wiggling the display ribbon restores the image; backlight on but no image.
  - Reddit r/googlehome teardown questions (snippets only).
- No GitHub or Hackaday hardware-reuse project specific to the Max was found. Hackaday's 2023 teardown covers the 2nd gen, not the Max. No completed open project has swapped the mainboard of any smart display.

### Boards and assemblies

| Assembly | What is known | Evidence |
|---|---|---|
| Mainboard | Silk `G651-01768-05`, about 117 x 62 mm. About 8 gold-ringed mounting holes including one near the board centre, plus many gold EMI pads. | Seen, FCC p6 |
| Mainboard, SoC side | SoC + memory under a shield frame, a row of inductors, two large electrolytic cans. Seen only at thumbnail scale; not inspected closely. Amlogic T931 and Samsung eMMC identified on p4/5. | Seen, FCC p4/5 |
| Mainboard, connector side | nRF52840 with coax jack; three identical QFNs near the speaker header (audio amplifiers, Inferred); Wi-Fi chip in a shield outline; two coax jacks; a QFN + "100" inductor beside the display connector (backlight driver, Inferred). | Seen, FCC p6 |
| Display assembly | Black back, one wide orange flex near the centre (about 18–19 mm wide on p10), a small green PCB strip on one edge (p1). Two small green microphone boards flank the camera hole at the top, each with its own connector (p10). A second small orange flex patch near the bottom edge was not identified. | Seen, FCC p1, p10 |
| Camera module | Rigid black board about 23 mm wide with one ZIF connector, staggered tails, about 25 contacts (±2, low-medium confidence) at about 0.3 mm effective pitch. Connects by a separate flex, not a board-to-board mezzanine. | Seen, FCC p9 |
| DC-jack board | A separate DC-jack board is shown on FCC p8. Its connector and cable to the mainboard are not characterised. | Seen (page guide), FCC p8 |
| Antenna boards | Visible in the chassis on p1. Stock Wi-Fi uses MHF4 antenna connectors (WikiDevi). | Seen, FCC p1; WikiDevi |
| Chassis cabling | An orange flex plus several blue-tabbed white FFCs enter the top of the mainboard. | Seen, FCC p1 |

The base (speaker enclosure) and the display head are separate volumes. Where the mainboard sits (behind the display or in the base) was not confirmed in text sources.

Replacement display assemblies are sold as one bonded LCD + digitizer + bezel unit with a front camera window (eBay), which suggests the display stack is glued and the practical service boundary is the whole front assembly (Inferred). One listing notes that bending the flex over 90 degrees results in a black display.

### Mainboard connectors

No designators were legible. Positions are as seen in the FCC p6 view. All pin counts are photo estimates.

| Position | Likely destination | Style | Contacts (confidence) |
|---|---|---|---|
| Right edge, vertical, white housing, about 16–18 mm long | Display (same white-ZIF look as the other models; QFN + "100" inductor beside it) | ZIF FPC | About 27 solder tails visible on one side at about 0.5–0.6 mm spacing. Either roughly 27–30 pins at 0.5 mm, or roughly 51–55 pins at 0.3 mm staggered. **Unresolved, low confidence.** |
| Top-centre, large black header | Speakers (6 filter-part pairs beneath it) | Wire-to-board, right-angle | 8 (medium-high), about 1.25–1.5 mm pitch |
| Top-right, black | Unknown (camera or top boards) | ZIF FPC | About 23–24 (low-medium), about 0.5 mm |
| Left edge, small black | Unknown | ZIF | About 5–6 (low) |
| Bottom edge, 4 connectors left to right | Unknown individually (DC-jack board, mic boards, etc.) | ZIF FFC, black | About 10, about 6, about 16–20, about 8–10 (all low) |
| Coax jacks | Antennas (one beside the nRF52840, two more) | Coax | – |

A separate low-resolution viewing of p4/5 described the connector side as carrying "roughly eight FPC/FFC connectors: one fine-pitch near top centre, one long vertical one on the right edge, several small ones along the bottom" and could not tell which is the display connector. The table above (seven FPC/FFC positions plus the speaker header) is from the better p6 view; the two descriptions do not fully agree on count or on what sits at top centre.

Inter-board cable and connector families (part numbers, pitch, contact side) are not documented anywhere.

### What leaves with the mainboard

| Item | Status | Basis |
|---|---|---|
| SoC, RAM, eMMC | Lost | On mainboard |
| Wi-Fi/BT (Broadcom BCM43458) | Lost | On mainboard (Wi-Fi chip in shield outline, Seen) |
| Thread radio (Nordic nRF52840) | Lost | On mainboard (Seen, FCC p6) |
| Three TAS5720 amplifiers | Lost | Three identical QFNs near the speaker header (identity Inferred from photo; part from Fuchsia) |
| Camera regulators (analog and digital supplies) | Lost | Enable GPIOs on the SoC show they are generated on the mainboard (Inferred) |
| LP8556 backlight driver | Lost (Inferred) | Panel datasheet summary says no driver on the module; QFN + inductor pattern beside the display connector in FCC p6. The earliest notes treated its location as uncertain. |
| "lcd_bias" IC at 0x3E | Lost (Inferred) | Panel needs external AVDD/AVEE; later notes place the bias IC on the mainboard. Chip marking not identified. The earliest notes treated its location as uncertain. |
| Regulators, input protection, PMIC | Lost | Parts not documented |
| TMP112 sensors, BMA456 accelerometer (prototype revisions) | Not relevant | Removed or disabled in the `b4` revision |

| Item | Stays in the enclosure? | Basis |
|---|---|---|
| Display panel and backlight LEDs | Yes | Separate assembly |
| Touch controller (FT5726) | Probably (Inferred) | Most likely chip-on-flex on the touch sensor flex; unconfirmed |
| Microphones | Yes | Two small mic boards on the display rear (Seen, FCC p10). Earlier notes listed mic location as unknown. |
| Camera module | Yes | Separate module on its own flex (Seen, FCC p9) |
| Speakers | Yes | In the enclosure; wire-to-board header on mainboard |
| DC jack | Yes | Separate DC-jack board (FCC p8). Earlier notes listed this as unknown. |
| Antennas | Probably (Inferred) | MHF4 connectors imply separate coax-fed antennas; antenna boards seen in chassis |
| TCS3400 light sensor | Unknown | Probably on the camera/mic/sensor flex behind the front bezel (Inferred, unconfirmed) |
| Buttons, privacy switch, LEDs | Unknown | Not documented; may be on the mainboard edge or a separate flex/board |
| Thermistors | Unknown | Location of each not documented |

## Display

### Panel variants

| Variant | Panel | Driver IC (DDIC) | Touch IC |
|---|---|---|---|
| 1 | BOE TV101WXM-AG0 | Fitipower JD9364 | FocalTech FT5726 |
| 2 | Innolux P101DEZ-DD0 | Fitipower JD9364 | FocalTech FT5726 |
| 3 | BOE TV101WXM-AG0 | Fitipower JD9365 | FocalTech FT5726 |

Source: Fuchsia `display-panel.h` (Confirmed). Innolux + JD9365 is rejected by the stock software as unsupported.

The kernel device tree panel file for the Max lists entries `TV101WXM`, `TV101WXM_FT`, `G101B158`, `G101B158_FT`, plus 7-inch entries carried over. The source does not say how `G101B158` relates to the Innolux P101DEZ-DD0 named by Fuchsia; the two sources name the second panel differently.

All variants are 10.1-inch, 4-lane MIPI DSI video mode, natively **portrait 800x1280 at 60 Hz**, mounted landscape. The marketing figure "10-inch HD touchscreen 1280x800" (Google Store) describes use, not scan order. The new system must rotate the framebuffer 90 degrees.

### Panel identification

The stock board reads two strap pins on the display module (Confirmed, Fuchsia `post-init.cc`, `sherlock-gpios.h`):

| Module pin | Meaning | Stock SoC pin |
|---|---|---|
| ID0 | Vendor: 0 = BOE, 1 = Innolux | GPIO_PANEL_DETECT = GPIOH_0 |
| ID1 | DDIC: 0 = JD9365, 1 = JD9364 | GPIO_DDIC_DETECT = GPIOH_2 |

A replacement board should read these straps to select the init table, or the builder must identify the variant in hand. Where the ID pins sit on the connector is not known.

One marketplace listing (Facebook, snippet only) gives the flex marking as "TV101WXM-AG0-P0", "Flex Central 51 Pines", compatible with the Nest Hub Max. This is the only FPC marking found and an independent confirmation of the part number and 51-pin figure.

### Timings

| | BOE TV101WXM | Innolux P101DEZ |
|---|---|---|
| H active | 800 | 800 |
| H front porch / sync / back porch | 20 / 20 / 50 | 46 / 24 / 20 |
| V active | 1280 | 1280 |
| V front porch / sync / back porch | 20 / 4 / 20 | 20 / 4 / 20 |
| Pixel clock | 70.702 MHz | 70.702 MHz |
| Refresh | 60 Hz | 60 Hz |
| Sync polarity | hsync/vsync negative, progressive | not transcribed |

Source: Fuchsia `panel-config.cc` (Confirmed). The kernel device tree entry `TV101WXM_FT` agrees: 800 x 1280 active, 890 x 1324 total, hsync 20 / back porch 50, vsync 4 / back porch 20, 70.7016 MHz.

The Fuchsia source cites a "BOE TV101WXM-AG0 spec, Section 6.1 Signal Timing Spec, page 18" and JD9364/JD9365 datasheets (section 11.3). Those datasheets exist but are not public in the tree.

### DSI link

- 4 data lanes (`dphy_data_lane_count = 4`) on all three configs. Burst video mode per the kernel device tree.
- Fuchsia sets `maximum_dphy_clock_lane_frequency_hz = 283'000'000`. Source comments note this exceeds the JD936x limit of 250 MHz clock / 500 Mbps per lane and the BOE spec typical of 250 MHz. The kernel device tree gives 566 Mbit/s maximum per lane, which is the same cap.
- Inferred operating point: 4 lanes x 24 bpp at 70.7 MHz needs roughly 424 Mbps per lane (about 212 MHz clock), within the JD936x 500 Mbps per lane limit. The 283 MHz figure is a cap, not the operating point.
- Display bandwidth does not discriminate between compute modules.

### Supplies

| Rail | Value | Evidence |
|---|---|---|
| VDDIO (logic) | Listed as 3.3 V | Confirmed at distributor-summary level only (BOE TV101WXM-AG0 datasheet Rev.P0 summaries). **Unverified**: 3.3 V is unusual for a JD936x panel, which normally takes 1.8 V IOVCC, and the sibling TV101WXM-NLB uses VDDIO 1.8 V + VCC 3.3 V. Measure on a working unit. |
| AVDD | +5.5 V, external | Confirmed at distributor-summary level |
| AVEE | −5.5 V, external | Confirmed at distributor-summary level |
| Backlight | Raw LED strings, no driver on the module | Confirmed at distributor-summary level |

None of the three summary sources is the datasheet itself; all repeat one Panelook-derived data set that Panelook says is based on datasheet Rev.P0.

**± bias supply ("lcd_bias", I2C 0x3E).** Fuchsia lists an I2C device named "lcd_bias" at 0x3E on bus I2C3 with no part number (Confirmed that the device exists). Its identity is Inferred:

| Candidate | 7-bit address | Rails | Address evidence |
|---|---|---|---|
| TI TPS65132 | 0x3E | VPOS +4.0 to +6.0 V, VNEG −4.0 to −6.0 V, 0.1 V steps; default ±5.4 V (most variants) or ±5.0 V ("0" variants); ENP/ENN enable pins | Datasheet: "The device 7-bit address is 3E" |
| SGMicro SGM3804 | 0x3E | Single-inductor positive/negative dual output | Datasheet search snippet: "SLAVE ADDR. (3Eh)" |
| Kinetic KTD2151 | 0x3E | VPOS/VNEG, defaults ±5.0 V on the base part, other order codes up to ±5.6 V; vendor page says programmable +4.0 to +5.2 V / −4.0 to −5.2 V | Datasheet: "The 7-bit slave device address is 0111110 binary (or 3Eh)" |
| Orient-Chip OCP2131 | Not verified (fetched PDF had no address information) | +4 to +6 V / −4 to −6 V per snippet | – |

Inference: address 0x3E plus a panel needing AVDD +5.5 V / AVEE −5.5 V strongly suggests a TPS65132-class dual-output bias IC on the mainboard (TPS65132 or a register-compatible clone such as SGM3804). ±5.5 V is inside the TPS65132 range but is not its power-on default, which fits the stock firmware needing to write to it over I2C. The actual chip marking on the H2A board was not identified. A replacement mainboard must supply AVDD/AVEE itself. A DTS-style search for bias regulators at reg 0x3e returned nothing.

### Panel parameters (BOE TV101WXM-AG0, distributor summaries)

| Item | Value |
|---|---|
| Interface | MIPI, 4 data lanes |
| Connector pin count | 51 pins |
| Backlight | WLED 6S6P (36 LEDs), no driver on module, 15K h life |
| Touch | Module is an "Assembly" with PCAP touchscreen |
| Luminance / contrast | 400 cd/m², 1000:1 |
| Active area | 135.36 x 216.576 mm |
| Outline | 142.32 x 228.2 x 4.3 mm (panelcd); twscreen lists 245.2 x 164 mm (probably including cover glass). Sources disagree. |
| Mass-production date / status | Q2 2018, discontinued (panelcd); twscreen says "In Production". Sources disagree. |

Not found for this panel: connector part number, pitch, pin order, LED forward voltage and current, ID pin positions, reset pin position, whether touch I2C/INT/RST are among the 51 pins.

**Innolux P101DEZ-DD0:** nothing found. An exact-phrase search returned zero results. The closest hit is the P101DEZ-3Z3 (Sibling, and a poor one): 800x1280, MIPI 4 lanes, FOG (no backlight), driver IC ER88577 (not JD9364), no pin count or pinout on the page.

### Init sequence source

Full DSI init sequences are published in Fuchsia, one header per variant (297 / 264 / 376 lines):

- `boe-tv101wxm-fitipower-jd9364.h`
- `boe-tv101wxm-fitipower-jd9365.h`
- `innolux-p101dez-fitipower-jd9364.h`

Sequence shape (BOE variants): sleep 10 ms; LCD reset high 30 ms, low 10 ms, high 30 ms; read register 0x04 (3 bytes, panel ID); sleep 10 + 120 ms; then generic short writes (2 parameters) starting `E0=00, E1=93, E2=65, E3=F8, 80=03, E0=01 ...` (JD936x page-select style), ending `E0=00` then DCS 0x29 (display on).

The kernel device tree panel file carries the same data as the Fuchsia tables (stated for the 2nd gen entries; the Max entries were read for timings and power steps).

Having a trusted init sequence, timings and D-PHY configuration removes what Raspberry Pi engineers describe as the hardest part of bringing up an unknown DSI panel, where failure shows only as a black screen with "almost no debug information" (Raspberry Pi forum). The Fuchsia files should be archived at a pinned commit, since nothing guarantees they stay on the main branch.

### Power and reset sequence

- LCD reset line: GPIO_LCD_RESET = GPIOH_6. The Fuchsia display driver takes only this GPIO ("gpio-lcd-reset").
- Fuchsia defines rail power-on/off ordering as `kLcdPowerOnSequenceForAstroSherlockNelson` / `...PowerOff...` in `panel-config.cc` (shared with the 7-inch models). The contents of that table were not transcribed, and the 1st gen notes could not locate its definition in the files read.
- The kernel device tree resolves the previously open question of the sequence on the SoC side: for the Max 10.1-inch entries, power-on is only "start DSI signal", then the 30 / 10 / 30 ms reset pulse (high / low / high), then the init table. Power-off holds reset low for 100 ms. No multi-rail sequence appears in the device tree.
- **The ±5.5 V bias supply does not appear in the kernel device tree**, so it is presumably enabled earlier, by the bootloader. Its sequencing still has to be measured.
- Sibling only (TV101WXM-NLB, different panel): power-on order VDDIO → VCC → RESX → LP11/init → video → VLED (T1 ≥ 5 ms, T4 ≥ 10 ms, T5 ≥ 100 ms). Do not assume this applies.

## Display connector and pinout

**No pin-by-pin assignment for the Nest Hub Max panel is public.** The exact datasheets (BOE TV101WXM-AG0 Rev.P0, Innolux P101DEZ-DD0) are not publicly readable. The kernel device tree and Fuchsia source describe the SoC side only and do not give connector pin numbers.

### What is known

| Item | Value | Evidence |
|---|---|---|
| Panel interface pin count | 51 pins | Confirmed (distributor summaries of BOE datasheet Rev.P0; Facebook listing snippet "51 Pines") |
| Panel datasheet connector part | Not found | – |
| Pitch, pin-1 orientation | Not found | – |
| Mainboard display socket | White ZIF FPC on the right edge, about 16–18 mm long; contact count unresolved (about 27–30 at 0.5 mm, or about 51–55 at 0.3 mm staggered) | Seen, FCC p6, low confidence |
| Display flex width | About 18–19 mm | Seen, FCC p10 |
| Signals that must be on the display path | 4 DSI data pairs + clock pair, LCD reset, two ID straps, VDDIO, AVDD, AVEE, LED anode and six cathode strings (6S6P) | Derived from the facts above; arrangement unknown |
| Display and touch on one flex | Inferred shared: only one large flex is visible leaving the panel on FCC p10 | Inferred |
| Touch on its own tail | A mydigit.cn teardown (text only) says the touch ribbon was torn while prying the screen off ("触摸排线搞断了"), which distinguishes a touch flex from the rest | Inferred, weak |

### Open discrepancies

- **51 pins at the panel versus an unresolved socket count on the mainboard.** The 0.3 mm staggered reading (51–55) would match the panel figure; the 0.5 mm reading (27–30) would not. Not resolvable from the photo.
- **Shared flex versus separate touch tail.** The photo reading infers one flex; the mydigit.cn text hints at a separate touch ribbon. The TV101WXM-AG0 is sold as an LCD + PCAP "Assembly" with a single 51-pin figure, and whether the 51 pins include touch is not stated. An unidentified second small orange flex patch near the bottom edge of the display rear (FCC p10) is also unexplained.
- **VDDIO 3.3 V** is as listed but unverified (see Display, Supplies).

### Sibling pinout: BOE TV101WXM-NLB (Sibling. Do not wire the Max panel from it.)

A full pinout was found for the sibling BOE TV101WXM-NLB (35-page Lenovo "10WX LCD HADS" approval sheet, Rev 1.0, 2021-03-30). It is recorded for BOE conventions only.

Why it does not transfer:

| | TV101WXM-NLB (Sibling) | TV101WXM-AG0 (Nest Hub Max) |
|---|---|---|
| Pins | 39 | 51 |
| Driver IC | Himax HX8252-A02 | Fitipower JD9364/JD9365 |
| Rails on connector | VCC 3.3 V + VDDIO 1.8 V; VSP/VSN made on the panel FPCA (charge pump) | VDDIO + external AVDD +5.5 V / AVEE −5.5 V |
| Backlight | 7S4P, 28 LEDs, 21 mA/string, Vf 2.7/2.9/3.2 V per LED | 6S6P, 36 LEDs |
| Touch | None (bare LCM) | PCAP assembly |
| Luminance | 350 nit | 400 nit |

Sibling mating connector: Hirose FH26W-39S-0.3SHW (39 positions, 0.3 mm pitch), "filled by supplier".

Sibling pin table (TV101WXM-NLB only):

| Pin | Symbol | Description |
|---|---|---|
| 1–3 | NC | |
| 4 | FB4 | LED cathode string 4 |
| 5 | FB3 | LED cathode string 3 |
| 6 | FB2 | LED cathode string 2 |
| 7 | FB1 | LED cathode string 1 |
| 8 | NC | |
| 9–11 | VLED | LED anode |
| 12–13 | NC | |
| 14 | LEDPWMOUT | PWM output (CABC) |
| 15 | ID1 | 3.3 V |
| 16 | RESX | Reset (in) |
| 17 | ID2 | 0 V / GND |
| 18 | NC | |
| 19–21 | VCC | 3.3 V (3.1–3.5 V) |
| 22–23 | VDDIO | 1.8 V (1.7–1.9 V) |
| 24 | GND | |
| 25 / 26 | MIPI_D3_P / MIPI_D3_N | |
| 27 | GND | |
| 28 / 29 | MIPI_D2_P / MIPI_D2_N | |
| 30 | GND | |
| 31 / 32 | MIPI_CLK_P / MIPI_CLK_N | |
| 33 | GND | |
| 34 / 35 | MIPI_D1_P / MIPI_D1_N | |
| 36 | GND | |
| 37 / 38 | MIPI_D0_P / MIPI_D0_N | |
| 39 | GND | |

Other sibling data: IVDDIO 20 mA typical, IVCC 49 mA typical (white); timing 970 x 1364 total, 79.4 MHz pixel clock, 467 Mbps per lane (differs from the 70.7 MHz known for the Max).

Value of the sibling: general BOE conventions only (0.3 mm-pitch Hirose FH26-class connector, GND-separated lane pairs, one cathode pin per LED string, three anode pins). It is Inferred, low confidence, that the AG0 follows similar grouping.

Other TV101WXM siblings seen in listings: NL1 (39 pins), NU5, NL3, NL9, NLA, LL0, NA0, NF0, NU4, TW0. None was found with 51 pins or a readable datasheet. Earlier searches also turned up -NP1 (eDP, 30-pin) and -NM0 ("40 pins MIPI 4 data lanes"); these must not be assumed to match.

### Replacement-part listings

No listing text gave a panel model, FPC marking, pin count or connector type, other than the Facebook snippet above. Listings checked: two eBay items (one text-checked, one not read), Amazon (snippet only), joesge.com (part numbers for other parts only), AliExpress (snippet only).

## Backlight

### LED arrangement

- WLED 6S6P: 6 LEDs in series x 6 parallel strings = 36 LEDs, no driver on the module (Confirmed at distributor-summary level).
- The kernel device tree configures the LP8556 for "6-phase, 6 drivers", matching the 6S6P figure. String count was previously an open question; this settles it.
- Six strings match the six current sinks of the LP8556 (Inferred).
- LED forward voltage and string current are not found. Inferred only: 6S at about 3 V per LED implies roughly 17–20 V anode voltage. Do not size anything from this without measuring.
- Fuchsia `backlight_max_brightness = 350.0` nits (same table for BOE and Innolux). The panel datasheet summary lists 400 cd/m². These describe different things (software maximum versus panel luminance) and are both recorded.

### Driver

- TI LP8556 at I2C 0x2C on stock bus I2C3 (shared with the camera sensor at 0x36 and "lcd_bias" at 0x3E). Confirmed, Fuchsia `sherlock-i2c.cc`.
- Backlight enable: GPIO_BACKLIGHT_ENABLE = GPIOA_10.
- Location: Inferred to be on the mainboard beside the display connector (QFN + "100" inductor pattern in FCC p6, same arrangement as the 2nd gen where the LP8556 was read from its marking). On that reading the LED anode and cathode strings run through the display connector.

### LP8556 register values

| Register | Fuchsia (`post-init/backlight.cc`) | Kernel device tree (Max) | Meaning, per kernel source comments |
|---|---|---|---|
| 0x01 Device Control | 0x85 | 0x85 | Brightness by I2C register only, backlight enabled |
| 0xA2 (CFG2) | 0x20 | 0x20 | Undervoltage protection, 2.5 V |
| 0xA3 (CFG3) | 0x32 | not listed for the Max | 50 ms slope, 12-bit resolution (meaning given for the 2nd gen entry) |
| 0xA5 (CFG5) | 0x04 | 0x04 | 9.6 kHz PWM; 6 phases |
| 0xA7 (CFG7) | 0xF4 | 0xF4 | Boost inductor current limit 1.6 A |
| 0xA9 (CFG9) | 0x60 | 0x60 | Maximum boost voltage 25 V |
| 0xAE (CFGE) | 0x09 | 0x09 | LED fault threshold 3 V |

The two sources agree except for 0xA3: Fuchsia writes 0x32 on the Max; the kernel device tree table shows no 0xA3 entry for the Max. The Max backlight node for the second panel vendor was only skimmed in the kernel tree.

Decoding these values against the TI datasheet was previously listed as not done; the kernel source comments now give the meanings above. They have not been independently checked against the LP8556 datasheet. The Max table has no current-setting registers (0xA0/0xA1 are written only on the 1st gen), so maximum string current for the Max is still not known from these sources.

Reusing the LP8556 on the new board means copying these values, and keeps the backlight on a mainline driver.

## Touch

| Item | Value | Source |
|---|---|---|
| Controller | FocalTech FT5726 | Fuchsia `post-init/touch.cc`, node "focaltech-touch-38" |
| I2C address | 0x38 | Fuchsia, kernel device tree |
| Stock bus | I2C2 (SDA/SCL = GPIOZ_14/15); only device on that bus; 400 kHz | Fuchsia `sherlock-i2c.cc`; kernel device tree |
| Interrupt | GPIO_TOUCH_INTERRUPT = GPIOZ_1 | Fuchsia, kernel device tree |
| Reset | GPIO_TOUCH_RESET = GPIOZ_9 | Fuchsia, kernel device tree |
| Resolution / touches | 800 x 1280, 5 touches | Kernel device tree (`fts` node) |
| Firmware | `needs_firmware = true`: the stock driver uploads firmware | Fuchsia |
| Panel metadata | The touch driver also receives the panel type, so firmware/config may differ per panel vendor | Fuchsia |

- Touch needs four signals plus power: SDA, SCL, INT, RESET.
- Inferred: the controller is most likely chip-on-flex on the touch sensor flex (typical for FT5x26 modules), which would make it reusable. Unconfirmed.
- Inferred: "needs_firmware" means the stock OS can (re)flash controller firmware. A unit that has already run stock software should retain working firmware in the controller's flash, so a new host may not need to upload any. Verify by reading the chip and firmware ID registers.
- Vendor driver in Google's GPL tree: `focaltech_touch`, with configs for FT3427 and FT5726. Source was listed, not read.
- Mainline driver: `edt-ft5x06.c`, compatible `evervision,ev-ft5726` (see Driver coverage). Mainline does not upload firmware, so it relies on the controller keeping its flashed firmware. FT5726 support in this driver was previously unverified; the driver coverage check confirmed the compatible string exists.
- Not public: touch connector, pinout, supply voltage (1.8 V versus 3.3 V I/O), and whether touch shares the display FPC or has its own tail (see Display connector and pinout).

## Audio

### Microphones

| Item | Value | Source |
|---|---|---|
| Count | 2 far-field digital PDM microphones | Fuchsia `sherlock-audio.cc` (`number_of_channels = 2`); GadgetGuy "Mic: 2 x far-field" |
| Wiring | One PDM clock, one data line (only one data input is muxed) | Fuchsia |
| Stock pins | DCLK on GPIOA_7, DIN0 on GPIOA_8 | Fuchsia |
| Clock config | `sysClockDivFactor = 4`, `dClockDivFactor = 250` | Fuchsia |
| Location | Two small green mic boards flank the camera hole at the top of the display rear, each with its own connector | Seen, FCC p10 |

- Inferred: two mics on one data line means one is strapped left and the other right (standard stereo PDM pair). A new board needs only PDM CLK + one DATA input, or an I2S/PDM codec.
- The earlier text-only notes guessed the mics were on a separate board near the top bezel beside the camera; the FCC p10 photo supports that.
- Not documented: mic part numbers, PDM clock frequency actually used, supply voltage, and the connector/flex carrying them (which mainboard connector they reach is not established).
- Not documented: whether the privacy switch gates mic power or the PDM clock in hardware.

### Speakers

- "Stereo speaker system (2x 18 mm 10 W tweeters, 1x 75 mm 30 W woofer)" (Google Store). The mydigit.cn thread also mentions a 30 W woofer.
- Speaker impedance, true continuous power, and connector pinout are not documented. Measure DC resistance on a real unit; the only figures are Google's driver sizes and nominal wattages.
- Mainboard speaker connector: large black right-angle wire-to-board header at top-centre, 8 positions (medium-high confidence), about 1.25–1.5 mm pitch, with 6 filter-part pairs beneath it (Seen, FCC p6). Whether leads are discrete wire pairs was listed as undocumented in the text notes; the photo shows a wire-to-board header.

### Amplifiers

| Address | Part | Drives | I2S slot |
|---|---|---|---|
| 0x6C | TI TAS5720 | Left tweeter | Left slot |
| 0x6D | TI TAS5720 | Right tweeter | Right slot |
| 0x6F | TI TAS5720 | Woofer | Right slot |

Source: Fuchsia `sherlock-i2c.cc`, `sherlock-audio.cc`; the kernel device tree agrees on parts, addresses and roles. All three are on the always-on I2C bus (shared with the TCS3400).

- Audio bus: TDM_C in I2S mode, 16-bit samples in 32-bit slots. Stock pins: D0 = GPIOZ_2, D1 = GPIOZ_3, FS = GPIOZ_6, SCLK = GPIOZ_7, MCLK = GPIOAO_9. Both data lanes enabled with 2 channels each (`lanes_enable_mask[0] = 3`, `[1] = 3`).
- Channel split (source comments): 4-channel stream; channel 0 unused; channel 1 = woofer (mono mix of L+R); channel 2 = left tweeter; channel 3 = right tweeter. Inferred: lane 0 carries (unused, woofer) and lane 1 carries (left tweeter, right tweeter).
- Control lines: GPIO_SOC_AUDIO_EN = GPIOH_7 (driven high to enable; the kernel device tree describes the same pin as amplifier shutdown), GPIO_AUDIO_SOC_FAULT_L = GPIOZ_8 (active-low fault).
- A thermistor channel named "therm-audio" exists (100 k NTC, 47 k pull-up).
- Location: three identical QFNs near the speaker header on the connector side of the mainboard (Seen; identity as the amplifiers is Inferred). They leave with the mainboard.
- The 24 V input rail exists mainly for the amplifiers (Inferred; TAS5720 runs from up to 26.4 V PVDD). A new board can reuse TAS5720 parts directly on 24 V.

### Crossover

- Source comment: "Ranges could be wider, but only using them crossed-over at 1'200 Hz in this product". The crossover is digital at about 1.2 kHz.
- Tweeter amplifiers run 6.4 dB below the woofer amplifier (`delta_gains = 0, -6.4, -6.4`).
- Inferred: the stock crossover is entirely digital, so there is probably no passive crossover in the enclosure. Verify by inspection.
- Tweeters passed full-range signal without a high-pass will be damaged. A replacement must provide three amplifier channels and a crossover in DSP or software, with the woofer fed an L+R mono mix.
- Stock EQ and limiter tuning is not in the board driver. Speaker protection must be re-derived.

## Sensors and other devices

### I2C bus map (stock board)

| Stock bus (SDA/SCL pins) | Address | Device |
|---|---|---|
| I2C "AO" (GPIOAO_2/3) | 0x6C | TAS5720, tweeter left |
| I2C "AO" | 0x6D | TAS5720, tweeter right |
| I2C "AO" | 0x6F | TAS5720, woofer |
| I2C "AO" | 0x39 | TCS3400 light sensor |
| I2C2 (GPIOZ_14/15) | 0x38 | FT5726 touch |
| I2C3 (GPIOA_14/15) | 0x2C | LP8556 backlight |
| I2C3 | 0x36 | IMX227 camera |
| I2C3 | 0x3E | "lcd_bias" |

Source: Fuchsia `sherlock-i2c.cc`. Camera, backlight driver and LCD bias share one bus on the stock board; nothing forces that on a new design. Prototype-revision devices seen only in the kernel device tree are listed below.

### Ambient light sensor

- ams TCS3400 RGBC ambient light/colour sensor, node "tcs3400-light-39", I2C 0x39 on the always-on bus (Fuchsia `sherlock-light.cc`).
- Interrupt: GPIO_LIGHT_INTERRUPT = GPIOAO_5, input, no internal pull.
- Kernel device tree: declared as `tcs3472`; the kernel config sets `CONFIG_USE_TCS3400`.
- Location: Inferred to sit behind the front bezel (Google's "Ambient EQ" sensor), probably on the camera/mic/sensor flex and not the mainboard, hence reusable. Unconfirmed.

### Camera (Sony IMX227)

| Item | Value | Source |
|---|---|---|
| Sensor | Sony IMX227; node "imx227-sensor"; sensor ID constant `kSensorId = 0x0227` | Fuchsia `sherlock-camera.cc`, imx227 `constants.h` |
| Interface | 2-lane MIPI CSI-2, 1000 Mbps per lane, RAW10, linear | Fuchsia `imx227_modes.h` |
| I2C | 0x36 on stock bus I2C3 | Fuchsia; kernel device tree |
| Master clock | 24 MHz (CLK_CAM_INCK_24M) output on GPIOAO_10, 4 mA drive | Fuchsia `sherlock-camera.cc`; kernel device tree |
| Reset | GPIO_CAM_RESET = GPIOZ_0 | Fuchsia, kernel device tree |
| Analog supply enable | GPIO_VANA_ENABLE = GPIOA_6 | Fuchsia, kernel device tree |
| Digital supply enable | GPIO_VDIG_ENABLE = GPIOZ_12 | Fuchsia, kernel device tree |
| Modes | 2200x2720 at 30 fps; 1920x1080 at 30 fps; a "fullsensor" mode (about 28 fps); test patterns. All `lanes = 2`, `mbps = 1000` | Fuchsia `imx227_modes.h` |
| Register tables | `imx227_seq.h`; OTP handling in `imx227_otp.cc` | Fuchsia imx227 driver directory |
| Stock pipeline | CSI PHY0 → Amlogic MIPI adapter → ARM ISP, plus GDC (dewarp) and GE2D | Fuchsia `sherlock-camera.cc` |
| Marketing | 6.5 MP, 127 degree field of view, "privacy circuit and green/red LED indicator" | GadgetGuy, 2019-09-09 |
| Module | Rigid black board about 23 mm wide, one ZIF connector, about 25 contacts (±2) at about 0.3 mm effective pitch, on a separate flex | Seen, FCC p9 |
| Google part number | Webcam `G653-01357-02` | joesge.com listing |

Power-up order in the Fuchsia driver: VANA enable high, 50 ms; VDIG enable high, 50 ms; enable 24 MHz clock, 10 ms; drive cam_rst low, 50 ms. Power-down reverses (cam_rst high, clock off, VDIG low, VANA low). Initial state is cam_rst high, enables low.

Conflict: WikiDevi lists the camera as "Sony IMX277 - 12Mp ?" with a question mark. The Fuchsia driver name and sensor ID say IMX227 and are the reliable source.

Inferences:

- The stock reset GPIO is driven low to run the sensor and high to hold it in reset, so there is probably an inverter or level shifter on the stock board (IMX sensors' XCLR is active-low). Confirm polarity at the module connector.
- The VANA/VDIG enable GPIOs imply the sensor regulators (analog about 2.7–2.8 V, digital about 1.0–1.2 V, plus 1.8 V I/O) are on the mainboard and must be re-created. Exact voltages come from the IMX227 datasheet, not from these sources.
- 2200x2720 is about 6.0 MP, consistent with the "6.5 MP" sensor being cropped.
- A raw Bayer sensor needs an image signal processor on the new SoC. The wide field of view means a dewarp step (stock uses GDC) is needed for a normal-looking image.

Gaps:

- Not public: camera module connector type, exact pitch and pin count, pinout, module maker, lens details, whether the module has its own EEPROM beyond sensor OTP.
- IMX227 datasheet not located.
- No mainline Linux IMX227 driver exists. The Fuchsia `imx227_seq.h` tables are the only public register sequences found (see Driver coverage).
- Whether the privacy slide switch physically cuts sensor power, MCLK or PDM clock (the "privacy circuit") is not documented. Software sees one GPIO.

The camera is the highest-risk peripheral in the family. Treating it as a later phase or leaving it dark is a legitimate scoping choice.

### Thermistors

- Three NTC thermistors on the SoC SAR ADC: therm-base, therm-audio, therm-ambient, exposed as "adc-1", "adc-2", "adc-3" (Fuchsia `sherlock-thermistor.cc`, `sherlock-adc.cc`).
- Part: Fuchsia gives the family "ncpXXwf104" (100 k at 25 °C; full R-T table in source) with 47 k pull-ups. The kernel device tree gives Murata NCP03WF104 with a 47 kΩ pull-up to 1.8 V, three on the Max.
- Which ADC channel number maps to which thermistor: the names suggest channels 1/2/3 but the enum definition was not located.
- Physical location of each thermistor is not documented.

### Devices present only in prototype revisions (kernel device tree)

| Device | Bus / address | Notes |
|---|---|---|
| TMP112 temperature sensors | I2C 0x48–0x4B | Four per board, driven as LM75. On the Max they are named for the mic, Wi-Fi, audio and SoC areas. **Removed in the Max `b4` revision.** |
| BMA456 accelerometer | I2C 0x18 | **Disabled in the Max `b4` revision**; may be prototype-only. The tree carries a vendor `bma4xy` driver; mainline support was not checked. |

The Fuchsia-based notes state that no radar, accelerometer or proximity sensor appears in the sherlock board driver and that the I2C list is complete at eight devices. The kernel device tree adds the two prototype-era devices above. Both statements are recorded; for retail (`b4`) hardware they agree.

### Radios and other mainboard devices (lost with the mainboard; reference only)

| Device | Detail | Source |
|---|---|---|
| Wi-Fi/BT | Broadcom BCM43458 (abgn+ac, 1x1, BT 5.0), antenna connector MHF4. Wi-Fi on SDIO; Bluetooth with PCM audio on TDM_A (GPIOX_8..11); BT_REG_ON GPIOX_17; 32.768 kHz LPO on GPIOX_16 | WikiDevi (dated 2020-10-04); Fuchsia `sherlock-audio.cc`, `sherlock-gpios.h` |
| Thread / 802.15.4 | Nordic nRF52840 on SPICC0 (MOSI GPIOC_0, MISO GPIOC_1, SS GPIOC_2, SCLK GPIOC_3, about 1.3 MHz), reset GPIOA_2, interrupt GPIOA_4, bootloader GPIOA_5. Seen on the mainboard with its own coax jack. | Fuchsia `sherlock-spi.cc`, `sherlock-ot-radio.cc`; FCC p6 |
| USB | USB PHY configured with one peripheral-mode, OTG-capable port. Whether it is exposed as a hidden service connector on the Max was not confirmed. | Fuchsia `sherlock-usb.cc` |
| Board ID | Five board-ID strap GPIOs (HW_ID0..4) encode board build and option | Fuchsia `sherlock-gpios.h` |
| Flash | Samsung KLM4G1FETE-B041 (4 GiB eMMC) per WikiDevi; "Samsung 4 GB eMMC" per mydigit.cn; Samsung eMMC seen in FCC p4/5 | |
| RAM | WikiDevi lists "1 GiB" with part Micron MT53E512M32D2NP-053 WT:E. That line is internally inconsistent: a 512M x 32 LPDDR4 part is 2 GB, not 1 GiB. mydigit.cn says Micron 2 GB RAM. Treat memory figures as unverified. | |
| FCC grants | 2.4 GHz BT/BLE, 2.4/5 GHz WLAN (20/40/80 MHz), 802.15.4 (2405–2475 MHz) | fccid.io A4RH2A |

Inferred: MHF4 antenna connectors imply the Wi-Fi/BT (and probably Thread) antennas are separate parts in the enclosure on coax pigtails and can be reused with a new radio module.

## Buttons, LEDs, thermal

### Buttons and switches

| Input | Stock pin | Polarity | Notes |
|---|---|---|---|
| Volume up | GPIO_VOLUME_UP = GPIOZ_4 | Active low (inverted) | Internal pull-up also enabled |
| Volume down | GPIO_VOLUME_DOWN = GPIOZ_5 | Active low (inverted) | Internal pull-up also enabled |
| Volume both | GPIO_VOLUME_BOTH = GPIOZ_13 | Active low (inverted) | Used as FDR (factory data reset) |
| Mic + camera privacy switch | GPIO_MIC_PRIVACY = GPIOH_3 | Active high (not inverted) | "MicAndCamMute" |

Source: Fuchsia `sherlock-gpios.h`, `sherlock-buttons.cc`. Source comment: "No need for internal pull, external pull-ups used". The kernel device tree lists volume up and down and a separate mic mute switch (key code 248).

- Inferred: "volume-both" being a separate GPIO implies a small hardware AND/diode network that asserts a third line when both volume buttons are held. It may be on the mainboard or the button flex.
- The privacy switch gives the SoC a single logic level; one switch covers both mic and camera. Whether it also cuts power or clocks in hardware is not documented.
- Not documented: whether buttons and switch are on the mainboard edge (lost) or on a separate flex/daughterboard; switch part, connector, pinout.

### LEDs

| LED | Stock pin | Control |
|---|---|---|
| AMBER_LED | GPIOAO_11 | PWM_AO_A brightness |
| GREEN_LED | GPIOH_5 | PWM |

Source: Fuchsia `sherlock-light.cc`; the kernel device tree agrees (amber and green on the Max, PWM). Both are brightness-controllable. The GadgetGuy review describes a green/red camera indicator; the source code names amber and green. Physical location of the LEDs is not documented.

### Thermal and mechanical

- Thermal sensing: three NTC thermistors (base, audio, ambient); see Sensors.
- Not documented in any text source: disassembly procedure, screw locations, adhesive, mounting hole positions, heatsink and shield arrangement, height limits, heat-spreader contact areas.
- Seen in FCC photos: shield frame over SoC + memory; about 8 gold-ringed mounting holes; many gold EMI pads.
- Whether a hotter compute module can live in the enclosure is an open engineering judgement, not a sourced figure, and needs validation with the chosen module.

## Power

| Item | Value | Source |
|---|---|---|
| Adapter | "30 W power adapter", "DC power jack" | Google Nest device specifications |
| Rating | "24V/1.25 (30W) pin plug adapter 1.5m cable (no battery)" | GadgetGuy, 2019-09-09 |
| Conflicting figure | A Reddit thread (snippet only, not opened) claims "Nest Max: 24 Vdc, 2A ... Most Google Nest products accept same barrel connector and work with 14 Vdc. Only 10" MAX Hub needs 24 Vdc". The 2 A figure conflicts with the 30 W / 1.25 A rating and is unverified. | r/googlehome |
| DC jack | On a separate DC-jack board (FCC p8) | FCC photos |

- Budget a new board for 24 V in at up to about 1.25 A with the stock adapter; step down to 5 V / 3.3 V / 1.8 V for logic and feed 24 V to the amplifiers.
- The barrel plug reportedly matches other Google Nest speakers physically while the voltage differs (14 V versus 24 V). Do not interchange adapters.
- Not found: barrel outer/inner diameter, centre polarity, adapter model number. The widely quoted 3.5 x 1.35 mm plug size for Google 14 V plugs rests only on marketplace listings and is unsourced.
- Not documented: stock input protection and PMIC parts; the cable/connector between the DC-jack board and the mainboard.
- Logic levels are not documented. On these Amlogic designs each bus can be 1.8 V or 3.3 V per bank. The thermistor pull-ups go to 1.8 V (kernel device tree).

## Driver coverage

Android runs on the Linux kernel, so "driver" here means a Linux kernel driver, plus an Android HAL where one is needed. Mainline status was read from `torvalds/linux` master or confirmed absent by code search. "Vendor" means GPL source exists outside mainline (Google's GPL kernel release, Linux 4.9, as mirrored at github.com/frederic/elaine-linux; a third-party mirror, and the same code should be obtainable from Google's open-source release page) and needs porting to a modern kernel. No drivers have been built or tested.

| Part | Status | Driver / source | Work needed |
|---|---|---|---|
| JD9364 / JD9365 panel driver IC | Mainline | `drivers/gpu/drm/panel/panel-jadard-jd9365da-h3.c` | Add a panel entry with the Max timings and init sequence from the Fuchsia tables. The driver already carries a K&D panel (`kingdisplay,kd101ne3-40ti`) and Radxa displays as models to copy. No mainline entry for these exact panel models exists. |
| FocalTech FT5726 touch | Mainline | `edt-ft5x06.c`, compatible `evervision,ev-ft5726`. Vendor `focaltech_touch` (FT5726 config) is in the GPL tree. | Device-tree node. Mainline does not upload firmware, so it relies on the controller keeping its flashed firmware. |
| ams TCS3400 light sensor | Vendor | Mainline `tcs3472.c` does not support it. Google's GPL tree patches `drivers/iio/light/tcs3472.c` with `CONFIG_USE_TCS3400`. | Forward-port Google's patch to the mainline TCS3472 driver. Zephyr also has a TCS3400 driver, useful as a register reference. |
| Sony IMX227 camera | None | Not in mainline. Old kernels (4.13–4.14) had it inside the Intel-only atomisp staging driver, which is not reusable as-is. Fuchsia's driver has the register tables. | Write a V4L2 sensor driver from the Fuchsia tables, plus ISP tuning on the chosen SoC. Highest risk overall. |
| PDM microphones | Platform | Generic `dmic` codec is mainline; the PDM receiver is the compute module's | Depends on module choice. |
| Buttons, privacy switch | Mainline | `gpio-keys` | Device-tree node. |
| Status LEDs | Mainline | `pwm-leds` | Device-tree node. |
| NTC thermistors | Mainline | `ntc_thermistor` via an ADC | Needs an ADC on the new board. |
| TI LP8556 backlight (stock mainboard part) | Mainline | `drivers/video/backlight/lp855x_bl.c` | Reuse the part and copy Google's register values. |
| TI TPS65132 bias supply (stock mainboard part; **inferred identity**) | Mainline | `drivers/regulator/tps65132-regulator.c` | Applies only if a TPS65132 is fitted on the new board; the stock chip is not identified. |
| TI TAS5720 amplifier (stock mainboard part) | Mainline | `sound/soc/codecs/tas5720.c` | Three instances; crossover and gain offsets are ours to implement. |
| TMP112 (prototype revisions only) | – | Driven in the stock tree by the standard `lm75` driver | Not needed for `b4` hardware. |
| BMA456 (prototype revisions only) | Not checked | Vendor `bma4xy` in the GPL tree; mainline support not checked | Not needed for `b4` hardware. |

Reusing the LP8556, TPS65132 and TAS5720 part numbers on the new board keeps every one of them on a mainline driver, with Google's register values already published. Candidate substitute amplifiers checked: Maxim MAX98357A is mainline (`sound/soc/codecs/max98357a.c`); TI TAS5825M is not in mainline (a `tas5825m.c` is in Google's GPL tree).

The earlier Fuchsia-based notes recorded mainline support for the JD9365 panel driver, `lp855x_bl`, `edt-ft5x06` with FT5726, and TCS3400 as unverified or from memory. The driver coverage check resolved these as shown above. No mainline Linux device tree for sherlock was found.

Android layer above the kernel (not verified; from general knowledge of AOSP):

- Display, touch, buttons: no HAL work. DRM and evdev are picked up directly.
- Backlight: needs the lights HAL pointed at the backlight sysfs node.
- Light sensor: needs a sensors HAL that reads IIO devices.
- Audio: needs an audio HAL configuration (tinyalsa/mixer paths) for the amplifiers and PDM capture.
- Camera: needs a camera HAL on top of V4L2, and depends on the SoC's ISP support.

## What a replacement board must provide

Minimum host interface (Inferred from the Fuchsia bus map):

| Function | Requirement |
|---|---|
| Display | 4-lane MIPI DSI + clock at about 424 Mbps per lane; LCD reset GPIO; two panel-ID strap inputs; 90-degree rotation in software |
| Panel supplies | VDDIO (listed 3.3 V, unverified); AVDD +5.5 V and AVEE −5.5 V from a dual ± bias supply (TPS65132-class, I2C 0x3E on stock); sequencing to be measured |
| Backlight | LED boost driver for 6 strings of 6 LEDs (LP8556 with Google's register values is the low-risk path if the LED strings are exposed at the display connector); backlight-enable GPIO; I2C |
| Touch | I2C (0x38) + interrupt + reset; supply voltage to be measured |
| Camera (if attempted) | 2-lane MIPI CSI-2 at 1000 Mbps per lane; 24 MHz master clock; I2C (0x36); reset; analog and digital sensor regulators with two enables (plus I/O rail); an ISP and dewarp on the SoC |
| Audio out | Three mono class-D amplifier channels (stock: three TAS5720 on 24 V); 2 I2S data outputs (or one TDM) + MCLK; amplifier enable and fault GPIOs; digital crossover at about 1.2 kHz with tweeters 6.4 dB below the woofer and the woofer fed L+R mono |
| Microphones | 1 PDM clock + 1 PDM data input (two channels) |
| Light sensor | I2C (0x39) + interrupt |
| Buttons | 4 GPIO inputs: volume up, volume down, volume-both, privacy switch |
| LEDs | 2 PWM outputs (amber, green) |
| Thermistors | 3 ADC inputs if retained (100 k NTC, 47 k pull-up to 1.8 V on stock) |
| I2C | 2–3 buses |
| GPIO count | About 12: LCD reset, backlight enable, 2 panel ID, touch INT/RST, camera reset + 2 regulator enables, amp enable/fault, light sensor INT, 4 buttons, 2 LED PWM |
| Power | 24 V / 1.25 A (30 W) barrel input via the stock DC-jack board; step-down to logic rails; 24 V to the amplifiers |
| Radios | Wi-Fi/BT (and Thread, if wanted) must be re-provided; stock antennas on MHF4 coax are probably reusable |
| Mechanical | Fit the `G651-01768-05` outline (about 117 x 62 mm), mounting holes and connector positions, all still to be measured |

Design decisions specific to this model:

- **Own board.** The Max differs from the 7-inch models in voltage, panel size, amplifier count and camera, and is undocumented mechanically, so it warrants its own board. The electrical architecture (4-lane DSI portrait panel, I2C touch, PDM mics, TCS3400, LP8556) is shared with the family.
- **Amplifier strategy.** Like-for-like (three TAS5720) or a simpler common part; how to protect small drivers without Google's tuning; where the 1.2 kHz crossover runs (DSP or software).
- **Backlight.** Reuse the LP8556 with Google's published register values, or design a different driver once the strings are characterised.
- **Radio and antennas.** On-module Wi-Fi with its own antenna, or a module wired to the enclosure's stock antennas; whether to restore Thread, which the stock board implements with a Nordic nRF52840.
- **Camera scope.** Whether the camera is attempted at all; deferring it is a legitimate choice.
- **Microphone path.** Stock PDM microphones straight into the SoC, through a voice front end, or replaced. Covered in the shared compute-module document.
- **Thermal headroom** with the chosen module is unvalidated.
- **Staging.** Prove panel, touch and audio on an off-the-shelf module IO board through small flex adapter boards before committing to a full form-fit carrier. Panel bring-up failures present as a silent black screen, so this turns the largest single risk into a cheap experiment.

What does not transfer with reuse: amplifier tuning, far-field voice processing and the camera pipeline. Each must be rebuilt, substituted or dropped.

## Open questions and measurements to take

Getting the display pinout:

1. Ask a distributor for the TV101WXM-AG0 spec; twscreen's page metadata references a "TV101WXM-AG0.pdf" available on inquiry.
2. On a physical unit: read the panel label and flex markings, and count contacts at both ends of the display flex.
3. Probe a stock mainboard with the display unplugged before powering the panel from a new board: grounds by continuity, LED anode and cathodes back to the LP8556 pins, bias rails back to the bias IC outputs, I2C and reset lines back to known test points (FT5726 I2C), DSI pairs by their matched routing to the T931 DSI test points.
4. With a unit still running stock firmware, measure rail voltages and backlight string voltage at the connector before disassembly goes further. This is passive measurement only. Capturing I2C traffic and rail voltages this way also yields touch, backlight and amplifier behaviour.

Open questions:

| Area | Question |
|---|---|
| Display connector | Part, pitch, contact count (27–30 at 0.5 mm or 51–55 at 0.3 mm), contact side, pin-1 orientation, full pinout |
| Display flex | Whether display and touch share one flex or touch has its own tail; what the second small orange flex patch on the display rear is |
| Panel variant | Which panel and DDIC is fitted (read label and ID straps); how kernel entry `G101B158` relates to the Innolux P101DEZ-DD0; any data at all for the Innolux panel |
| Panel supplies | VDDIO voltage (1.8 V or 3.3 V); AVDD/AVEE actual values; bias supply sequencing relative to reset and DSI start; the contents of the Fuchsia `kLcdPowerOnSequenceForAstroSherlockNelson` table (a desk task) |
| Bias IC | Chip marking of the 0x3E device; confirm it is on the mainboard |
| Backlight | Confirm the LP8556 is on the mainboard; LED string forward voltage and current; boost output on a working unit; whether 0xA3 is written on the Max (Fuchsia says 0x32, kernel table has no entry) |
| Touch | Controller location; supply and I/O voltage; pinout; whether the retained firmware works without upload (read chip and firmware ID registers) |
| Camera | Connector part, pin count, pinout; reset polarity at the module; rail voltages; module EEPROM; IMX227 datasheet; which mainboard connector it uses |
| Privacy switch | Whether it cuts camera power, MCLK, mic power or PDM clock in hardware, or only signals the SoC |
| Microphones | Part numbers, supply voltage, PDM clock frequency, connector and flex, which mainboard connector |
| Speakers | DC resistance and markings; impedance and safe power; pin assignment of the 8-position header; confirm no passive crossover |
| Light sensor, LEDs, buttons, thermistors | Physical location of each; which board and connector; thermistor-to-ADC-channel mapping |
| Mainboard connectors | Destination and pinout of the top-right, left-edge and four bottom-edge connectors |
| Power | Adapter label and model; barrel dimensions and polarity; DC-jack board cable; input protection |
| Logic levels | 1.8 V or 3.3 V on each bus |
| Mechanical | Mainboard outline, mounting holes, connector positions, height limits, heat-spreader contact areas, positions of any board-mounted switches, antenna arrangement; where the mainboard sits in the enclosure; disassembly procedure |
| Memory | Actual RAM and eMMC capacity (reference only) |
| USB | Whether the OTG-capable port is exposed as a service connector |

Desk tasks not yet done:

- Download and inspect the FCC internal-photo PDF at full resolution (pages 3 and 7–10 were not reviewed in one viewing; no page was read at a resolution that resolves pin counts or markings).
- Transcribe the Fuchsia panel power-on sequence table.
- Compare other `newman` board revisions; read the Max backlight node for the second panel vendor (only skimmed); read the vendor driver sources (only listed).
- Check the LP8556 register meanings against the TI datasheet.
- Read the WikiDevi page in full, watch the YouTube screen-replacement video, and view the login-gated mydigit.cn photos.
- Archive the Fuchsia sherlock, amlogic-display and imx227 files at a pinned commit.

## Sources

Fuchsia source (`refs/heads/main`):

- Sherlock board driver directory: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/
- sherlock-i2c.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-i2c.cc
- sherlock-gpios.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-gpios.h
- sherlock-audio.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-audio.cc
- sherlock-camera.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-camera.cc
- sherlock-light.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-light.cc
- sherlock-thermistor.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-thermistor.cc
- sherlock-adc.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-adc.cc
- sherlock-buttons.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-buttons.cc
- sherlock-spi.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-spi.cc
- sherlock-ot-radio.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-ot-radio.cc
- sherlock-usb.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/sherlock-usb.cc
- post-init/post-init.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/post-init.cc
- post-init/display.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/display.cc
- post-init/backlight.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/backlight.cc
- post-init/touch.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/devices/board/drivers/sherlock/post-init/touch.cc
- display-panel.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/lib/device-protocol-display/include/lib/device-protocol/display-panel.h
- amlogic-display driver: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/
- panel-config.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel-config.cc
- boe-tv101wxm-fitipower-jd9364.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv101wxm-fitipower-jd9364.h
- boe-tv101wxm-fitipower-jd9365.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/boe-tv101wxm-fitipower-jd9365.h
- innolux-p101dez-fitipower-jd9364.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/graphics/display/drivers/amlogic-display/panel/innolux-p101dez-fitipower-jd9364.h
- ti-lp8556 driver: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/ui/backlight/drivers/ti-lp8556/ti-lp8556.cc
- imx227 driver directory: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/
- imx227.cc: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/imx227.cc
- imx227_modes.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/imx227_modes.h
- imx227 constants.h: https://fuchsia.googlesource.com/fuchsia/+/refs/heads/main/src/camera/drivers/sensors/imx227/constants.h

Kernel:

- Google GPL kernel mirror (Linux 4.9.244, last pushed 2022-06; `newman-b4.dts`, `newman-common.dtsi`, `newman-panel.dtsi`, `newman-bx_defconfig`): https://github.com/frederic/elaine-linux
- Mainline Linux (`torvalds/linux` master) for driver status.

FCC and teardown material:

- FCC ID A4RH2A: https://fccid.io/A4RH2A
- FCC internal photos (10 pages): https://fccid.io/A4RH2A/Internal-Photos/Internal-Photos-rev-4255949
- FCC internal photos PDF: https://fccid.io/m/8c59f709b216b01ac8f2f0d85a67fc2926d86477df2f34c568bc2c981f562af8.pdf
- mydigit.cn teardown thread (2023-10-01, login-gated photos): https://www.mydigit.cn/thread-416860-1-1.html
- TechInsights DDT-1909-802 (paywalled): https://www.techinsights.com/products/ddt-1909-802
- YouTube, David Mudge, failed screen replacement (not watched): https://www.youtube.com/watch?v=MynDJedChLs
- Reddit r/fixit, display ribbon cable (snippet only): https://www.reddit.com/r/fixit/comments/qmz1kw/trying_to_locate_google_nest_hub_max_display/
- Reddit r/googlehome, how to take apart (snippet only): https://www.reddit.com/r/googlehome/comments/eojpgl/how_to_take_apart_the_nest_hub_max/
- Reddit r/googlehome, has anyone taken apart (snippet only): https://www.reddit.com/r/googlehome/comments/qos819/has_anyone_taken_apart_the_nest_hub_max_found/
- Reddit r/googlehome, power cable (snippet only): https://www.reddit.com/r/googlehome/comments/ep5ch8/how_on_earth_to_get_a_nest_hub_max_power_cable/
- WikiDevi (page dated 2020-10-04): https://wikidevi.wi-cat.ru/Google_Nest_Hub_Max_(H2A)

Panel and bias IC:

- panelcd TV101WXM-AG0: https://panelcd.com/product/tv101wxm-ag0/
- twscreen TV101WXM-AG0: https://www.twscreen.com/zh-TW/lcdpanel/22135/boe/tv101wxm-ag0/10.1/1280x800-%E5%B9%B3%E6%9D%BF%E9%9B%BB%E8%85%A6-%E6%B6%B2%E6%99%B6%E9%9D%A2%E6%9D%BF
- yongslcd TV101WXM-AG0: https://ba.yongslcd.com/info/boe-tv101wxm-ag0-general-features800-rgb-69076280.html
- Panelook TV101WXM-AG0 (search snippet only; page behind CAPTCHA): https://www.panelook.com/TV101WXM-AG0_BOE_10.1_Assembly_parameter_42124.html
- Facebook marketplace listing (snippet only): https://www.facebook.com/groups/1605288103416160/posts/2198085010803130/
- Sibling TV101WXM-NLB spec (Rev 1.0, 2021-03-30): https://www.panelook.cn/upload/202311/TV101WXM-NLB_Rev1.0_20210330_202311014276.pdf
- Sibling TV101WXM-NL1 (snippet): https://www.twscreen.com/zh-TW/lcdpanel/2824
- Sibling Innolux P101DEZ-3Z3: https://www.richshinetech.com/goods/P101DEZ3Z3-101-inch-WXGA-IPS_13240.html
- TI TPS65132 datasheet: https://www.ti.com/lit/ds/symlink/tps65132.pdf
- SGMicro SGM3804 datasheet: https://www.sg-micro.com/rect/assets/31cc3b29-c39e-4e40-bead-a6e62c3dcab2/SGM3804.pdf
- Kinetic KTD2151 datasheet: https://manuals.plus/m/e710c5c3f34bd92eab290c336d18cd490821a1ca58840f8f5bd3fe01401cfbf3.pdf
- Kinetic KTD2151 product page: https://kinet-ic.com/display-power/lcd-bias-supply-ics/ktd2151/
- Orient-Chip OCP2131 datasheet: https://www.orient-chip.com/Public/Uploads/uploadfile/files/20231016/OCP2131datasheetver.1.920180306.pdf

Product specifications, reviews and listings:

- Google Store tech specs: https://store.google.com/gb/product/google_nest_hub_max_tech_specs?hl=en-GB
- Google Nest device specifications: https://support.google.com/googlehome/answer/7072284?hl=en
- GadgetGuy review, 2019-09-09: https://www.gadgetguy.com.au/google-nest-hub-max-adding-vision-to-voice-commands/
- Wccftech: https://wccftech.com/google-nest-hub-max-has-a-10-display-6-5mp-front-camera-75mm-woofer/
- eBay display assembly: https://www.ebay.com/itm/157228724198
- eBay (not read): https://www.ebay.com/itm/318552288271
- Amazon display assembly (snippet only): https://www.amazon.com/Google-Screen-Replacement-Display-Assembly/dp/B0F26G7CXR
- joesge.com spare parts: https://www.joesge.com/products/google-nest-hub-max-repair-spare-replacement-parts
- AliExpress (snippet only): https://www.aliexpress.com/w/wholesale-google-nest-hub-max-screen.html
- Raspberry Pi forum, DSI panel bring-up: https://forums.raspberrypi.com/viewtopic.php?t=392157

Project source documents consolidated here:

- `reports/Nest Hub Android hardware research.md`
- `reports/Display connector pinout findings.md`
- `reports/Driver coverage matrix.md`
- `reports/Stock kernel device tree findings.md`
- `research_notes/Nest Hub Android hardware research/nest_hub_max_hardware.md`
- `research_notes/Display connector pinouts/pinout_10inch.md`
- `research_notes/Display connector pinouts/connector_photo_evidence.md`
