# Nest Hub Max (H2A) 10.1" LCD FPC pinout research notes

Date: 2026-11-17. Scope: display FPC connector pin assignment, connector type, electrical specs.

## Bottom line

**No pin-by-pin assignment for the Nest Hub Max panel was found.** The exact datasheets
(BOE TV101WXM-AG0 Rev.P0, Innolux P101DEZ-DD0) are not publicly readable. Only summary
parameters for TV101WXM-AG0 were obtained. The one full pinout found (TV101WXM-NLB) is a
SIBLING with a different pin count, driver IC, backlight and power scheme - it must NOT be
used to wire the Nest Hub Max panel.

Labels used: CONFIRMED-FOR-THIS-PANEL / SIBLING / INFERRED.

---

## 1. BOE TV101WXM-AG0 - summary parameters (no pin table)

Label: CONFIRMED-FOR-THIS-PANEL, but only at "distributor parameter summary" level. All three
sources repeat the same panelook-derived data set (panelook says it is based on datasheet
Rev.P0); I could not read the datasheet itself.

| Item | Value | Source |
|---|---|---|
| Interface | MIPI, 4 data lanes | panelcd, twscreen, yongslcd |
| Connector pin count | **51 pins** | panelcd, twscreen, yongslcd |
| Supplies (typ.) | **3.3 / 5.5 / -5.5 V = VDDIO / AVDD / AVEE** | panelcd, twscreen, yongslcd |
| Backlight | WLED **6S6P** (6 LEDs in series x 6 parallel strings = 36 LEDs), no driver on module, 15K h | panelcd, twscreen, yongslcd |
| Touch | module is an "Assembly" with PCAP touchscreen | panelcd, yongslcd |
| Luminance / CR | 400 cd/m2, 1000:1 | all |
| Active area | 135.36 x 216.576 mm | panelcd |
| Outline | 142.32 x 228.2 x 4.3 mm (panelcd); twscreen lists 245.2 x 164 mm (probably incl. cover glass) - sources disagree | panelcd, twscreen |
| MP date / status | Q2 2018, discontinued (twscreen says "In Production") | panelcd |

Sources:
- https://panelcd.com/product/tv101wxm-ag0/
- https://www.twscreen.com/zh-TW/lcdpanel/22135/boe/tv101wxm-ag0/10.1/1280x800-%E5%B9%B3%E6%9D%BF%E9%9B%BB%E8%85%A6-%E6%B6%B2%E6%99%B6%E9%9D%A2%E6%9D%BF
- https://ba.yongslcd.com/info/boe-tv101wxm-ag0-general-features800-rgb-69076280.html
- https://www.panelook.com/TV101WXM-AG0_BOE_10.1_Assembly_parameter_42124.html (search snippet only:
  "Based on datasheet with version of Rev.P0"; page itself is behind a slider CAPTCHA, not read)

Cautions:
- "VDDIO 3.3 V" is what the summaries say. It is unusual (JD936x panels normally take 1.8 V
  IOVCC, and the sibling below uses VDDIO 1.8 V + VCC 3.3 V). Treat the logic rail voltage as
  UNVERIFIED until measured on a working unit.
- Not found for this panel: connector part number, pitch, pin order, LED Vf/current, ID pins,
  reset pin, whether touch I2C/INT/RST are among the 51 pins.

Not-found specifics (AG0): pitch, mating connector P/N, pin table, LED forward voltage and
current. INFERRED only: 6S at ~3 V/LED implies roughly 17-20 V anode voltage; six strings
matches the six current sinks of the TI LP8556 already known to be the stock backlight
driver. Do not size anything from this without measuring.

## 2. Innolux P101DEZ-DD0

Nothing found. Exact-phrase search returned zero results. Closest hit is P101DEZ-3Z3
(SIBLING, and a poor one): 800x1280, MIPI 4 lanes, **FOG (no backlight)**, driver IC
**ER88577** (not JD9364), no pin count / pinout on the page.
- https://www.richshinetech.com/goods/P101DEZ3Z3-101-inch-WXGA-IPS_13240.html

## 3. Sibling with a full pinout: BOE TV101WXM-NLB (SIBLING - NOT for the Nest Hub Max)

Source (full 35-page spec, read online):
https://www.panelook.cn/upload/202311/TV101WXM-NLB_Rev1.0_20210330_202311014276.pdf
(Lenovo "10WX LCD HADS" approval sheet, Rev 1.0, 2021-03-30)

Why it does not transfer to the Nest Hub Max panel:

| | TV101WXM-NLB (sibling) | TV101WXM-AG0 (Nest Hub Max) |
|---|---|---|
| Pins | 39 | 51 |
| Driver IC | Himax HX8252-A02 | Fitipower JD9364/JD9365 |
| Rails on connector | VCC 3.3 V + VDDIO 1.8 V; VSP/VSN made on the panel FPCA (charge pump) | VDDIO + external AVDD +5.5 V / AVEE -5.5 V |
| Backlight | 7S4P, 28 LEDs, 21 mA/string, Vf 2.7/2.9/3.2 V per LED | 6S6P, 36 LEDs |
| Touch | none (bare LCM) | PCAP assembly |
| Luminance | 350 nit | 400 nit |

Mating connector (sibling): Hirose **FH26W-39S-0.3SHW** (39-pos, 0.3 mm pitch), "filled by supplier".

Pin table (SIBLING, every row from the URL above):

| Pin | Symbol | Description |
|---|---|---|
| 1-3 | NC | |
| 4 | FB4 | LED cathode string 4 |
| 5 | FB3 | LED cathode string 3 |
| 6 | FB2 | LED cathode string 2 |
| 7 | FB1 | LED cathode string 1 |
| 8 | NC | |
| 9-11 | VLED | LED anode |
| 12-13 | NC | |
| 14 | LEDPWMOUT | PWM output (CABC) |
| 15 | ID1 | 3.3 V |
| 16 | RESX | reset (in) |
| 17 | ID2 | 0 V / GND |
| 18 | NC | |
| 19-21 | VCC | 3.3 V (3.1-3.5 V) |
| 22-23 | VDDIO | 1.8 V (1.7-1.9 V) |
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

Other sibling data (same URL): IVDDIO 20 mA typ, IVCC 49 mA typ (white); power-on order
VDDIO -> VCC -> RESX -> LP11/init -> video -> VLED (T1 >= 5 ms, T4 >= 10 ms, T5 >= 100 ms);
timing 970 x 1364 total, 79.4 MHz pixel clock, 467 Mbps/lane (differs from the 70.7 MHz
known for the Nest Hub Max).

Value of this sibling: general BOE conventions only (0.3 mm-pitch Hirose FH26-class
connector, GND-separated lane pairs, one cathode pin per LED string, 3 anode pins). INFERRED,
low confidence, that the AG0 follows similar grouping. Other TV101WXM siblings seen in
listings (NL1 = 39 pins, NU5, NL3, NL9, NLA, LL0, NA0, NF0, NU4, TW0) - none found with 51
pins or with a readable datasheet.
- https://www.twscreen.com/zh-TW/lcdpanel/2824 (TV101WXM-NL1, 39 pins, snippet)

## 4. "lcd_bias" at I2C 0x3E - candidate ICs (INFERRED identification)

Verified addresses:
| IC | 7-bit addr | Rails | Source |
|---|---|---|---|
| TI TPS65132 | **0x3E** ("The device 7-bit address is 3E") | VPOS +4.0..+6.0 V, VNEG -4.0..-6.0 V, 0.1 V steps; default +/-5.4 V (most variants) or +/-5.0 V ("0" variants); ENP/ENN enable pins | https://www.ti.com/lit/ds/symlink/tps65132.pdf |
| SGMicro SGM3804 | **0x3E** ("SLAVE ADDR. (3Eh)", search snippet of datasheet) | single-inductor positive/negative dual output | https://www.sg-micro.com/rect/assets/31cc3b29-c39e-4e40-bead-a6e62c3dcab2/SGM3804.pdf |
| Kinetic KTD2151 | **0x3E** ("The 7-bit slave device address is 0111110 binary (or 3Eh)") | VPOS/VNEG, defaults +/-5.0 V on base part, other order codes up to +/-5.6 V; vendor page says programmable +4.0..+5.2 V / -4.0..-5.2 V | https://manuals.plus/m/e710c5c3f34bd92eab290c336d18cd490821a1ca58840f8f5bd3fe01401cfbf3.pdf , https://kinet-ic.com/display-power/lcd-bias-supply-ics/ktd2151/ |
| Orient-Chip OCP2131 | NOT VERIFIED (PDF fetched had no address info) | +4..+6 V / -4..-6 V per snippet | https://www.orient-chip.com/Public/Uploads/uploadfile/files/20231016/OCP2131datasheetver.1.920180306.pdf |

INFERENCE: 0x3E + a panel that needs AVDD +5.5 V / AVEE -5.5 V strongly suggests a
TPS65132-class dual-output bias IC on the mainboard (TPS65132 or a register-compatible
clone such as SGM3804). +/-5.5 V is inside the TPS65132 range but is not its power-on
default, which fits the stock firmware needing to write to it over I2C. The actual part
marking on the H2A board was not identified. The bias IC is on the mainboard, so a
replacement mainboard must supply AVDD/AVEE itself.

## 5. Touch: same FPC or separate tail?

Not established.
- mydigit.cn teardown (text only, images need login) says the *touch ribbon* was torn while
  prying the screen off ("触摸排线搞断了"), i.e. the author distinguishes a touch flex from
  the rest. Suggests (INFERRED, weak) that touch has its own tail. https://www.mydigit.cn/thread-416860-1-1.html
- TV101WXM-AG0 is sold as an LCD+PCAP "Assembly" with a single 51-pin figure; whether the 51
  pins include touch is not stated.
- A Facebook marketplace listing snippet reads "Pantalla LCD + Táctil 10.1'' BOE
  TV101WXM-AG0-P0 Flex Central 51 Pines Compatible con Google Nest HUB Max" - independent
  confirmation of the part number / 51-pin figure for this device, and the only FPC marking
  found ("TV101WXM-AG0-P0"). Page could not be opened; snippet only.
  https://www.facebook.com/groups/1605288103416160/posts/2198085010803130/

## 6. Teardowns / internals found

- FCC ID A4RH2A internal photos (10 pages): https://fccid.io/A4RH2A/Internal-Photos/Internal-Photos-rev-4255949
  Viewed pages 1, 2, 4, 5 and part of 6 at low resolution in a browser (not downloaded).
  Observed: p.1 shows the removed display assembly (black back, a wide orange flex near the
  centre, a small green PCB strip on one edge) and the chassis with the mainboard, an orange
  flex plus several blue-tabbed white FFCs entering the top of the board, and antenna boards.
  p.4/5 show both mainboard sides with a cm ruler (Amlogic T931, Samsung eMMC, board about
  11 cm wide). The connector side carries roughly eight FPC/FFC connectors: one fine-pitch
  near top centre, one long vertical one on the right edge, several small ones along the
  bottom. Resolution is far too low to count pins or read markings, and I could not tell
  which connector is the display. Pages 3 and 7-10 were not reviewed (viewer stopped
  rendering). Schematics and block diagram exhibits are confidential (metadata only).
- mydigit.cn teardown thread, 2023-10-01, 22 numbered photos (login-gated): mainboard, T931,
  Micron 2 GB RAM, Samsung 4 GB eMMC, 30 W woofer. No display connector text. URL above.
- TechInsights Deep Dive Teardown DDT-1909-802 (paywalled, 2019-11-26): https://www.techinsights.com/products/ddt-1909-802
- YouTube "Opening a Google Nest Hub Max to replace screen [FAILED ATTEMPT]" (David Mudge):
  https://www.youtube.com/watch?v=MynDJedChLs - not watched/transcribed.
- Reddit r/fixit "Trying to locate Google Nest Hub Max display ribbon cable" (symptom:
  wiggling the display ribbon restores image; backlight on but no image) - snippet only,
  Reddit not scrapeable: https://www.reddit.com/r/fixit/comments/qmz1kw/trying_to_locate_google_nest_hub_max_display/
- Reddit r/googlehome teardown questions (snippets only):
  https://www.reddit.com/r/googlehome/comments/eojpgl/how_to_take_apart_the_nest_hub_max/
  https://www.reddit.com/r/googlehome/comments/qos819/has_anyone_taken_apart_the_nest_hub_max_found/
- WikiDevi page exists: https://wikidevi.wi-cat.ru/Google_Nest_Hub_Max_(H2A) (not read)

## 7. Replacement-part listings

No listing text gave a panel model, FPC marking, pin count or connector type (other than the
Facebook snippet in section 5).
- https://www.ebay.com/itm/157228724198 (text checked: nothing; notes "bending over 90 degrees
  will result in a black display"; photos could not be opened)
- https://www.ebay.com/itm/318552288271 (not read)
- https://www.amazon.com/Google-Screen-Replacement-Display-Assembly/dp/B0F26G7CXR (snippet only)
- https://www.joesge.com/products/google-nest-hub-max-repair-spare-replacement-parts
  (Google part numbers for other parts only: AUX PCB G651-01948-03, webcam G653-01357-02;
  no LCD part number)
- https://www.aliexpress.com/w/wholesale-google-nest-hub-max-screen.html (snippet only)

## 8. Summary

Confirmed for this panel (summary level, not from the datasheet itself):
- TV101WXM-AG0: 51-pin connector, MIPI 4-lane, external AVDD +5.5 V / AVEE -5.5 V plus VDDIO
  (listed as 3.3 V - verify), 6S6P WLED backlight without on-module driver, PCAP touch
  assembly. Listing marking "TV101WXM-AG0-P0", "51 pines" for the Nest Hub Max.

Sibling only:
- TV101WXM-NLB full 39-pin pinout, Hirose FH26W-39S-0.3SHW, 7S4P backlight, VCC 3.3 V +
  VDDIO 1.8 V. Different pin count, driver IC and power scheme - reference for BOE
  conventions only.
- Innolux P101DEZ-3Z3: FOG, ER88577, no pin data.

Inferred:
- lcd_bias @0x3E is a TPS65132-class +/- bias supply (TPS65132, SGM3804 and KTD2151 all
  verified at 0x3E) generating AVDD/AVEE.
- Touch probably on its own flex tail (single weak textual hint).
- LED anode voltage about 17-20 V; 6 strings map to LP8556's 6 sinks.

Not found:
- Any pin-by-pin assignment for TV101WXM-AG0 or P101DEZ-DD0.
- Connector part number, pitch, pin-1 orientation for the 51-pin FPC.
- LED Vf / string current for the AG0 backlight.
- Any P101DEZ-DD0 data at all.
- Any teardown that documents the display connector; any project reusing the screen.
- Touch FPC pinout.

Searches / fetches that came up empty or blocked:
- "P101DEZ-DD0" exact phrase - 0 results.
- "TV101WXM" "51 pins" MIPI AVDD AVEE - 0 results (twice, two phrasings).
- "TV101WXM-AG0" "51" pin Google Nest Hub Max LCD - 0 results.
- "TV101WXM-AG0-P0" / 规格书 / datasheet / pinout - only distributor summaries.
- Nest Hub Max display reuse / driver board / HDMI / MIPI hack - nothing relevant.
- Nest Hub Max 分解 / 拆解 / zerlegen - only the mydigit.cn thread and Reddit questions.
- DTS-style search for bias regulators at reg 0x3e - 0 results.
- panelook.com and m.panelook.com AG0 pages - CAPTCHA wall.
- Reddit and Facebook pages - scraper unsupported.
- eBay listing photos - browser navigation denied.
- OCP2131 datasheet - address not present in the fetched PDF.
- mydopoint.com AG0 page - no interface detail.

Recommended next step (outside this research): the pinout has to come from the hardware -
either buy the TV101WXM-AG0 datasheet/spec from a distributor (twscreen page metadata
references a "TV101WXM-AG0.pdf" available on inquiry) or probe a stock mainboard's display
connector (continuity to the bias IC outputs, LP8556 LED pins, T931 DSI test points, FT5726
I2C) before powering the panel from a new board.
