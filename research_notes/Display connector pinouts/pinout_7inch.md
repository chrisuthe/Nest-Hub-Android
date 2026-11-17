# Nest Hub 7" LCD FPC pinout research notes (2026-11-17)

## Bottom line

**No pin-by-pin assignment was found for any of the four panels, nor for any sibling.**
Nothing in this file is a usable pinout. Do not design a board from it; the connector
must be probed (continuity + scope on a working unit) or a datasheet obtained from a
distributor.

Labels: CONFIRMED-FOR-THIS-PANEL = distributor spec page for the exact part number
(still a listing summary, not the datasheet itself) / SIBLING / INFERRED.

## 1. What was found per panel

### BOE TV070WSM-TG1 (1st gen) - listing-level specs only
| Item | Value | Label | Source |
|---|---|---|---|
| Interface | MIPI, 4 data lanes | CONFIRMED-FOR-THIS-PANEL | https://www.twscreen.com/lcdpanel/24300 |
| FPC pin count | 39 pins | CONFIRMED-FOR-THIS-PANEL | https://eechain.com/en/product/detail/TV070WSM-TG1 ; twscreen (above) |
| Pitch | 0.3 mm ("39 pins 0.3mm Connector") | CONFIRMED-FOR-THIS-PANEL (search-snippet text of the Panelook page; page itself not readable) | https://m.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_overview_51122.html |
| Supply | 3.3 / 3.3 V typ (VCC / VDDI) | CONFIRMED-FOR-THIS-PANEL | twscreen, Panelook snippet |
| Backlight | WLED 7S3P (21 LEDs), 15k h, no on-module driver | CONFIRMED-FOR-THIS-PANEL | twscreen |
| Touch | "On-Cell" | CONFIRMED-FOR-THIS-PANEL | eechain |
| Outline / AA | 114.28 x 177.37 mm, 3.76 mm thick / 89.244 x 152.267 mm | CONFIRMED-FOR-THIS-PANEL | eechain |
| Luminance / CR | 450 nit / 1000:1 | CONFIRMED-FOR-THIS-PANEL | eechain |
| Mass production | 2020 Q4 | CONFIRMED-FOR-THIS-PANEL | eechain |
| Mating connector P/N | NOT FOUND | - | - |
| Pin assignment | NOT FOUND | - | - |
| LED Vf / If | NOT FOUND | - | - |

Caveats:
- EEChain gives mass production 2020 Q4, two years after the Home Hub (2018). Either the
  distributor date is a re-listing date or TG1 is not the only/original 1st-gen BOE suffix.
  Worth re-checking the suffix on the label of a physical unit.
- VCC/VDDI both 3.3 V with no AVDD/AVEE listed suggests the bias rails are generated on the
  module. INFERRED, not stated by any source.
- twscreen page metadata references a file "TV070WSM-TG1.pdf" but exposes no download link.
- Panelook lists "datasheet 1 files" for TG1; it sits behind login + slider CAPTCHA (not bypassed).

### Innolux P070ACB-DB0 (1st gen) - nothing for the exact part
- Exact-part searches return zero hits. Panelook/eechain/twscreen have no indexed DB0 page.
- SIBLING data, P070ACB-DB1 / -DB2 (same family, 600x1024, NOT confirmed identical to DB0):
  - MIPI 4 lanes, 39-pin connector, 3.3 V typ, WLED 6S3P (18 LEDs) 12k h no driver,
    On-Cell touch with I2C (one listing says I2C/SPI).
  - DB2 outline 192 x 106.6 x 5.64 mm, AA 152.37 x 89.28 mm, 350 nit, 800:1. The outline
    differs from the BOE TG1 (177.37 x 114.28), so DB2 is a different mechanical module
    with cover lens; do not assume the same FPC.
  - Sources: https://mohotec.com/product/7-0-innolux-p070acb-db2/ ;
    https://www.enlcd.com/products/19772 (DB1) ;
    https://www.yinjidz.com/products-detail.php?ProId=1893 (DB2) ;
    https://www.alibaba.com/product-detail/P070ACB-DB2-7-Inch-LCD-Touch_1601222037979.html
  - None gives connector P/N, pitch, or pin table.
- Other suffixes seen: P070ACB-DB6 (FCC test report component list,
  https://fccid.io/m/a914b77207fa202ac79b31af529dd6eb6f20992ac6d662ad1fe66f74fda5286f.pdf),
  listed alongside BOE "TV070WSM-TL1-38P0" as alternate LCDs in one product (report not opened).

### BOE TV070WSM (2nd gen, suffix unknown) - nothing
- Other TV070WSM suffixes that exist: NM0, TG0, TG1, TG2, TH0, TL0, TL1, TL2, TU0.
- SIBLING, electrically DIFFERENT - do not use: TV070WSM-NM0 (1st-gen Amazon Echo Show panel)
  is 40-pin 0.5 mm, mating connector Kyocera 04-6299-640-020-846+, supplies 1.8/3.3/9/19/-7 V.
  Sources: https://m.panelook.cn/TV070WSM-NM0_BOE_7.0_LCM_parameter_cn_32259.html (snippet,
  pin definition shown as "N/A") ;
  https://ronboelectronics.en.made-in-china.com/product/QZzJFhVdbaWm/China-7-0-Inch-Tablet-LCD-Screen-Resolution-1024-600-Boe-TV070wsm-Nm0-Mipi-40-Pins.html
  This shows the TV070WSM family does NOT share one pinout or even one pin count.

### K&D KD070D82 (2nd gen) - nothing
- No datasheet. The only public variant is KD070D82-39TI-A00x, the Lenovo Tab M7 (TB-7305)
  LCD+touch assembly. Lenovo's FCC report lists "TV070WSM-TL2 BOE" and "KD070D82-39TI-A004 K&D"
  as its two LCD sources: https://fccid.io/m/0c706c2d3b04f1d71bb6fdeaca4258ebf14f0436a8d83122ba5047f2533b96a5.pdf
- "39" in "-39TI" probably means 39 pins. INFERRED from the part number only.
- A flex marking seen in a marketplace comment: "AS2538A S8C-KD070D82-39TI-A014-LCD-FPC-V1.0 240613"
  (a different tablet): https://am.ozon.com/product/tachskrin-sensornoe-steklo-dlya-plansheta-atouch-q37-1529015875/
- Whether the Nest Hub 2 uses a -39TI variant is unknown.

## 2. Connector type / cable arrangement (1st gen)

- iFixit guide, step "Remove Screen Ribbon": a single screen ribbon goes into a flip-lock ZIF
  socket; pin 1 is marked with a triangle and "1". No pin count or pitch stated.
  https://www.ifixit.com/Guide/How+to+Remove+Screen+Google+Nest+Hub/139253
  Images (not inspected at full resolution): https://guide-images.cdn.ifixit.com/igi/IQEnESt6vdP1nWcD.large ,
  https://guide-images.cdn.ifixit.com/igi/UIT1hptFjbIFTCsM.large
- EDN teardown photos (my reading of low-res images, INFERRED): the back of the panel carries one
  connector near the edge, and a separate loose jumper flex with gold-finger ends links screen
  half and base half. So the mainboard does not take the panel tail directly; there is an
  intermediate jumper flex. Pin count not resolvable from the photos.
  https://www.edn.com/teardown-google-home-hub-smart-speaker-and-display/
  https://www.edn.com/wp-content/uploads/Google_Home_Hub_screen_back.jpg
  https://www.edn.com/wp-content/uploads/Google_Home_Hub_flex_PCB_cable.jpg
- 2nd gen: a Facebook group post asks for the "orange cable that connects the screen to the main
  board", so it also uses a separate jumper flex (snippet only):
  https://www.facebook.com/groups/GoogleAssistant/posts/1429846610882402/
- Touch on same FPC or separate tail: NOT DOCUMENTED. iFixit describes one ribbon, and 39 pins
  on an on-cell-touch module is consistent with touch I2C/INT/RST sharing the FPC. INFERRED.

## 3. Backlight
- TG1: 7S3P, 21 LEDs (CONFIRMED-FOR-THIS-PANEL, twscreen). P070ACB-DB1/DB2: 6S3P (SIBLING).
- String voltage / current: NOT FOUND. Roughly 7 x ~3 V = ~21 V (6S: ~18-19 V) at about 20 mA
  per string is typical. INFERRED from generic white-LED values; measure on a working unit.
- The two 1st-gen sources may have different series counts (7S vs 6S). A constant-current boost
  like the stock LP8556 handles both; a fixed-voltage design would not.

## 4. Replacement-part listings (no FPC markings or pin counts given in any)
- https://www.aliexpress.com/item/4000400520350.html (1st gen; the one iFixit answers point to; page text not retrievable)
- https://www.aliexpress.com/i/1005004924443271.html (2nd gen)
- https://he.aliexpress.com/item/1005013448231437.html (1st / 2nd gen)
- https://www.amazon.com/Home-Nest-Hub-Replacement-Generation/dp/B0CZ9G4HGD (2nd gen; states 2nd-gen screen does not fit 1st gen)
- https://www.amazon.com/Replacement-Generation-Display-Digitize-Assembly/dp/B0D1GD6QRV (2nd gen)
- iFixit answer thread quotes a panel label string "inxg86000087011088w02334183008530ge"
  ("inx" prefix suggests an Innolux unit): https://www.ifixit.com/Answers/View/600556/
- 1st- and 2nd-gen screens are sold as separate, non-interchangeable parts.

## 5. Forum / reuse posts
- https://www.reddit.com/r/googlehome/comments/11m628w/google_nest_hub_screen_driver/ - someone asking how to
  drive the screen from another board. Could not be read (Reddit blocked for both tools); snippet shows a question only.
- No Hackaday, EEVblog, GitHub or blog post documenting the connector was found.

## 6. Searches that came up empty
- "P070ACB-DB0" (exact, alone and with datasheet/pinout terms, English and Chinese): 0 results
- "TV070WSM-TG1"/"TG0"/"TG2" + 39 pins / on-cell / pin assignment: only distributor summaries
- "TV070WSM-TL0/TL1/TL2" datasheet: 0 results
- TV070WSM datasheet PDF search: only FCC component lists and an Echo Show teardown
- "P070ACB" PDF search: only FCC component lists
- KD070D82 datasheet / "KD070D82-39TI" pinout / Lenovo TB-7305 schematic: retail listings only
- Generic "7 inch 600x1024 JD9365/JD9364 39pin 0.3mm on-cell" datasheet: 0 results
- Nest Hub display "39 pin"/0.3mm reuse, GitHub hardware reverse engineering: nothing relevant
- Panelook parameter/datasheet pages (panelook.com, m.panelook.com, m.panelook.cn): CAPTCHA wall, not bypassed
- beyondinfinite.com: no P070ACB or TV070WSM files indexed

## 7. Suggested next steps
1. Log in to Panelook manually and pull the TG1 datasheet (and check for P070ACB-DB0):
   https://www.panelook.com/TV070WSM-TG1_BOE_7.0_Assembly_parameter_51122.html
2. Request datasheets from twscreen / eechain / enlcd (they offer them on inquiry).
3. Read the silkscreen/label on the actual panel and jumper flex, and count the contacts.
4. Probe a working unit: ground pins by continuity, DSI pairs by scope, LED A/K by the LP8556
   boost output, touch I2C by address scan, rails by DMM.
