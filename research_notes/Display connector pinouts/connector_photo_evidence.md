# Nest Hub family: internal connector photo evidence

Compiled 2026-11-17. Nothing was downloaded to disk; images were viewed in a browser and zoomed in-page.
Every claim is tagged **SEEN** (seen in photo, URL given), **TEXT** (stated in text, URL given) or **INFERRED**.
Pin counts from photos carry a confidence level. Dimensions are photo-scaled estimates (about +/-5-10%); verify with calipers before committing a PCB.

Scale references used:
- 2nd gen hackaday photos: nRF52811 QFN48 = 6.0 mm, S905D3G measured 14.1 mm, DDR3 96-FBGA 7.5 x 13.3 mm (all three agree).
- FCC photos: the ruler in frame. The ruler lies in front of the board, so true sizes are equal to or slightly larger than computed.

---

## 1. Nest Hub 2nd gen (GUIK2, FCC ID A4RGUIK2) - best documented

### Key images
| What | URL |
|---|---|
| Connector side of mainboard, all daughterboards attached | https://cdn.hackaday.io/images/6772551679376436399.jpg |
| Close-up of left half of connector side (best J12 view) | https://cdn.hackaday.io/images/5872331679376498675.jpg |
| SoC / heat-sink side, all boards attached | https://cdn.hackaday.io/images/9216291679376325164.jpg |
| Radar board both sides, annotated pinout | https://cdn.hackaday.io/images/6514751679376803135.jpg |
| Radar board + FFC + mainboard corner (LDOs) | https://cdn.hackaday.io/images/1670461679376915403.jpg |
| FCC internal photos (8 pages; p3 = connector side with ruler, p4 = SoC side with ruler, p7 = radar board with ruler, p8 = display back) | https://fccid.io/A4RGUIK2/Internal-Photos/Internal-Photos-20200702-v1-Internal-Photos-5035982 (PDF: https://fccid.io/m/07ad8aaf6f2e26cc51c5cb66f053116c926b5cf32b7ccc54e8e6662bfa81cd47.pdf) |
| Component list (text) | https://hackaday.io/project/190147/components |

### Mainboard
- Silkscreen `G651-04810-06 REV B` (retail unit, hackaday 5872331...) and `G651-04810-04 REV03` (FCC sample, p3). **SEEN**
- Outline: wide rectangle with scalloped cut-outs on top and bottom edges. About 101 x 46 mm (hackaday scale) / 101.5 mm wide by FCC ruler. **SEEN** (9216291..., FCC p3), medium confidence.
- All FPC/FFC connectors and the speaker header are on the side opposite the SoC. The SoC side carries S905D3G, 2 of the 4 DDR3 chips under one shield frame, TAS5805M amplifier, a 2200 uF / 25 V can, inductors L6, L7, L12 (R68), L2 (4R7). **SEEN** (9216291...)
- Connector side carries eMMC (Samsung KLM4G1FETE) or raw NAND (FCC sample shows a different flash part), 2 DDR3 (U20, U21), BCM4345x Wi-Fi, nRF52811 + SKY66408 (Thread), LP8556 backlight driver (U1), and shields SH2/SH3/SH4. **SEEN** (6772551..., 5872331..., FCC p3)
- Mounting: gold-ringed holes. Approx positions on the SoC-side view, mm from top-left of bounding box: (11, 8) with a slot at (11, 11); (68, 6); (95, 8) with second hole (95, 11.5); (95, 27); (14, 41). FCC p3 shows 6 EMI spring-gasket pads plus about 5 screw holes. **SEEN**, low-medium confidence on coordinates.
- Two U.FL-style coax jacks: J3 (top-left of connector side) and one at the opposite bottom corner. **SEEN** (5872331..., FCC p3)

### Connectors on the mainboard
| Ref | Goes to | Style | Pins (confidence) | Pitch | Evidence |
|---|---|---|---|---|---|
| **J12** | Display + touch, via orange jumper FPC | ZIF FPC, white housing, dark flip actuator on the side away from the cable entry, staggered long/short contacts | **33** (medium-high, +/-1; counted across two overlapping zooms: 19 + 16 - 2) | **0.3 mm** (measured 0.28-0.29) | **SEEN** 5872331... ; cross-checked on 6772551... (33 +/-2). Housing about 10.3-10.9 mm long (FCC p3 ruler and hackaday scale). FPC neck about 10 mm wide. |
| **J15** | Soli radar board (BGT60TR13C) | ZIF FFC, white housing / black flip | **10** (high; 10 tails counted at the radar-board end, about 10 at mainboard end, 10 test pads on radar board) | 0.5 mm (measured 0.52) | **SEEN** 6514751..., 6772551..., FCC p3 ("J15" legible) |
| **J6** | Top mic / sensor / mute-switch strip board (FFC printed "AWM 20861 105C 60V VW-1", stiffener marked "MLB") | ZIF FFC | **12** (medium, +/-1) | 0.5 mm | **SEEN** 6772551... |
| **J5** | Small board top-centre (silk `G651-04807-02`), single hole/sensor | ZIF FFC | **6** (medium) | 0.5 mm | **SEEN** 6772551..., 5872331... |
| **J8** | Side board on the edge opposite the display connector (silk `...10994...`), 6-conductor FFC | ZIF FFC | **6** (high; 6 contacts and 6 conductors visible) | 0.5 mm | **SEEN** 6772551... |
| **J4** | DC-jack / power board | ZIF FFC | **16** (medium-high, +/-1) | 0.5 mm | **SEEN** 6772551..., 5872331..., FCC p3 |
| **J7** | Speaker | Wire-to-board, black shrouded header, right-angle | **5 positions, 3 wires** (red, black/grey, green in positions 1/3/5) (high) | about 1.2-1.25 mm (measured 1.17 hackaday, about 1.1 FCC) | **SEEN** 6772551..., FCC p3 |
| J3 + 1 more | Antennas | U.FL-type coax | - | - | **SEEN** |

Which of J5/J8 boards hold the volume buttons, SHTC3 and TCS3400 was not determined from the photos.

### Display chain
- **One flex carries display and touch.** The orange jumper FPC is a separate part with ZIF fingers at both ends, stiffeners printed `MAINBOARD` and `DISPLAY`, marked `G653-01131-02`, `FF-H 94V-0`, `Fw4320`, `E351586`. **SEEN** (6772551...)
- Its DISPLAY end plugs into a same-size ZIF (CN1) on the panel's own orange FPC assembly, printed `TV070W... FPCA`, date `20200923`. That FPCA carries a Goodix touch controller (listed as GT6853 - **TEXT** hackaday components; package marking **SEEN**), a second small connector CN2 (touch-sensor tail, about 8-10 contacts, low confidence), a small QFN with many passives (panel bias supply, **INFERRED**), silk `TP_VDD`, `LEDPWM`, `VDDIN`, and four soldered backlight tail pads marked + and -. **SEEN** (6772551...)
- **Backlight driver is on the mainboard**: U1 marked `L8556E9` with TI logo (LP8556, E09 option), beside a "101"/"100" inductor and a Schottky, a few mm from J12. **SEEN** (5872331...); part number **TEXT** (hackaday components: LP8556SQ-E09). So LED anode/cathode strings run through J12. **INFERRED**
- Five pairs of small pads/vias sit beside the upper half of J12, consistent with a 4-lane + clock MIPI DSI group. **SEEN** (pads), **INFERRED** (function).
- FCC sample display back label: `KD070D82-39TI-A010 / 20200403 BB2 AB`; cover glass `103040000-00294 TG 2012`. The retail unit FPCA is printed `TV070W...`. So at least two panel sources exist. **SEEN** (FCC p8, 6772551...)
- Candidate connector family for J12: 33-position, 0.3 mm, back-flip ZIF (for example Hirose FH35C-33S-0.3SHW class). **INFERRED** - not read from any marking.

### Radar board (silk `G651-05613-03 REV A` retail, `G651-05613-01 REV02` FCC)
- About 12.5 x 21-23 mm. **SEEN** (FCC p7 ruler, hackaday scale)
- Connector: 10-pin 0.5 mm ZIF with "ACES"-style logo on the actuator. **SEEN** (6514751...)
- Parts: BGT60TR13C, LP5907-1.8 (1V8 LDO), NSC5103A 80 MHz oscillator. **SEEN/TEXT** (annotations in 6514751...)
- Test-pad labels transcribed from the annotated photo, top to bottom: `3V3` (LDO in), `1V8` (LDO out), `GND`, `3V4`, `CS`, `RST`, `MISO`, `MOSI`, `IRQ`, `CLK`. SPI bus. **SEEN** (annotation text in 6514751...)
- **The photo labels test pads, not connector pin numbers.** The pad-to-FFC-pin order is not given and could not be traced.
- Mainboard side: TPS7A2501 LDO (U13) makes 3V4, another LDO makes 3V3, near the radar connector corner. **SEEN** (annotation in 1670461...)
- Protocol detail (SPI, 80 MHz clock, IRQ-driven burst reads, register dump): **TEXT** https://hackaday.io/project/190147/log/217014-decoding-bgt60tr13c-project-soli

### Where things sit
- Amplifier (TAS5805M): mainboard, SoC side. **SEEN**
- Backlight driver (LP8556): mainboard, connector side. **SEEN**
- Touch controller: on display FPCA, not mainboard. **SEEN**
- Microphones (3x PDM), mute switch: on the strip board bonded to the display glass edge, via J6. **SEEN** (FCC p8, 6772551...); mic count **TEXT**
- Radar: own daughterboard via J15. **SEEN**

---

## 2. Nest Hub / Home Hub 1st gen (H1A, FCC ID A4RH1A)

### Key images
| What | URL |
|---|---|
| FCC internal photos (6 pages; p5 = board with shields + bare back, p6 = shields removed, inch ruler) | https://fccid.io/A4RH1A/Internal-Photos/Internal-Photos-v1-3987263 (PDF: https://fccid.io/m/d0ffc974e815edd84cf0220df452538aadc1f4816f616b415c17a4db32294a93.pdf) |
| Board in chassis, display flex and mic FFC attached | https://www.edn.com/wp-content/uploads/Google_Home_Hub_PCB_under_heat_sink.jpg (897x613) |
| Back of board | https://www.edn.com/wp-content/uploads/Google_Home_Hub_PCB_back_no_Faraday_cages.jpg |
| Display jumper flex on its own | https://www.edn.com/wp-content/uploads/Google_Home_Hub_flex_PCB_cable.jpg (700x308) |
| Display rear | https://www.edn.com/wp-content/uploads/Google_Home_Hub_screen_back.jpg |
| I/O board ribbon | https://guide-images.cdn.ifixit.com/igi/cGjpTa6QnAILqSFp.huge |

### Mainboard
- Blue PCB, silk `G651-01770-05 REV01`. **SEEN** (FCC p5/p6)
- Outline: rectangle about 121 x 46 mm with a narrow tab (about 27 x 15 mm) extending the top-left corner. **SEEN** (FCC p6, inch ruler), medium confidence.
- All connectors are on the SoC side; the back has only test pads, gold ground pads and EMI gaskets. **SEEN** (EDN PCB_back_no_Faraday_cages)
- Seven screws. Hole positions in mm from top-left of bounding box: (54, 3), (91, 4), (118, 8), (30, 43), (65, 43), plus gold-pad holes at (23, 12) and (118, 31). **SEEN** (FCC p6, EDN under_heat_sink), low-medium confidence.
- Two shield cans (Wi-Fi on the left, SoC/DDR/NAND large can) with a heat-sink plate and thermal pad over the large can. **SEEN/TEXT** (EDN)
- Visible parts: S905D2, 2x Nanya DDR3 on front + 2x on back, NAND, BCM4345, 220 uF can, two 6R8/100 inductors (amp output), 4R7 inductor. **SEEN/TEXT** (EDN)

### Connectors (no J-designators were legible)
| Position | Goes to | Style | Pins (confidence) | Evidence |
|---|---|---|---|---|
| Right edge, middle; white housing | Display + touch via orange jumper FPC | ZIF FPC, same look and same length as 2nd-gen J12 (about 10.6-11.1 mm housing) | **Not countable** at available resolution. Probably the same 0.3 mm class, about 33 (low confidence, INFERRED from size match only) | **SEEN** FCC p5/p6, EDN under_heat_sink |
| Top-right; black | Mic / ambient-light / mute-switch board, white FFC with blue stiffener | ZIF FFC | about 10-12 (low) | **SEEN** FCC p5, EDN; destination **TEXT** (EDN) |
| Bottom-right; black | I/O board (DC jack + micro-USB) via white FFC | ZIF FFC | about 10-12 (low) | **SEEN** FCC p5; destination **TEXT** (EDN, iFixit) |
| Left edge; black | Speaker, 2-wire | Wire-to-board | 2 (medium) | **SEEN** FCC p6, EDN; **TEXT** (EDN: "two-wire cable") |

- A QFN plus a "100" inductor and four capacitors sit right beside the display connector - the same arrangement as the 2nd gen's LP8556. Backlight driver on mainboard: **INFERRED** (pattern **SEEN** in FCC p5).
- **Display and touch share one flex**: one removable jumper FPC with gold fingers at both ends; the display rear shows a single ZIF on a small green board. **SEEN** (EDN flex_PCB_cable, screen_back)
- Mics (2) and light sensor are on a separate board, not the mainboard. **TEXT** (EDN) / **SEEN**
- Amplifier on mainboard (IC near the two output inductors). **INFERRED** from layout.
- I/O daughterboard has a flip-lock ZIF; iFixit describes flipping the clip up 90 degrees. **TEXT** https://www.ifixit.com/Guide/Google+Home+Hub+1st+Generation+I-O+Daughter+Board+Replacement/149013

---

## 3. Nest Hub Max (H2A, FCC ID A4RH2A)

### Key images
FCC internal photos, 10 pages: https://fccid.io/A4RH2A/Internal-Photos/Internal-Photos-rev-4255949 (PDF: https://fccid.io/m/8c59f709b216b01ac8f2f0d85a67fc2926d86477df2f34c568bc2c981f562af8.pdf)
- p4 bottom: SoC side with ruler. p5 bottom: connector side with shields. **p6 bottom: connector side, shields off, ruler (best).** p7: close-ups. p8: DC-jack board. p9: camera module. p10: display rear.

### Mainboard
- Silk `G651-01768-05`. **SEEN** (p6)
- About 117 x 62 mm by the ruler (likely a little larger in reality), near-rectangular with a small step at the bottom-left. **SEEN** (p6), low-medium confidence.
- About 8 gold-ringed mounting holes, including one near the board centre, plus many gold EMI pads. **SEEN** (p6)
- SoC side: SoC + memory under a shield frame, a row of inductors, two large electrolytic cans. **SEEN** only at thumbnail scale (p4) - not inspected closely.
- Connector side: nRF52840 with coax jack, three identical QFNs near the speaker header (audio amplifiers, **INFERRED**), Wi-Fi chip in a shield outline, two coax jacks. **SEEN** (p6)

### Connectors (no designators legible)
| Position (p6 view) | Likely destination | Style | Pins (confidence) |
|---|---|---|---|
| Right edge, vertical, white housing, about 16-18 mm long | Display (same white-ZIF look, QFN + "100" inductor beside it) | ZIF FPC | About 27 solder tails visible on one side at about 0.5-0.6 mm spacing. Either roughly 27-30 pins at 0.5 mm, or roughly 51-55 pins at 0.3 mm staggered. **Unresolved, low confidence.** Display flex is about 18-19 mm wide on p10. |
| Top-centre, large black header | Speakers (6 filter-part pairs beneath it) | Wire-to-board, right-angle | 8 (medium-high), about 1.25-1.5 mm pitch |
| Top-right, black | Unknown (camera or top boards) | ZIF FPC | about 23-24 (low-medium), about 0.5 mm |
| Left edge, small black | Unknown | ZIF | about 5-6 (low) |
| Bottom edge, 4 connectors left to right | Unknown individually (DC-jack board, mic boards, etc.) | ZIF FFC, black | about 10, about 6, about 16-20, about 8-10 (all low) |

- Camera module (p9): rigid black board about 23 mm wide with one ZIF connector, staggered tails, about 25 contacts (+/-2, low-medium) at about 0.3 mm effective pitch. It connects by a separate flex, not a board-to-board mezzanine. **SEEN**
- Display rear (p10): one wide orange flex leaves the panel; two small green mic boards flank the camera hole at the top, each with its own connector. **SEEN**
- Display and touch on one flex: **INFERRED** (only one large flex visible; a second small orange flex patch near the bottom edge was not identified).
- Backlight driver on mainboard beside display connector: **INFERRED** from the QFN + inductor pattern.

---

## Sources not viewed or not useful
- 1st gen FCC p2-p4 (chassis, mic board in place): seen only as thumbnails.
- EDN photos other than the five listed (PCB_removed, PCB_side1, PCB_MEMS, PCB_antenna, PCB_SoC, power_flex, power_module): not opened. EDN images are small (about 700-900 px wide), so they will not resolve pin counts.
- iFixit: only the 1st-gen I/O board guide was read. Not opened: https://www.ifixit.com/Guide/How+to+Remove+Screen+Google+Nest+Hub/139253 and the device/answers pages. No iFixit guide for the 2nd gen or Max was found in search.
- YouTube, Reddit, Japanese/Chinese teardown blogs, TechInsights, fcc.report mirrors: not searched.
- FCC schematics and block diagrams for all three are confidential ("metadata only").

## Summary: seen versus not seen
- **2nd gen**: every mainboard connector seen at useful resolution, with designators. J12 display connector counted at 33 contacts, 0.3 mm pitch. Radar test-pad labels transcribed, but not the FFC pin order. Not established: J12 pinout, exact connector part numbers, which side board does what.
- **1st gen**: board outline, connector positions, single-jumper display link and all-connectors-on-one-side layout seen. No connector pin count could be read reliably; no designators legible.
- **Max**: board outline and connector positions seen from FCC p6; camera connector seen. Display connector pin count and pitch are ambiguous (two readings given). Destinations of most small connectors not established.
