# Compute modules and board design

Everything that applies to the project as a whole and not to one model: what the three models share, the open design decisions, chip and module selection for the Linux stack, and building blocks for the new board. Model-specific hardware facts are in `Nest Hub 1st gen.md`, `Nest Hub 2nd gen.md` and `Nest Hub Max.md`.

Scope: a custom replacement mainboard built around an off-the-shelf compute module, reusing the stock display, touch, microphones, speakers, sensors and enclosure. Modifying or bypassing the stock firmware is out of scope.

The OS is Linux running Hearth, an open-source Flutter smart-home kiosk, with Home Assistant's Linux Voice Assistant alongside it for voice. Raspberry Pi silicon is excluded. Android was the earlier plan; the Android material in this document is kept for reference only.

## What the three models share

- **Sources.** Google's open-source Fuchsia board drivers ("astro", "nelson", "sherlock") name every peripheral, bus, I2C address and control line, and publish DSI timings and init tables for every panel. Google's GPL Linux kernel tree ("estelle", "elaine", "newman") confirms and extends this. Neither gives connector pinouts. The Fuchsia files should be archived at a pinned commit, since nothing guarantees they stay on the main branch.
- **One electrical architecture.** All three use a 4-lane MIPI DSI panel, I2C touch, PDM microphones, a TCS3400 light sensor at 0x39 and an LP8556 backlight driver at 0x2C.
- **Portrait-native panels mounted landscape.** The system must rotate the framebuffer 90 degrees. The marketing resolutions (1024x600, 1280x800) describe use, not scan order.
- **Several panel variants per model**, chosen on the stock board by strap pins on the display module. The new board should read the straps, or the builder must identify the panel in hand.
- **The DSI link is undemanding.** By the researchers' arithmetic, the 7-inch panels need about 265 to 300 Mbit/s per lane and the 10-inch about 424 Mbit/s, within the 500 Mbit/s JD936x limit quoted in the Fuchsia comments. Display bandwidth does not discriminate between compute modules.
- **What leaves with the mainboard:** amplifiers, backlight driver, radios and regulators. The new board must recreate them. Reusing the stock part numbers (LP8556, TPS65132, TAS2770, TAS5805M, TAS5720) keeps them on mainline Linux drivers with Google's register values published.
- **What does not transfer:** amplifier tuning, far-field voice processing, the camera pipeline and the radar algorithms. Each must be rebuilt, substituted or dropped.
- **No public pinout exists for any connector on any model.** A sacrificial unit of each target model, ideally still booting stock firmware for passive rail and bus measurements, is the most valuable purchase before design work.

## Open design decisions

1. **Target model, and one board or three.** The 1st gen is the best-documented and simplest: no radar or camera, simple audio, known 14 V supply. The 2nd gen shares its panel family and bus layout (and is now also confirmed at 14 V) and adds the radar, the GT6853 touch risk and a third microphone. The Max differs in voltage, panel size, amplifier count and camera and warrants its own board. A shared schematic core for the two 7-inch models is plausible; a shared outline is unknowable until both boards are measured.
2. **Compute module.** Linux running Hearth, no Raspberry Pi silicon, on a module narrow enough for a 46 mm board. The research shortlist is RK3576 (Boardcon LGA3576) and i.MX 8M Plus (Ka-Ro QSXP), with RK3566/RK3568 as a cheaper fallback limited to H.264 under mainline; see "Linux stack and chip selection". Nothing is chosen until a dev board passes the go/no-go checks and runs Hearth. The kernel version of the chosen module also sets how much porting the vendor touch and light-sensor drivers need.
3. **Microphone path.** Stock PDM microphones straight into the SoC with software echo cancellation, through an XMOS-class voice processor, or replaced by an added array. The first option now depends on the chip: mainline PDM support was found for RK3566/RK3568 but not for RK3588 or RK3576, and is unconfirmed on several others (see the chip comparison).
4. **Amplifier strategy.** Like-for-like stock parts or a simpler common part, and how to protect small drivers without Google's tuning. On the Max this includes where the 1.2 kHz crossover runs.
5. **Backlight.** Reuse the LP8556 with Google's published register values (the low-risk path), or design a different driver once the LED strings are characterised.
6. **Radio and antennas.** On-module Wi-Fi with its own antenna, or a module wired to the stock antennas; and whether to restore Thread, which the 2nd gen and Max implement with Nordic radios on the stock mainboard.
7. **Optional peripherals and thermal headroom.** Whether the radar ships in version one as a presence sensor, whether the Max camera is attempted, and whether a hotter module can live in an enclosure designed for an SoC dissipating a few watts within a 15 W budget. That thermal judgement is engineering opinion, not a sourced figure.

**Recommended process:** prove panel, touch and audio on an off-the-shelf module IO board through small flex adapter boards before committing to a form-fit carrier. Panel bring-up failures show only as a black screen, so this turns the largest risk into a cheap experiment.

## Android layer above the kernel (superseded)

Superseded by decision D8: the product runs Linux, so none of these Android layers are needed. Kept for reference. Not verified; from general knowledge of AOSP.

- **Display, touch, buttons:** no HAL work. DRM and evdev are picked up directly.
- **Backlight:** the lights HAL pointed at the backlight sysfs node.
- **Light and humidity sensors:** a sensors HAL that reads IIO/hwmon devices. The community Raspberry Pi build has an I2C sensor HAL to extend.
- **Audio:** an audio HAL configuration (tinyalsa/mixer paths) for the amplifier and PDM capture.
- **Radar:** no Android framework concept; expose it through a custom service or app.
- **Camera:** a camera HAL on top of V4L2, dependent on the SoC's ISP support.

## Status of earlier risks

- Section 3 below calls obtaining the panel init sequence "the single largest technical risk". That was written before the Fuchsia and GPL kernel tables were found; the init sequences, timings and reset sequence are now published for every panel variant. The remaining display risk is the physical pinout and supply rails.
- The gap in section 3 about which touch controller, light sensor and panel driver IC the devices use is closed; see the model documents.

# Linux stack and chip selection

Research date: 2026-11-17. "Unverified" marks background knowledge not checked against a source. Findings older than 2025 carry their date. "Not found" means the research did not find it, not that it is absent.

## What Hearth needs from a chip

Hearth's Linux stack:

| Layer | Component |
|---|---|
| UI | flutter-pi (Hearth fork): DRM/KMS + GBM + EGL/OpenGL ES, no compositor |
| Video | GStreamer through the flutterpi_gstreamer_video_player plugin, frames imported into GLES as EGL dmabuf images |
| Web dashboards | WPE WebKit through GStreamer `wpevideosrc`; the fork shares one GstGLDisplay and GstGLContext across `wpevideosrc` pipelines |
| Audio | ALSA / PipeWire |
| Voice | Home Assistant Linux Voice Assistant, running alongside |

Go/no-go checks for any candidate, run on the BSP or mainline image that would ship:

| Check | Why |
|---|---|
| `EGL_EXT_image_dma_buf_import` | flutter-pi's video player refuses to upload frames without it |
| `EGL_EXT_image_dma_buf_import_modifiers` | Optional in code, but without it current flutter-pi master has no format list and video asserts or finds no format |
| `EGL_KHR_surfaceless_context` | Required by flutter-pi's GL renderer |
| `GL_OES_EGL_image_external` | Enables external OES textures for imported frames |
| Atomic KMS | flutter-pi uses atomic modesetting (with a legacy fallback); multi-plane commits and explicit fencing need atomic |
| Hardware H.264 decode (ideally HEVC) reachable from GStreamer with dmabuf output | Software decode hits flutter-pi's broken CPU-copy path (#525) |
| 4-lane MIPI DSI | All three panels are 4-lane DSI |
| PDM or I2S microphone input | Stock microphones are PDM |
| WPE `wpevideosrc` running | Highest-risk component; see below |

No non-Pi board has a report of flutter-pi video or WPE `wpevideosrc` working. Whatever chip is chosen, a bench test of the full stack is required before committing to it.

## flutter-pi and other embedders

**Upstream status.** The flutter-pi README says it is "only tested on a Rasberry Pi 4 2GB" and should work on other Linux platforms with KMS/DRI plus Mesa GLES/EGL, libdrm and libgbm; non-Mesa drivers are "untested". There is no board list and no CI on other hardware. The maintainer fixes non-Pi bugs as they are reported.

**UI reports by chip** (flutter-pi issue tracker):

| SoC | GPU driver | Result | Date |
|---|---|---|---|
| RK3566 (Radxa Zero 3E) | Mesa Panfrost | "Works well" (#447) | 2024 |
| RK3566 (Odroid M1S) | Bifrost blob | UI works; a DSI panel skipped frames until its front porch was fixed (#441) | 2024 |
| RK3588 | not stated | Failed on master, worked on a branch since merged (#327) | 2023 |
| RK3568 (Rockchip SDK, kernel 5.10) | vendor | Touch and mouse fail, open (#518) | 2026 |
| Allwinner H618 | Panfrost | Works once built for arm64 (#421) | 2024 |
| Amlogic S905Y4 | Panfrost | Image artefacts, also seen in Weston, open (#521) | 2026 |
| TI AM625 | PowerVR | flutter-gallery runs; hardware cursor fails (#392) | 2024 |
| i.MX 8M Nano | Vivante blob | Rendering artefacts (#271) | 2022 |
| i.MX 8QM | Mesa etnaviv | Atomic commit fails on Vivante tiled modifiers; works with an alignment patch that is not merged (#494) | 2025 |
| Custom Vivante SoM | etnaviv | Raster jank from blocking on page flip (#528); addressed by #529 | 2026 |

Failures fall into recurring classes: vendor tiled modifiers, display drivers with odd plane/zpos/atomic behaviour, implicit-sync bugs, and vendor GL blob crashes. No report was found for RK3576, RK3588 under Panthor, i.MX 8M Plus, i.MX 93/95, AM62P, MediaTek Genio or QCS6490.

**Video and WPE are the weak point.** Video broke on Rockchip libmali blobs (RK3399, 2023; RK3566, Nov 2024; both still open) because those blobs lack `EGL_EXT_image_dma_buf_import_modifiers`. Mesa drivers (Panfrost/Panthor, etnaviv, freedreno) expose it (unverified for Panfrost specifically). The CPU-copy fallback for software-decoded frames is broken on master (#525, 2026). WPE through `wpevideosrc` adds its own GPU requirements (WPEBackend-fdo, EGL, a GL context shared through GstGL) and has not been validated on any non-Pi board. The research infers that a SoC with a mainline Mesa GPU driver and mainline KMS is the strongest choice, and that vendor-blob BSPs (Rockchip libmali, NXP Vivante galcore, TI/IMG PowerVR DDK, Qualcomm Adreno) are where Hearth would carry patches.

**Maintenance.** Last push 2026-11-15; few other commits in 2026. #529 "avoid blocking raster thread on page flip" merged 2026-11-08; Pi CM4 reporters say it removes about 90% of raster jank. The Hearth fork (last push 2026-08-12) should rebase onto it. Open and unmerged as of Nov 2026: #535 (2D GBM buffers for copied video frames), #534 (GStreamer video uploads, event lifetime, looping), #522 (serialise GBM allocation, SIGSEGV with concurrent pipelines), #523 (CEF webview), #490 (frame scheduling). The upstream webview tracking issue (#491, 2025) lists CEF, WPE and Servo; none is merged.

**Alternatives:**

| Embedder | State | Fit for Hearth |
|---|---|---|
| ivi-homescreen / meta-flutter (Toyota Connected) | Very active (Nov 2026). DRM/KMS EGL and Vulkan backends, software backend, zero-copy dma-buf platform views, GStreamer `video_player_linux`, CEF webview marked WIP. meta-flutter builds ivi-homescreen, flutter-auto, flutter-pi and flutter-elinux for Yocto | Best fallback. Switching means re-implementing Hearth's video and WPE path as a plugin or platform view: weeks of work (inference) |
| Sony flutter-elinux | Embedder last commit 2025-05, tool 2025-07, plugins 2024-12. Video player copies through the CPU by default; no webview. Its own table says DRM-GBM does not work on i.MX 8M (Vivante blob lacks EGL on GBM) | Stale |
| Vendor support | TI: Processor SDK AM62x ships Flutter via meta-flutter as Sony's `flutter-wayland-client` on Weston (Sep 2026). Toradex: partner (Crossware) template using flutter-elinux, Flutter 3.27. NXP: latest i.MX Linux release notes (LF6.18.37_2.1.0, Sep 2026) do not mention Flutter | Thin; no vendor ships flutter-pi |

## Chip comparison

| SoC family | GPU and open driver (GLES) | Mainline display / DSI | Hardware decode (mainline) | PDM mic input | flutter-pi reports | Verdict |
|---|---|---|---|---|---|---|
| RK3588 / RK3588S | Mali-G610; Panthor + Panfrost, GLES 3.1 conformant (Jul 2024); Vulkan 1.4 | VOP2 6.8; DSI 6.16 | H.264, HEVC 7.0 (VDPU381, single core); AV1 6.7; VP9 patches sent | No PDM node in mainline DT or Collabora matrix | 2023 only, pre-Panthor | Most complete mainline stack; no 30 mm-class module found |
| RK3576 | Mali-G52 MC3; Panfrost, GLES 3.1 | VOP 6.15; DSI 6.18-rc1; PWM not upstream | H.264, HEVC 7.0 (VDPU383); VP9, AV1 WIP | No PDM node or matrix row | None found | Shortlist |
| RK3566 / RK3568 | Mali-G52 2EE; Panfrost, GLES 3.1 | VOP2 and 2x DSI in mainline DT | H.264, VP8, MPEG-2 only (Hantro); HEVC/VP9 (VDPU346) patches posted Dec 2025 to Jan 2026, not merged | Yes: `rk3568-pdm` node | UI works (2024); video failed on blob (Nov 2024) | Cheap mature fallback, H.264 only |
| Amlogic S905X3 / S905D3 / A311D | Mali-G31 / G52; Panfrost, GLES 3.1 | meson-drm; DSI on S905D3 and A311D, not S905X3 | Staging stateful driver: H.264, VP9, MPEG-1/2; no HEVC | Yes: `sm1-pdm` | S905Y4 artefacts (2026) | Not recommended; newer A311D2 and S928X have no mainline GPU |
| MediaTek Genio 510 / 700 / 1200 | Mali-G57; Panfrost, GLES 3.1 | mediatek-drm, 2x DSI; DSI on EVKs from 6.14 | Stateless H.264, HEVC, VP9, AV1 (MT8188 / MT8195) | `dmic-codec` in mainline Genio DT; driver details not read | None found | Strong on paper; no orderable module fits a 46 mm board |
| NXP i.MX 8M Plus | Vivante GC7000UL; etnaviv GLES 3.0 conformant (Sept 2026); BSP blob GLES 3.1 | LCDIF + Samsung DSIM (unverified) | Hantro; HEVC added 2021; mainline G2 HEVC/VP9 quirks (unverified). BSP: 1080p60 H.264/HEVC/VP8/VP9 | MICFIL 8-ch (mainline driver unverified) | None found | Shortlist |
| NXP i.MX 8M Mini | GC NanoUltra; BSP blob GLES 2.0; etnaviv not checked | Not checked | BSP: H.264/HEVC/VP8/VP9; mainline not checked | MICFIL 8-ch (unverified) | None found (8M Nano blob artefacts, 2022) | Simple UIs only |
| NXP i.MX 93 | No 3D GPU (2D PXP only) | 4-lane DSI | None; no VPU (unverified) | 8-ch PDM | Software rendering only | Ruled out |
| NXP i.MX 95 | Mali-G310; Panthor + Panfrost GLES 3.1 "functional and stable" (NXP UG10159), but release notes say OSS conformance not passed and DDK is default; BSP blob GLES 3.2 | DPU not checked | Wave6 stateful H.264/HEVC; upstream v4 Oct 2025, merge not confirmed | Not checked | None found | Plausible, untested |
| TI AM62x | PowerVR AXE-1-16M; Mesa Vulkan 1.2, not conformant; GLES only via Zink | No DSI (OLDI/LVDS, RGB) | None; no VPU (unverified) | Not found (McASP) | AM625 UI works (2024) | Ruled out |
| TI AM62P / AM67 | PowerVR BXS-4-64; Mesa Vulkan 1.2 conformant; GLES only via Zink. TI SDK ships DDK 26.1 blob | 4-lane DSI; GPU kernel support queued for 6.20/7.0; Cadence DSI mainline not confirmed | Wave5 H.264/HEVC (upstream since about 6.8, unverified; DT enablement not confirmed) | Not confirmed | None found | Possible on the TI blob; mainline GLES immature |
| Qualcomm QCS6490 | Adreno 643; freedreno, GLES 3.2 (unverified) | 4-lane DSI; DPU + DSI upstream (unverified) | Iris: H.264, HEVC, VP9 upstream | 3 DMIC ports (datasheet excerpt); upstream depends on AudioReach | None found (DB410c, 2020) | Promising mainline; UBWC import untested |

Per-family notes:

- **Mesa and Mali.** From Mesa 27.0 every Valhall or newer Mali (G57, G610, and G310 by the same table) uses the new Kraid compiler; Mesa 26.3 is the last release with the old one. For a 24/7 product, pin Mesa 26.3.x and soak-test 27.0 before adopting it. Bifrost G52 and G31 are not affected. Mesa lists GLES 3.1, not 3.2, for every Mali generation; that is enough for Flutter and WPE (inference).
- **RK3588.** Mainline decode uses one VDPU381 core; multi-core support is pending and parallelises separate streams, not one stream. HEVC on VDPU381/383 needs GStreamer 1.28 or newer (released Jan 2026). A DSI kiosk needs kernel 6.16 or newer for DSI and 7.0 or newer for HEVC. No 24/7 Panthor stability report and no flutter-pi or WPE report on Panthor were found. The vendor MPP path runs on the Rockchip 6.1 kernel with the libmali blob, the combination that broke the video player.
- **RK3576.** Mainline is moving fast (Collabora and Flipper). PWM is not upstream, which matters for a PWM backlight; check whether the backlight can be driven over I2C instead (the stock LP8556 sits on I2C at 0x2C). G52 MC3 is much weaker than G610; heavy WPE pages plus video could be tight (no benchmark read). Module vendor BSPs are Linux 6.1.
- **RK3566 / RK3568.** The only Rockchip parts with a PDM node in mainline. Mainline decode is H.264 only through Hantro (`v4l2slh264dec`), limited to 1080p60 (unverified).
- **Amlogic.** The mainline decoder is a stateful staging driver, so GStreamer integration differs from Rockchip and MediaTek. The main BayLibre status page dates from Oct 2020; LibreELEC HEVC notes are from 2023.
- **MediaTek Genio.** The vendor IoT Yocto BSP (v26.0, Jul 2026) ships kernel 6.6 with the Mali DDK r48 blob, so mainline means Collabora Debian images or your own Yocto. Decoder output is a tiled format that may need a detiling step before GLES import (unverified).
- **i.MX 8M Plus.** On mainline: etnaviv (Mesa 26.x) with stateless Hantro decode. On the BSP: galcore GLES 3.1 with stateful decode. Tiling and modifier compatibility between Hantro output and the GPU must be tested. A Sep 2026 forum user on etnaviv still saw only ES 2.0 before conformance work reached a release. The 8QM alignment patch was for that chip's display controller; whether 8M Plus needs similar handling is not known.
- **i.MX 8M Mini.** Flutter runs on ES 2.0, but the flutter-pi research calls it a poor fit for Impeller and recent Flutter, and WPE plus video compositing would be tight. The two notes differ in emphasis here; both rule it out for anything but simple UIs.
- **i.MX 95.** Arm's proprietary DDK stops getting new features for G310 in 2028 (critical fixes and security patches only after that), which favours designing for Mesa. A Toradex user's Mesa swap on Verdin iMX95 failed under Weston (Mar 2026), most likely because kernel and device tree were unchanged (inference). Wave6 merge, GPU DTS and display controller mainline status are unconfirmed. 15-year availability.
- **AM62x and i.MX 93: ruled out.** AM62x has no MIPI DSI and no VPU (VPU absence unverified). i.MX 93 has no 3D GPU and no VPU.
- **AM62P / AM67.** On the TI BSP (DDK blob, out-of-tree `pvrsrvkm`, Wave5 stateful V4L2) every primitive flutter-pi needs is present: the AM67A SDK 10.00 (2024) lists `EGL_EXT_image_dma_buf_import(_modifiers)`, `EGL_KHR_surfaceless_context` and `GL_OES_EGL_image_external`. Whether DDK 26.1 on AM62P keeps them was not confirmed. On mainline, GLES is Zink over the PowerVR Vulkan driver, an extra translation layer.
- **QCS6490.** Qualcomm Linux (meta-qcom, kernel 6.18) defaults to upstream drivers; KGSL, camera, audio and video blobs are optional. Iris decoder output is often UBWC-compressed NV12; whether freedreno imports it through GStreamer modifier negotiation is the open question, and forcing linear output costs bandwidth. A Sept 2026 Qualcomm forum question on Flutter, GBM and UBWC got only an email reply. Longevity to 2036-07. Production SoMs exist (Thundercomm C6490P, Lantronix Open-Q 6490CS, Quectel SG560D); their sizes were not checked.

## Module, system-in-package or chip-down

**The 46 mm board width is the real constraint.** Most solder-down modules are 40 to 45 mm on a side, which leaves no room for FPC connectors, microphones, amplifier and Wi-Fi on the 101x46 and 121x46 boards. The realistic set for those two boards is the 30 mm-class modules; the 117x62 Max board can also take 40 to 45 mm modules (inference).

**System-in-package.** No orderable SiP (SoC plus DDR, optionally PMIC) with 4-lane DSI, a 3D GPU and a VPU was found. The only current Linux-class SiPs found are Octavo's AM62x parts: OSD62x-PM (9x14 mm, 500-ball 0.5 mm, AM62x + 1 GB DDR4, 4-layer layout, $22.50) and OSD62x (21x21 mm, with PMIC, beta). Both lack DSI and a VPU, so they are ruled out. No Rockchip, i.MX 8M/9, MediaTek or AM62P SiP was found; absence is not proven.

**Solder-down modules that fit a 46 mm board:**

| Module | Size | SoC | Wi-Fi on module | Price seen | Caveats |
|---|---|---|---|---|---|
| Boardcon LGA3576 | 45 x 30 mm, 330-pad LGA, 1.25 mm pitch | RK3576 | No | Not public; RK3576 LGA modules likely about $75 to $100 at qty 1 (inferred from MYIR's 2024 prices) | DSI lane count and PDM not stated; BSP Linux 6.1; RK806S-5 PMIC; height not found |
| Ka-Ro QSXP (QSXP-ML81) | 29 x 29 x 2.6 mm, 108 pads, 1 mm pitch | i.MX 8M Plus | No | Mouser US $312.35 qty 1, $291.90 qty 10; Mouser LU EUR 172.44 (inconsistent) | 4-lane DSI, 8-ch PDM, 18x I2S/TDM, 2x CSI; Yocto, Debian |
| Ka-Ro QSXM (QSXM-MM60) | 29 x 29 x 2.6 mm, 108 pads | i.MX 8M Mini | No | $227.09 and $161.89 seen | GLES 2.0 GPU; one Mouser listing says the factory is not taking orders |
| Ka-Ro QS95 | 29 x 29 mm | i.MX 95 | No | Not found | 4-lane DSI; expensive SoC (inference) |
| Ezurio Carbon AM67 OSM-MF | 45 x 30 mm | TI AM67x | Optional Sona TI351 (Wi-Fi 6 + BT 5.4) or IF513 (Wi-Fi 6E); certifications not stated on page | About $338 (4GB/16GB) to $594 (8GB) | Ships Q4 2026; 10+ year availability; PDM not confirmed |
| Grinn GenioSOM-360 / 360P | 30 x 30 mm, LGA-303 | Genio 360 / 360P (Mali-G57 MC2) | Not stated | Not found | Not yet orderable (2026-07-31); DSI lanes not stated |

i.MX 93 modules in the same size class (Ka-Ro QS93, Compulab MCM-iMX93, OSM-S) fit physically but are ruled out by the SoC.

**40 to 50 mm modules (Max board only):**

| Module | Size | SoC | Notes |
|---|---|---|---|
| MYIR MYC-LR3576 | 43 x 45 mm, 381-pin LGA | RK3576 | 2x 8-ch PDM, 1x DSI; $75 / $99 (2024-11-06); 12-layer module PCB |
| Geniatech SOM-3566-OSM | 45 x 45 mm (OSM-L) | RK3566 | 8-ch PDM; no prices to individuals |
| Digi ConnectCore 8M Mini | 40 x 45 x 3.5 mm | i.MX 8M Mini | Pre-certified 802.11ac + BT 5 on module (US, CA, EU, JP, AU/NZ) |
| Grinn GenioSOM-510 / 700 | 37.0 x 42.6 mm, LGA-312 | Genio 510 / 700 | 2x 4-lane DSI; open BSP |
| Advantech AOM-2421 | 45 x 45 mm (OSM-L) | Genio 520 | 1x 4-lane DSI |

No LGA modules were found from Radxa (CM3/CM4/CM5 are board-to-board), Firefly, FriendlyElec, Variscite, SolidRun or Seeed; this is a gap, not a confirmed absence.

**Chip-down.** Published break-even analyses put chip-down past roughly 50k to 100k units (Oxit 2025, for a simpler module); Digi says it "tends to make sense only at very high volumes". At hundreds to low thousands of units, DDR layout, bring-up and RF certification NRE is unlikely to be recovered (inference).

| SoC | Layer-count evidence | Effort |
|---|---|---|
| RK3566 | Rockchip thermal simulation on 4 layers; open 6-layer LPDDR4 devkit | Most tractable: many reference designs, RK809/RK817 PMIC templates; 0.4 mm-pitch fanout is the hard part |
| RK3576 | MYIR's module PCB is 12 layers | Hard |
| i.MX 8M Mini | NXP validation board 8 layers (HDG, 2019); Phytec carriers from 6 layers | Higher |
| AM62x (via OSD62x-PM SiP) | 4 layers, 3/3 mil | Lowest, but no DSI |

**Wi-Fi certification.** Most modules need a separate certified Wi-Fi module on the carrier. Digi ConnectCore has pre-certified Wi-Fi on the module; Ezurio Carbon offers on-module Wi-Fi but its certifications were not confirmed. Every Rockchip, Boardcon, MYIR and Ka-Ro option needs a separate module.

## Recommendation

This is the research's recommendation, not a decision.

- **Shortlist:** RK3576 on the Boardcon LGA3576 (fits 46 mm, strong and fast-moving mainline: Panfrost, DSI, H.264/HEVC decode) and i.MX 8M Plus on the Ka-Ro QSXP (smallest module, 8-ch PDM, mature decode, etnaviv GLES 3.0 conformant Sept 2026).
- **Fallback:** RK3566/RK3568, cheap and mature with mainline PDM, but limited to H.264 decode under mainline.
- **Next step:** buy a dev board for the top one or two, run the go/no-go checks, then run Hearth itself (UI, video and WPE) before choosing.

## Bench evaluation boards

Research date: 2026-11-17. Nothing has been bought. Mainline status was checked against the device trees in `torvalds/linux` master (7.4-rc3).

**Neither product module's official carrier suits the bench test.** Ka-Ro's QSBASE4 for the QSXP wires only 2 DSI lanes (15-pin Raspberry-Pi-style connector), and mainline has no QSXP device tree; the kit is €265 and backordered. Boardcon's Idea3576 for the LGA3576 has 4-lane DSI on a 2x13 header with the pinout in its manual, but no mainline device tree (vendor 6.1.141 kernel), no I2S or PDM header, and no public price.

**No RK3576 board has DSI working in mainline yet.** The DSI controller, PHY and bridge are in the SoC device tree since 6.18-rc1, but Collabora's status page still lists DSI as TODO on its reference boards, and no RK3576 board file sets up a panel. A DSI + panel device-tree overlay is ours to write on any RK3576 board. The panel side is not a gap: the stock panels use the JD9364/JD9365, which the mainline `panel-jadard-jd9365da-h3` driver covers (see the driver tracker).

### RK3576

| Board | Price | DSI connector | Audio on header | Mainline / distro |
|---|---|---|---|---|
| **Radxa ROCK 4D** (recommended) | About $99 (Allnet; variant not stated); sold out at Arace | 39-pin 0.3 mm FPC, 4 lanes, pinout and schematics published | SAI2, SAI3 and PDM0 on the 40-pin header; SPI1/2, several I2C | Mainline DT; Collabora reference board; onboard audio patch submitted, not merged; DSI TODO; Armbian Standard |
| ArmSoM Sige5 (runner-up) | $164 for 8 GB (out of stock at ArmSoM) | 34-pin 0.5 mm FPC, 4 lanes, pinout published | SAI0/1/3/4 and PDM0 on the 40-pin header | Mainline DT with ES8388 sound (6.16); Collabora reference board; Armbian Community |
| FriendlyElec NanoPi M5 | $75 | 30-pin 0.5 mm FPC, 4 lanes, pinout and schematics | No SAI or PDM on the header | Mainline DT with sound; vendor 6.1 images |
| Firefly ROC-RK3576-PC | $259–289 | 30-pin 0.5 mm, 4 lanes; pinout not checked | 20-pin header, I2C/SPI; no I2S found | Mainline DT, no sound node |
| Boardcon Idea3576 | Not published | 2x13 header, 4 lanes, pinout in manual | No I2S/PDM header | Vendor 6.1.141 only |

The ROCK 4D's 39-pin 0.3 mm DSI connector is the same class as the 1st gen BOE panel's I-PEX connector. That may make an adapter flex simpler, but the pin assignments are unknown and almost certainly differ; never connect the panel directly.

### i.MX 8M Plus

| Board | Price | DSI connector | Audio on header | Mainline / BSP |
|---|---|---|---|---|
| **NXP FRDM-IMX8MPLUS** (recommended) | $250 at NXP; from $220.59 at Future Electronics | J15, 22-pin 0.5 mm FPC, 4 lanes; schematics and design files from NXP, no pin table in the user manual | I2S/SPI/I2C on the 40-pin header; WM8962 codec | Mainline DT, DSI and sound not enabled; NXP 6.6.36 images |
| Polyhex DEBIX Model A (runner-up) | $146–299, in stock | 24-pin 0.5 mm, 4 lanes (from a search snippet) | SPI, I2C; I2S "via software config"; no PDM listed | Mainline DT (DSI not enabled); Yocto 6.6.36, Ubuntu, Debian 12; schematics |
| Toradex Verdin iMX8MP + Dahlia | Dahlia kit $196.40 + module (not priced) | Samtec 60-pin, 4 lanes, pinout in datasheet (2024) | I2S on the same connector; WM8904 codec | Mainline DTs with sound; Flutter shown via meta-flutter |
| NXP 8MPLUSLPD4-EVK | $842 | Mainline enables 4-lane DSI to an HDMI bridge; physical connector not verified | Mainline enables PDM mics (MICFIL) and SAI3 + WM8960 | Most complete mainline DT |
| Ka-Ro QSBASE4 + QSXP | €265, backordered | 2 lanes only | No SAI documented | No QSXP DT in mainline |

### Why these two

- **ROCK 4D:** mainline device tree, a Collabora reference board (board-level DSI should land there first), Armbian Standard support, a published 4-lane DSI pinout, I2S and PDM pins on the header, and the lowest price among the well-supported boards.
- **FRDM-IMX8MPLUS:** mainline device tree, 4-lane DSI on a common 0.5 mm 22-pin connector, full NXP schematics and design files, I2S on the header. Adding the DSI and audio nodes ourselves is work the QSXP product board needs anyway.

No evidence was found of flutter-pi, WPE `wpevideosrc` or GStreamer dmabuf import working on any of these boards.

### Bench test plan

1. Boot a mainline image (Armbian or Debian on the ROCK 4D; Debian or our own build on the FRDM) and record kernel and Mesa versions.
2. Run the go/no-go checks above (`eglinfo`, KMS atomic, GStreamer hardware decode with dmabuf output).
3. Build Hearth's flutter-pi fork, rebased onto upstream #529, and run Hearth on HDMI: UI, a 1080p H.264 stream, an HEVC stream, and a Home Assistant dashboard through WPE.
4. Run Linux Voice Assistant with an I2S or PDM microphone on the header.
5. Only then: write the DSI panel overlay and connect a Nest Hub panel through an adapter flex, once its pinout is measured.

### Sources

- Radxa ROCK 4D: https://www.radxa.com/products/rock4/4d/ ; https://docs.radxa.com/en/rock4/rock4d/hardware-use/hardware-info ; https://docs.radxa.com/en/rock4/rock4d/hardware-use/mipi-dsi ; https://docs.radxa.com/en/rock4/rock4d/hardware-use/pin-gpio ; https://www.armbian.com/radxa-rock-4d/ ; https://shop.allnetchina.cn/products/rock-4d ; https://arace.tech/products/radxa-rock-4d
- ArmSoM Sige5: https://docs.armsom.org/armsom-sige5 ; https://www.armsom.org/product-page/sige5 ; https://www.armbian.com/armsom-sige5/ ; https://sbc.compare/armsom-sige5
- FriendlyElec NanoPi M5: https://wiki.friendlyelec.com/wiki/index.php/NanoPi_M5 ; https://www.friendlyelec.com/index.php?route=product/product&product_id=309
- Firefly ROC-RK3576-PC: https://www.firefly.store/products/roc-rk3576-pc-octa-core-6t-aiot-mini-computer ; https://www.t-firefly.com/products/roc-rk3576-pc-octa-core-6t-aiot-mini-computer
- Boardcon Idea3576 / LGA3576: https://www.armdesigner.com/Idea3576_SBC ; https://www.armdesigner.com/download/Idea3576_hardware_manual.pdf ; https://www.electronics-lab.com/news/boardcon-lga3576-som-and-idea3576-sbc-feature-rockchip-rk3576-soc-with-6-tops-npu-and-rk806s-5-pmic/
- Luckfox Omni3576 (not assessed): https://wiki.luckfox.com/Luckfox-Omni3576/Luckfox-Omni3576-Introduction/ ; https://spotpear.com/wiki/Raspberry-Pi-8-inch-DSI-MIPI-LCD-TouchScreen-Display-800x1280-Luckfox-Lyra-RK3506-Luckfox-Omni3576-ESP32-P4.html
- RK3576 mainline status: https://gitlab.collabora.com/hardware-enablement/rockchip-3588/notes-for-rockchip-3576/-/blob/main/mainline-status.md ; https://docs.flipper.net/one/cpu-software/rk3576-mainlining
- NXP FRDM-IMX8MPLUS: https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-IMX8MPLUS ; https://community.nxp.com/t5/FRDM-Training-Hub/FRDM-IMX8MPLUS-Hub/ta-p/2107907 ; https://www.futureelectronics.com/resources/featured-products/nxp-frdm-imx-8m-plus-development-board ; https://www.digikey.be/en/products/detail/nxp-usa-inc/FRDM-IMX8MPLUS/26918033 ; user manual https://static6.arrow.com/aropdfconversion/cbd2b3d04f7d52672c1ae245567a590050f9f46a/um12340.pdf
- Polyhex DEBIX Model A: https://debix.io/product/debix-model-a/ ; https://debix.io/blog/debix-model-a-gpio-instruction-manual/ ; https://device.report/manual/10193342
- Toradex Dahlia: https://www.toradex.com/products/carrier-board/dahlia-carrier-board-kit ; https://docs.toradex.com/116797-dahlia_carrier_board_datasheet.pdf ; Flutter demo https://www.industrialflutter.com/videos/
- NXP 8MPLUSLPD4-EVK: https://www.digikey.com/en/products/detail/nxp-usa-inc/8MPLUSLPD4-EVK/13897867
- Ka-Ro QSXP / QSBASE4: https://www.karo-electronics.de/qsxp/ ; https://karo-electronics.github.io/docs/hardware-documentation/baseboards/qsbase4/pinout.html ; https://karo-electronics.github.io/docs/hardware-documentation/baseboards/qsbase4/quickstart.html ; https://strategic-embedded.com/products/qsbase4-evalkit-qsxp-ml81
- SolidRun HummingBoard (mainline DSI overlay on the IIoT variant; Pulse/Mate not assessed): https://shop.solid-run.com/product/SRMP8-EVKHP-H8-R1.2/
- Mainline device trees: https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/rockchip ; https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/freescale

## Sources

What Hearth needs from a chip:
- flutter-pi `gl_renderer.c`: https://github.com/ardera/flutter-pi/blob/master/src/gl_renderer.c
- flutter-pi `modesetting.c`: https://github.com/ardera/flutter-pi/blob/master/src/modesetting.c
- flutter-pi `gstreamer_video_player/frame.c`: https://github.com/ardera/flutter-pi/blob/master/src/plugins/gstreamer_video_player/frame.c
- Hearth flutter-pi fork: https://github.com/chrisuthe/flutter-pi-hearth
- `wpevideosrc`: https://gstreamer.freedesktop.org/documentation/wpe/wpevideosrc.html

flutter-pi and other embedders:
- flutter-pi README and issues (#86, #125, #147, #249, #271, #327, #339, #356, #370, #392, #412, #421, #429, #438, #441, #447, #457, #460, #491, #494, #508, #513, #518, #521, #525, #528, #529; PRs #490, #522, #523, #534, #535): https://github.com/ardera/flutter-pi and https://github.com/ardera/flutter-pi/issues/N
- Sony flutter-elinux: https://github.com/sony/flutter-embedded-linux , https://github.com/sony/flutter-elinux/wiki/Support-status , https://github.com/sony/flutter-elinux-plugins , https://github.com/sony/flutter-elinux-plugins/tree/main/packages/video_player
- ivi-homescreen: https://github.com/toyota-connected/ivi-homescreen , https://github.com/toyota-connected/ivi-homescreen-plugins
- meta-flutter: https://github.com/meta-flutter/meta-flutter
- TI AM62x GUI Frameworks guide: https://software-dl.ti.com/processor-sdk-linux/esd/AM62X/latest/exports/docs/linux/Demo_User_Guides/GUI_Frameworks_User_Guide.html
- NXP i.MX Linux release notes RN00210: https://www.nxp.com/docs/en/release-note/RN00210.pdf
- Toradex templates and forum: https://github.com/torizon/vscode-torizon-templates , https://community.toradex.com/t/flutter-dart-template-fails-to-solve-crossware-app-5050-public-repos-flutter-sdk/30835

Chip comparison:
- Mesa Panfrost: https://docs.mesa3d.org/drivers/panfrost.html
- Collabora, Panthor GLES 3.1 conformance (2024): https://www.collabora.com/news-and-blog/news-and-events/taming-the-panthor-opengl-es-31-conformance-achived-mali-g610.html
- Collabora, Rockchip year in review (Mar 2026): https://www.collabora.com/news-and-blog/blog/2026/03/02/running-mainline-linux-u-boot-and-mesa-on-rockchip-a-year-in-review/
- Collabora, Kraid compiler (Oct 2026): https://www.collabora.com/news-and-blog/news-and-events/panfrost-is-now-using-kraid.html
- GStreamer 1.28: https://gstreamer.freedesktop.org/releases/1.28/
- RK3588 mainline matrix: https://gitlab.collabora.com/hardware-enablement/rockchip-3588/notes-for-rockchip-3588/-/raw/main/mainline-status.md
- RK3588 DT: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/rockchip/rk3588-base.dtsi
- Collabora, RK3588/RK3576 decoders merged (Feb 2026): https://www.collabora.com/news-and-blog/news-and-events/rk3588-and-rk3576-video-decoders-support-merged-in-the-upstream-linux-kernel.html
- Radxa Mali GPU doc: https://docs.radxa.com/en/rock5/rock5b/radxa-os/mali-gpu
- RK3576 mainline matrix: https://gitlab.collabora.com/hardware-enablement/rockchip-3588/notes-for-rockchip-3576/-/raw/main/mainline-status.md
- RK3576 DT: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/rockchip/rk3576.dtsi
- Collabora and Flipper, RK3576 (May 2026): https://www.collabora.com/news-and-blog/news-and-events/collabora-flipper-opening-up-the-rk3576.html ; https://docs.flipper.net/one/cpu-software/rk3576-mainlining
- RK356x DT: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/rockchip/rk356x-base.dtsi
- rkvdec driver: https://raw.githubusercontent.com/torvalds/linux/master/drivers/media/platform/rockchip/rkvdec/rkvdec.c
- Hantro Rockchip variants: https://raw.githubusercontent.com/torvalds/linux/master/drivers/media/platform/verisilicon/rockchip_vpu_hw.c
- VDPU346 patches: https://lists.infradead.org/pipermail/linux-rockchip/2025-December/066129.html , https://lists.openwall.net/linux-kernel/2026/01/11/413
- Amlogic DTs and staging vdec: https://api.github.com/repos/torvalds/linux/contents/arch/arm64/boot/dts/amlogic , https://raw.githubusercontent.com/torvalds/linux/master/drivers/staging/media/meson/vdec/vdec_platform.c
- linux-meson status (2020): https://linux-meson.com/hardware.html
- MediaTek DTs and decoder: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/mediatek/mt8188.dtsi , https://raw.githubusercontent.com/torvalds/linux/master/drivers/media/platform/mediatek/vcodec/decoder/mtk_vcodec_dec_stateless.c
- Collabora, Genio update (Mar 2025): https://www.collabora.com/news-and-blog/news-and-events/mediatek-genio-update-kernel-debian-13-images-and-kernelci.html
- MediaTek IoT Yocto v26.0: https://genio.mediatek.com/doc/iot-yocto/latest/sw/yocto/release-notes/iot-yocto-v26.0-release-note.html
- Igalia, etnaviv GLES 3.0 conformant (Oct 2026): https://www.igalia.com/2026/etnaviv-is-now-OpenGL-ES-3.0-conformant.html
- NXP forum, etnaviv ES 2.0 on 8M Plus (Sep 2026): https://community.nxp.com/t5/i-MX-Processors/i-MX-8M-Plus-GC7000UL-only-reports-OpenGL-ES-2-0-via-Mesa/m-p/2413115
- Collabora, HEVC for i.MX 8M (2021): https://www.collabora.com/news-and-blog/blog/2021/02/17/adding-hevc-support-for-imx-8m/
- i.MX 93 GPU: https://forum.fs-net.de/index.php?thread/4819-grafik-starter-kit/
- NXP UG10159: https://docs.nxp.com/bundle/UG10159/page/topics/current_status_and_limitations.html , https://docs.nxp.com/bundle/UG10159/page/topics/mali_ddk_feature-freeze_lifecycle.html
- Toradex i.MX 95 Panfrost thread: https://community.toradex.com/t/using-mesa-panfrost-instead-of-mali-imx-proprietary-library-on-verdin-imx95-weston-compositor-failure/30087
- Wave6 driver: https://lwn.net/Articles/1035632/
- Mesa PowerVR: https://docs.mesa3d.org/drivers/powervr.html
- Phoronix, BeaglePlay open PowerVR stack (Jan 2026): https://www.phoronix.com/news/BeaglePlay-PowerVR-Success
- Phoronix, AM62P in Linux 7.0 (Jan 2026): https://www.phoronix.com/news/Imagination-AM62P-Linux-7.0
- TI AM62Px SDK release notes: https://software-dl.ti.com/processor-sdk-linux/esd/AM62PX/latest/exports/docs/devices/AM62PX/linux/Release_Specific_Release_Notes.html
- TI AM62Px Wave5 (2024): https://software-dl.ti.com/processor-sdk-linux/esd/AM62PX/09_02_01_09/exports/docs/linux/Foundational_Components_Multimedia_wave5.html
- TI AM67A Rogue graphics overview (2024): https://software-dl.ti.com/jacinto7/esd/processor-sdk-linux-am67a/10_00_00/exports/docs/linux/Foundational_Components/Graphics/Rogue/Overview.html
- QCS6490 status: https://www.techveda.live/qcs6490/
- Qualcomm Linux overlays and video: https://dragonwingdocs.qualcomm.com/Key-Documents/Migration-Guide/enable-and-evaluate-qualcomm-linux-overlays , https://dragonwingdocs.qualcomm.com/Technologies/Video/build-video-source-components
- Iris Gen2 on SC7280: https://lwn.net/Articles/1065596/
- Qualcomm forum, Flutter on QCS6490: https://mysupport.qualcomm.com/supportforums/s/question/0D5dK00000U2eMiSAJ/
- QCS6490 data sheet: https://docs.qualcomm.com/doc/80-23889-1/topic/Introduction.html

Module, system-in-package or chip-down:
- Octavo OSD62x-PM and OSD62x: https://octavosystems.com/octavo_products/osd62x-pm/ , https://octavosystems.com/octavo_products/osd62x/ , https://octavosystems.com/app_notes/osd62x-pm-layout-guide/
- AM62P datasheet (4-lane DSI): https://www.ti.com/lit/ds/symlink/am62p.pdf
- Boardcon LGA3576: https://www.electronics-lab.com/news/boardcon-lga3576-som-and-idea3576-sbc-feature-rockchip-rk3576-soc-with-6-tops-npu-and-rk806s-5-pmic/
- Ka-Ro QSXP, QSXM, QS93, QS95: https://www.mouser.com/datasheet/2/616/QSXP-2326726.pdf , https://www.glyn.de/Daten/DSS/Ka-Ro/Sales-Information/PromoPackage/QSMODULE/QSXM.pdf , https://www.karo-electronics.de/qs93/ , https://www.karo-electronics.de/qs95/
- Ka-Ro prices: https://www.mouser.com/ProductDetail/Ka-Ro-electronics/QSXP-ML81?qs=TuK3vfAjtkXTos%252BT9dtpkA%3D%3D , https://www.mouser.co.il/ProductDetail/Ka-Ro-electronics/QSXM-MM60?qs=QNEnbhJQKvYxUbulBQj1Sw%3D%3D
- Ezurio Carbon AM67: https://www.ezurio.com/system-on-module/osm-som/carbon-am67-som , https://linuxgizmos.com/tiny-osm-mf-module-runs-linux-on-sitara-am67x-with-4-tops-ai/
- Grinn GenioSOM: https://grinn-global.com/blog/mediatek-genio-360-360p-510-700-som-system-on-modules-grinn
- GLYN solder-down list: https://www.glyn.com/products/computer-on-module/solder-down-modules/
- MYIR MYC-LR3576 (2024): https://www.myirtech.com/news_list.asp?id=907
- Geniatech SOM-3566-OSM: https://www.geniatech.com/product/som-3566-osm/
- Digi ConnectCore 8M Mini: https://www.digi.com/products/embedded-systems/digi-connectcore/system-on-modules/digi-connectcore-8m-mini
- Compulab MCM-iMX93 (2024): https://www.compulab.com/products/computer-on-modules/mcm-imx93-nxp-i-mx-93-som-smd-system-on-module/
- Radxa CM3 / CM4: https://www.radxa.com/products/cm/cm3/ , https://www.radxa.com/products/cm/cm4/
- RK3566 Hardware Design Guide (2022): https://dl.xkwy2018.com/downloads/RK3568/RK356X/Hardware/Rockchip_RK3566_Hardware_Design_Guide_V1.1_EN.pdf ; RK3566 devkit reference: https://groundplanestudio.com/silicon/rk3566
- i.MX 8M Mini HDG (2019): https://www.zlgmcu.com/data/upload/file/highmpu/i.MX-8MMini-yhsc.pdf ; Phytec carrier layers: https://www.phytec.de/cdocuments/?doc=SwBeEw
- Break-even: https://oxit.com/module-or-chip-down-design-a-cost-comparison-and-breakeven-analysis/ , https://www.digi.com/blog/post/choose-the-starting-point-embedded-system-design

---

# Reference: platform research notes

These notes were written for the earlier Android plan and are kept for reference; their Raspberry Pi and Android-build material is superseded by "Linux stack and chip selection" above.

Research date: 2026-11-17. Method note: ~25 search/fetch calls. Most findings come from search-result excerpts or targeted page extractions, not full-page reads; where a claim rests only on an excerpt it is marked "(excerpt)". Items in "Inferences" marked "(background, unverified)" come from the researcher's prior knowledge and were NOT confirmed against a source in this session; treat them as leads, not facts.

## 1. Which SoMs are realistic candidates? (price, size, connector, DSI/CSI, audio, wireless, power, longevity)

### Takeaway
The realistic hobbyist shortlist is Raspberry Pi CM5 (best community Android, two 4-lane DSI/CSI ports, but memory-driven price rises through 2026), Radxa CM4/RK3576 and CM5/RK3588S (vendor Android 12/14 BSP, native Rockchip audio/display tooling, cheaper), with NXP i.MX 8M Mini/Plus SoMs the best-maintained vendor Android (Android 17 in Nov 2026) but targeted at EVKs and commercial SoM vendors. Qualcomm QCS6490 boards exist with Android 13-16 but are SBCs or commercial SoMs, not hobby-friendly modules.

### Cited Findings
**Raspberry Pi CM5 (BCM2712)**
- Two 100-pin high-density connectors (pins 1-100 top, 101-200 bottom); board 55 mm x 40 mm; "two 4-lane MIPI interfaces" each usable as CSI or DSI — [CM5 datasheet](https://pip.raspberrypi.com/documents/RP-008180-DS-cm5-datasheet.pdf), [Mouser copy](https://www.mouser.com/datasheet/2/635/cm5_datasheet-3537283.pdf)
- CM5 kept the CM4-style dual-100-pin form factor but replaced CM4's 2-lane MIPI interface pins (CAM0/DSI0) with two USB 3.0 ports — [DigiKey blog](https://www.digikey.com/en/maker/blogs/2025/meet-the-compute-module-5-whats-new-and-improved), [antmicro cm4-baseboard README](https://github.com/antmicro/cm4-baseboard)
- Launch pricing "from $45" — [Farnell CM5 datasheet copy](https://www.farnell.com/datasheets/4425770.pdf). Prices have since risen repeatedly: 2 Feb 2026 (+$10 2GB, +$15 4GB, +$30 8GB, +$60 16GB across Pi 4/5 and CM4/CM5), 1 Apr 2026 (+$12.50 2GB, +$25 4GB, +$50 8GB, +$100 16GB), 2 Nov 2026 (+$40 on all 16GB CM5). CM5 2GB Lite no-wireless was $55 after Feb 2026 and $67.50 after Apr 2026 — [raspberry.tips price/lifecycle tracker](https://raspberry.tips/en/raspberrypi-infos/raspberry-pi-prices-lifecycle), [Raspberry Pi news Feb 2026](https://www.raspberrypi.com/news/more-memory-driven-price-rises/), [Raspberry Pi news 2 Nov 2026](https://www.raspberrypi.com/news/price-rises-for-some-16gb-raspberry-pi-products/)
- 16GB CM5 "now starts at $375" (Nov 2026) — [Liliputing via Facebook (excerpt)](https://www.facebook.com/Liliputing/posts/the-raspberry-pi-5-with-16gb-of-ram-now-costs-385-after-an-80-price-hike-while-t/1716068097189037/)
- CM5IO reference carrier is 160 x 90 mm with two dual-purpose DSI/CSI-2 22-pin 0.5 mm FPC connectors — [CM5IO datasheet](https://pip.raspberrypi.com/documents/RP-008182-DS-cm5io-datasheet.pdf)

**Raspberry Pi CM4 (BCM2711)**
- July 2026 list prices from product brief: CM4 1GB Lite no-wireless $41.25; 2GB Lite $57.50; 4GB Lite $90; 8GB Lite $155; 8GB/64GB eMMC/wireless $195 — [raspberry.tips tracker citing CM4 product brief](https://raspberry.tips/en/raspberrypi-infos/raspberry-pi-prices-lifecycle)
- Same Hirose DF40C-100DS-0.4V mating connectors on the carrier (~$2.39 each at DigiKey at time of article); official CM4IO KiCad files are open — [DigiKey/Shawn Hymel](https://www.digikey.com/en/maker/projects/creating-a-raspberry-pi-compute-module-4-cm4-carrier-board-in-kicad/7812da347e5e409aa28d59ea2aaea490)

**Radxa CM5 (RK3588S2)**
- 55 x 40 mm; 2x MIPI DSI, 2x MIPI CSI on the CM5 IO board; VCC_SYSIN 3.6-5.2 V (4 V/3 A recommended) — [Radxa CM5 docs](https://docs.radxa.com/en/som/cm/cm5), [CM5 IO board](https://www.radxa.com/products/io-board/cm5-io-board/)
- RK3588S/RK3582 options; RAM 2-32 GB — [Radxa CM5 product page (excerpt)](https://www.radxa.com/products/cm/cm5/)

**Radxa CM4 (RK3576/RK3576J)**
- 4x A72 @2.2 GHz + 4x A53, Mali-G52 MC3, 6 TOPS NPU; launched at "$70 and up" (Aug/Sep 2025) — [Radxa CM4 docs](https://docs.radxa.com/en/som/cm/cm4), [Liliputing](https://liliputing.com/radxa-cm4-with-an-rk3576-processor-could-be-a-cheaper-alternative-to-the-radxa-cm5-with-rk3588/)
- Radxa CM3 = RK3566 (4x A55 @1.8 GHz, Mali-G52 2EE); the comparison table lists 55 x 40 mm — [Liliputing comparison table](https://liliputing.com/radxa-cm4-with-an-rk3576-processor-could-be-a-cheaper-alternative-to-the-radxa-cm5-with-rk3588/)
- AliExpress listings for Radxa CM4/CM5 in late 2026 show far higher prices (e.g. $142.79 for CM4 2GB/32GB) — [AliExpress listing (excerpt; reseller price, unreliable)](https://www.aliexpress.com/item/1005013087937503.html)

**NXP i.MX 8M Mini / Plus**
- NXP's Android BSP supports i.MX 8M Mini EVK and 8M Plus EVK in every release from Android 14 through Android 17 — [NXP Android for i.MX page](https://www.nxp.com/design/design-center/software/embedded-software/i-mx-software/android-os-for-i-mx-applications-processors:IMXANDROID)

**Qualcomm**
- Radxa Dragon Q6A (QCS6490 SBC, not a SoM): 1x 4-lane MIPI DSI, HDMI; OS support "Android 15 (CS) or Android 16 (ES)", Ubuntu, Qualcomm Linux — [Radxa Dragon Q6A docs](https://docs.radxa.com/en/dragon/q6a); "available for $60 and up" at launch — [Liliputing via Facebook (excerpt)](https://www.facebook.com/Liliputing/posts/radxa-dragon-q6a-is-a-raspberry-pi-sized-single-board-pc-with-a-qualcomm-qcs6490/1205091504953368/)
- Thundercomm RUBIK Pi 3 (QCS6490): Android 13, Qualcomm Linux, Debian 13; Thundercomm also sells C6490P and C8550 SoMs — [Thundercomm](https://www.thundercomm.com/product/rubik-pi/)

### Inferences
- Because CM5 dropped the 2-lane DSI0/CAM0 pair, a CM4-footprint carrier is not automatically DSI-compatible across both modules; design against DSI1/CAM1 (4-lane) positions if dual compatibility matters (follows from DigiKey/antmicro findings above).
- (background, unverified) CM4 exposes one 2-lane and one 4-lane DSI; CM4/CM5 expose PCM/I2S on GPIO18-21 and have no native multi-channel PDM input (RP1 on Pi 5/CM5 has a PDM-capable I2S block but Android HAL support is unknown). Rockchip RK3566/RK3576/RK3588 have dedicated PDM controllers (up to 8 mics) plus multiple I2S/TDM ports; i.MX 8M Mini/Plus have MICFIL PDM inputs (8 channels) and SAI. These make RK and i.MX natively better for 2-3 PDM mics than the Pi.
- (background, unverified) Radxa CM3/CM5 use 3x 100-pin connectors with a Pi-CM4-compatible subset; Radxa CM4 pinout should be checked in its hardware design guide.
- A 1024x600 or 1280x800 panel needs at most 4 lanes at modest bit rates; every candidate has enough DSI capacity, so the choice hinges on software, audio and price rather than display bandwidth.

### Gaps
- Exact Nov 2026 CM5 per-SKU list (4GB/8GB wireless+eMMC) not extracted; only deltas and a few SKUs confirmed. A search result for the product brief was too large to read in session.
- CM4/CM5 production-lifetime dates (commonly quoted as CM4 to at least Jan 2034, CM5 to at least Jan 2036) not confirmed in this session.
- Radxa CM4/CM5 current official prices, connector pin counts, DSI lane counts, on-module Wi-Fi/BT chip, and availability guarantees were not present in the pages read.
- No power-draw figures were retrieved for any module.
- No specific hobby-accessible i.MX 8M Mini/Plus SoM pricing (Variscite, Toradex Verdin, Compulab, SolidRun, etc.) was gathered; no "other Rockchip SoM" vendors (FriendlyElec CM3588, Firefly Core, Banana Pi, Turing RK1, LCSC TaishanPi) were researched beyond a passing TaishanPi DSI porting hit.
- No Qualcomm module in a CM4-like hobby form factor with public Android sources was found.

## 2. Android state per platform as of late 2026

### Takeaway
Raspberry Pi 4/5/CM5 have the most current community Android (KonstaKANG LineageOS 23.2 = Android 16 QPR2, build 20 May 2026) but with SELinux permissive, no userdata encryption, TWRP-based OTA and a non-commercial licence; NXP has the freshest vendor BSP (Android 17, Nov 2026); Radxa's Rockchip BSPs are stuck at Android 12-14 (latest Radxa manifests releases are 2025).

### Cited Findings
**Raspberry Pi (KonstaKANG LineageOS)**
- LineageOS 23.2 (Android 16 QPR2) for Pi 5; "Pi 5 build supports Raspberry Pi 5, Pi 500, and Compute Module 5"; latest build 20 May 2026 with 1 May 2026 security patch — [KonstaKANG rpi5 LineageOS 23 page (cached 16 Nov 2026)](https://konstakang.com/devices/rpi5/LineageOS23/), [RPi forum thread](https://forums.raspberrypi.com/viewtopic.php?t=392550)
- Working: HDMI/USB/BT audio, GPIO I2S DACs (e.g. HiFiBerry DAC+), V3D OpenGL + Vulkan, H.265 hardware decode, official Pi cameras and UVC, I2C/SPI/GPIO, RTC, Wi-Fi/BT, touch (official 7" DSI, USB, Waveshare SPI) — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- I2C sensor HAL support out of the box for MPU6050, LSM6DS3, LSM303DLHC, BME280/BMP280, and APDS9930 (ambient light + proximity) — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- DSI: official 7" touchscreen and Waveshare 7" DSI LCD (H) are selectable in Settings -> System -> Raspberry Pi settings -> DSI display; HDMI must be disconnected when using DSI — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- Known issues: "Various issues with CSI camera modules", SELinux permissive, userdata encryption unsupported — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- GMS: install MindTheGapps 16.0.0 arm64 via TWRP then factory reset. OTA: TWRP-flashable OTA zips; no OTA from builds older than 20260128 — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- Licence: includes CC BY-NC-SA 4.0 parts; "Commercial use is not allowed with this build" — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- KonstaKANG also publishes plain AOSP 15/16 builds for Pi 4/5 "for advanced users only", and a LineageOS 23.2 Android TV build (Jan 2026) — [RPi forum AOSP 15](https://forums.raspberrypi.com/viewtopic.php?t=376222), [XDA AOSP 16](https://xdaforums.com/t/dev-rom-aosp-android-16-for-raspberry-pi-4-pi-5.4744648/page-2), [X post](https://x.com/KonstaKANG/status/2016531180317725078)
- User report: "Only thing that doesn't work on lineage is TTS" on a Pi 5 install — [RealDash forum (anecdote)](https://forum.realdash.net/t/raspberry-pi-5-install-got-android-working-but-need-help-with-help-with-realdash-install/7186)

**GloDroid**
- GloDroidCommunity/raspberry-pi describes itself as "Android 15 for the Raspberry PI 4 series" — [GitHub (excerpt)](https://github.com/GloDroidCommunity/raspberry-pi). KonstaKANG and GloDroid are described as the two prominent AOSP distributions for Pi 4/5 — [Paradox Cat Medium article](https://medium.com/paradox-cat-tech-hub/everything-you-need-to-know-about-android-on-raspberry-pi-aa4b8eea72c6)

**AOSP cadence**
- "Effective in 2026 ... we will publish source code to AOSP in Q2 and Q4" — [source.android.com](https://source.android.com/). This slows community rebases relative to earlier years.

**Radxa / Rockchip**
- radxa/manifests releases: Radxa NX4 + C200 Android 14 (11 Nov 2025); Radxa CM4 Android 14 with Radxa 8-inch LCD (29 Aug 2025); ROCK 3C Android 14 in separate DSI and HDMI builds (18 Jul 2025); ROCK 5C Android 14 with Display 8HD on DSI (10 Aug 2025); ROCK 4D (28 May 2025); ROCK 5T Android 12 with Display 8HD/10FHD (26 Feb 2025). No GMS or OTA information on the releases page — [radxa/manifests releases](https://github.com/radxa/manifests/releases)
- ROCK 4D (RK3576) build docs reference "Android14_RK3576_Radxa_rk6" — [Radxa docs (excerpt)](https://docs.radxa.com/en/rock4/rock4d/other-os/android/low-level-dev/build-system)
- Radxa CM5 docs have an Android section (flash/use/compile) but the page read did not state a version — [Radxa CM5 docs](https://docs.radxa.com/en/som/cm/cm5)

**NXP i.MX**
- Current release: Android 17.0.0_1.0.0 on L6.18.37_2.1.0 BSP, Nov 2026, supporting i.MX 8M Plus EVK, 8M Mini EVK, i.MX 95 EVK/FRDM, i.MX 952 EVK. Prior: Android 16.0.0_2.0.0 (L6.18.20), 16.0.0_1.x (L6.12.x), 15.0.0_x (L6.6/L6.12), 14.0.0_x (L6.1/L6.6) — [NXP Android for i.MX (page revised 5 Nov 2026)](https://www.nxp.com/design/design-center/software/embedded-software/i-mx-software/android-os-for-i-mx-applications-processors:IMXANDROID), [NXP Android release notes RN00201](https://www.nxp.com/docs/en/release-note/RN00201.pdf)
- Sources are public at [github.com/nxp-imx-android](https://github.com/nxp-imx-android)

**Qualcomm**
- Dragon Q6A: Android 15 (CS) / Android 16 (ES) listed by Radxa; RUBIK Pi 3: Android 13 — [Radxa](https://docs.radxa.com/en/dragon/q6a), [Thundercomm](https://www.thundercomm.com/product/rubik-pi/)

### Inferences
- For a 1-10 unit personal build, KonstaKANG on CM5 gives the newest Android with Play Store via MindTheGapps and working GPU/Vulkan, but a custom DSI panel, custom touch and I2S audio topology require rebuilding the kernel/device tree (KonstaKANG publishes sources and build instructions; this was not re-verified here). The six-month gap since the last build (May -> Nov 2026) should be flagged as possibly stale: a newer build or a LineageOS 24 page may exist that this session did not find.
- Radxa's Android releases are one-shot BSP drops per board (2025 dates, Android 14) with no visible security-patch cadence; expect Android 14 on a ~6.1 Rockchip kernel to remain the practical ceiling for RK3566/RK3576/RK3588 vendor Android unless Rockchip's Android 15/16 SDK has reached Radxa (not found).
- NXP is the only vendor here shipping current-version AOSP with a modern kernel on a regular cadence, but only for its EVKs; SoM vendors each port it with their own delay.
- "GMS" on all of these means sideloaded community GApps on an uncertified device (device-ID registration with Google typically needed); Play Integrity-dependent apps may fail. (background, unverified)
- (background, unverified) Rockchip Android gives full VPU hardware decode/encode and Mali GPU via vendor blobs; i.MX 8M Mini has H.264/H.265/VP8/VP9 decode (Hantro) and GC NanoUltra GPU, 8M Plus adds GC7000UL and an NPU; Pi 5 has only HEVC hardware decode (confirmed above), so H.264/VP9/AV1 video is software-decoded on CM5.

### Gaps
- Whether KonstaKANG has builds newer than 20 May 2026 or an Android 17 line; status of CM4-specific builds (rpi4 LineageOS 23 page not read).
- GloDroid's current status for Pi 5/CM5 and Rockchip boards in 2026.
- Rockchip Android 15/16 SDK availability for RK3576/RK3588 and any Radxa CM3 (RK3566) Android release newer than Android 11/12.
- Community AOSP/LineageOS for Rockchip boards (none found in this session).
- Android kernel versions for the Radxa builds; OTA mechanism (A/B) on Radxa and NXP builds was not read (NXP is believed to support A/B OTA, unverified).

## 3. Bringing up an arbitrary MIPI DSI panel, I2C touch and ALS under Android/Linux

### Takeaway
A salvaged DSI panel needs three things nobody hands you: the vendor init command sequence, video timings, and D-PHY lane configuration; without them the result is a black screen with almost no diagnostics. Rockchip's BSP lets you describe all of this in device tree ("simple-panel-dsi" + panel-init-sequence), whereas Raspberry Pi and mainline need a C panel driver or a panel-simple entry plus an overlay.

### Cited Findings
- Raspberry Pi engineers' guidance (thread dated 25 Sep 2025): check for an in-tree driver first; otherwise you need to write a display driver and an overlay. Required inputs: an initialisation sequence, video timing restrictions, DSI D-PHY details (lane count, polarities) and the full display spec. Pitfalls: "almost no debug information"; a panel that dislikes the DSI data just stays black; vendor-supplied init info "frequently cannot be relied on" — [RPi forum: What does it take to write a DSI driver for Pi5/CM5](https://forums.raspberrypi.com/viewtopic.php?t=392157)
- CM4 user got a custom DSI display working "by using an overlay and a custom entry in the simple-panel.c file"; firmware-side (bootloader) DSI resolution was hardcoded — [raspberrypi/firmware issue #1819](https://github.com/raspberrypi/firmware/issues/1819)
- On CM4 the overlay selects the DSI controller, creates the panel node and assigns GPIOs — [Kadi Display blog (vendor content)](https://www.kadidisplay.com/blog-news/how-a-linux-mipi-dsi-panel-driver-works-on-raspberry-pi-cm4-timing-initialization-lanes-and-device-tree/)
- "The carrier or adapter board supplies the panel's power rails and backlight drive, and the software sends the panel's initialization sequence" — [Orient Display (vendor content)](https://orientdisplay.com/raspberry-pi-prototype-to-production-display/)
- Rockchip BSP pattern: `&dsi { panel@0 { compatible = "simple-panel-dsi"; reset-gpios; rockchip,lane-rate ... } }` with panel parameters in DTS — [youyeetoo Tinker Board 2 wiki](https://wiki.youyeetoo.com/en/tinker/page/DebianSystem/mipi-dsi), [even629 Linux LCD notes](https://even629.com/en/posts/2602173/), [EEWorld TaishanPi porting project](https://en.eeworld.com.cn/Reference_Designs/detail/94618)
- Radxa ships separate DSI and HDMI Android images for ROCK 3C, i.e. display selection is baked into the image/DTB — [radxa/manifests releases](https://github.com/radxa/manifests/releases)
- KonstaKANG exposes only preset DSI panels via a settings toggle (official 7", Waveshare 7" (H)) — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)
- KonstaKANG's sensor HAL already handles an APDS9930 ALS over I2C, showing an IIO-sensor-to-Android path exists on Pi — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)

### Inferences
- For the salvaged panel, the init sequence must be captured from the original device (logic-analyser/DSI sniff is hard; easier is dumping the stock kernel's panel driver/DTB or finding the panel's driver IC datasheet). This is the single largest technical risk of the whole retrofit.
- (background, unverified) Typical work list: (a) DT panel node with reset/enable GPIOs, regulators, `dsi-lanes`, mode flags (video burst/sync-pulse, LPM), timings; (b) init sequence in DT (Rockchip `panel-init-sequence`) or in a `drivers/gpu/drm/panel/panel-*.c` driver (mainline/Pi/i.MX); (c) backlight: LED boost driver IC (e.g. TPS61165/TPS61169, MP3302, LP8556/LP855x, TPS65132 for AVDD/AVEE bias rails if the panel needs +/-5.x V) driven by PWM -> `pwm-backlight` node, or I2C backlight driver; Android lights/brightness HAL then writes sysfs backlight; (d) bootloader splash needs separate U-Boot support, optional.
- (background, unverified) Touch: mainline `goodix` (GT911/GT9xx), `edt-ft5x06` (FocalTech FT5x06/FT5x26/FT8xxx), `ilitek_ts_i2c`, `hid-over-i2c` drivers bind via an I2C DT node with interrupt + reset GPIOs; Android consumes them as standard evdev multi-touch with no HAL work beyond an optional `.idc` file for orientation/calibration. Rockchip BSP uses its own `gt9xx` vendor driver with a config blob.
- (background, unverified) ALS: kernel IIO driver (e.g. tsl2563/tsl2591, veml6030, opt3001, ltr501, apds9960) -> Android Sensors HAL. Options are the AOSP IIO sensors HAL (HAL 2.x "iio" subHAL), KonstaKANG's HAL as a pattern, or NXP's sensor HAL. Auto-brightness additionally needs `config_autoBrightnessLevels` overlays in the device tree's framework overlay.

### Gaps
- No source read on NXP i.MX DSI panel bring-up specifics (Samsung DSIM on 8M Mini/Plus) or on Android lights HAL details.
- No source confirming which touch controller / ALS / panel driver IC the salvaged device uses (covered by other researchers).
- Rockchip official "new screen adaptation" SDK doc not retrieved; findings rely on third-party wikis.

## 4. Audio: PDM mic arrays, far-field/AEC, class-D I2S amplifiers

### Takeaway
The lowest-risk far-field path is an XMOS XVF3800 front end (4 PDM mics in, AEC/beamforming/noise suppression on-chip, I2S or USB out) so Android just sees a clean mono/stereo capture device; TAS5805M/TAS5825M have working but partly out-of-tree Linux drivers, and MAX98357A is the zero-configuration fallback.

### Cited Findings
- Seeed reSpeaker XVF3800: 4 PDM MEMS mics, AEC, multi-beamforming, de-reverb, DoA, noise suppression, 60 dB AGC, far-field to ~5 m; dual mode USB (plug-and-play UAC for SBCs/PCs) and I2S (for Raspberry Pi, Jetson, XIAO) — [Seeed product page](https://www.seeed.cc/product/respeaker-mic-array-v2-0), [Seeed reSpeaker series](https://www.seeedstudio.com/respeaker-series)
- AEC is done on the XMOS DSP using the playback reference, so the device does not self-trigger over music — [creatingsmarthome.com review, 12 Apr 2026](https://www.creatingsmarthome.com/index.php/2026/04/12/home-assistant-diy-voice-assistant-with-seeed-studio-respeaker-xvf3800/)
- Community sentiment in 2026 rates the XVF3800 highly for HA voice satellites; ESPHome integration thread dated Oct 2025 — [Reddit (opinion)](https://www.reddit.com/r/homeassistant/comments/1wtql9c/unpopular_opinion_the_respeaker_xvf3800_is_the/), [HA community](https://community.home-assistant.io/t/respeaker-xmos-xvf3800-esphome-integration/927241)
- TAS5805M/TAS5825M: out-of-tree kernel module from Sonocotta supports both chips with I2C address auto-detect (0x2d TAS5805M, 0x4c TAS5825M), uses simple-audio-card on Pi; TAS5805M described as I2S-input class-D with built-in DSP, ~25 W/channel — [sonocotta driver repo](https://github.com/sonocotta/tas5805m-for-raspbian-paspberry-pi-zero), [Reddit](https://www.reddit.com/r/raspberry_pi/comments/1qm0lyb/tas5805m_tas5825m_based_raspberry_pi_hats/), [TI E2E thread](https://e2e.ti.com/support/audio-group/audio/f/audio-forum/1165952/tas5805m-linux-driver-for-tas58xx-family)
- MAX98357A works on Pi as a plain I2S DAC via existing overlays; kernel driver with SD-mode pin exists but is not built by default on Pi; pops/crackle on stream start/stop are a known issue — [Adafruit guide](https://learn.adafruit.com/adafruit-max98357-i2s-class-d-mono-amp/raspberry-pi-usage), [RPi StackExchange](https://raspberrypi.stackexchange.com/questions/74456/max98357-i2s-dac-support-kernel-drivers-and-noise-pop-crackle-on-playb)
- KonstaKANG Android supports GPIO I2S DACs (HiFiBerry DAC+ class) and USB microphones, so both a USB-UAC XVF3800 and an I2S amp have a supported path on CM5 — [KonstaKANG](https://konstakang.com/devices/rpi5/LineageOS23/)

### Inferences
- With a 14-15 V rail available, a TAS5805M/TAS5825M (PVDD up to ~26 V class) can drive the 4-8 ohm woofer directly from the barrel input and use its internal DSP for crossover/EQ to tweeters on the larger model (2.0 or 1.1/bi-amp modes). MAX98357A is 5 V/~3 W only — adequate for the 7-inch unit, weak for the 10-inch. (voltage/power figures are background, unverified)
- Reusing the device's own 2-3 PDM mics directly: on RK/i.MX, native PDM controllers would capture raw channels, but Android would then need software AEC/beamforming (AOSP's WebRTC-based AEC/NS effects in the audio HAL preprocessing are near-field grade). An XVF3800/XVF3610 chip on the carrier fed by the salvaged PDM mics plus an I2S reference from the amp feed is the route that mirrors the HA Voice PE / reSpeaker designs. XVF3800 expects a specific 4-mic geometry in its stock firmware; a 2-3 mic linear array from the salvaged device may need XVF3610 (2-mic) firmware instead. (background, unverified)
- On Pi CM4/CM5, lacking a dedicated PDM block exposed in Android, the pragmatic options are: XMOS over USB (UAC, zero driver work) or XMOS/PDM-to-I2S codec (e.g. ADAU7002 2-ch PDM->I2S, ADAU7118 8-ch PDM->TDM; both have mainline drivers — unverified) into the single I2S interface shared with the amp.
- (background, unverified) Mainline has `tas5805m`, `tas571x`, `tas2770`, `tas2781`, `max98357a`, `max98390` codec drivers; mainline `tas5805m` requires a DSP config firmware blob; TAS5825M support in mainline was not confirmed.

### Gaps
- No source read for XVF3610, for XMOS chip-level (not Seeed module) purchasing/firmware licensing at 1-10 units, or for HA Voice PE's reference design (XU316 + AIC3204), which is relevant prior art.
- No source confirming Android audio HAL configuration steps (audio_policy_configuration.xml, tinyalsa HAL card/device mapping) for I2S amp + external mic on any platform.
- Rockchip PDM / i.MX MICFIL Android capture support not verified with sources.

## 5. Power architecture from ~14 V barrel input

### Takeaway
No sources were gathered specifically on power design; the only cited constraints are module input ranges. Everything else here is unverified engineering background.

### Cited Findings
- Radxa CM5 input VCC_SYSIN is 3.6-5.2 V, 4 V/3 A recommended — [Radxa CM5 docs](https://docs.radxa.com/en/som/cm/cm5)
- antmicro's open CM4/CM5 baseboard and a community CM5 carrier with "automotive power" input (4-layer, KiCad, DSI-to-LVDS) exist as reference power front ends — [antmicro/cm4-baseboard](https://github.com/antmicro/cm4-baseboard), [community CM5 carrier (Facebook excerpt)](https://www.facebook.com/groups/altium.designer.vietnam/posts/1245501824236079/)

### Inferences
- (background, unverified) Suggested tree: barrel 14-15 V -> input protection (TVS, reverse-polarity PFET/ideal diode, fuse) -> (a) PVDD direct to class-D amp; (b) 5 V/4-5 A synchronous buck for the SoM (Pi CM5 wants 5 V at up to ~5 A peak budget; e.g. TPS54560/TPS568215/AP63357/MP2338-class parts stocked at JLCPCB); (c) 3.3 V buck or LDO from 5 V for touch/ALS/mics/XMOS (CM modules also export limited 3.3 V/1.8 V); (d) backlight LED boost/buck driver from 5 V or direct from 14 V depending on string voltage (7-10" panels typically 6-10 LEDs in series, ~18-30 V at 20-40 mA per string, or several parallel strings); (e) panel bias rails (AVDD/VGH/VGL or +/-5.5 V) if the salvaged panel's FPC expects them from the mainboard.
- Reusing the original 14-15 V adapter means the original board's backlight driver topology is a good hint for the required LED string voltage/current (to be measured on the salvaged board).

### Gaps
- Entire question lacks sourced findings: no CM4/CM5 power-budget numbers, no reference buck designs, no backlight driver app notes were retrieved.

## 6. Prior art: open smart displays, CM carriers for tablets, retrofits, Android HA dashboards/voice satellites

### Takeaway
There is solid prior art for drop-in replacement PCBs in Google smart *speakers* (ESP32-based MiciMike boards for Nest Mini / Home Mini) and for running Linux on the Lenovo Smart Display via postmarketOS, but no completed open project was found that replaces the mainboard of a smart *display* (Nest Hub, Echo Show) with a CM-class Android board; the Echo Show attempts found are discussion threads and teardowns.

### Cited Findings
- MiciMike: open-hardware drop-in PCB for Google Nest Mini (2nd gen) built on ESP32-S3 / ESP32-P4 (+ optional C6), running ESPHome, reusing the original enclosure, speaker, power supply, flex cable, mute switch and touch surface — [iMike78/nest-mini-drop-in-pcb](https://github.com/iMike78/nest-mini-drop-in-pcb); a Home Mini (Gen 1) variant reached pre-launch in 2026 — [Reddit](https://www.reddit.com/r/homeassistant/comments/1rwken9/dropin_pcb_replacement_for_the_google_home_mini/)
- HA community request thread for ESP32 + XMOS drop-in PCBs for Google Home/Nest speakers matching the HA Voice PE reference design, inspired by Onju Voice — [HA community](https://community.home-assistant.io/t/request-open-source-esp32-esphome-based-drop-in-pcb-replacement-for-google-nest-google-home-speakers-with-added-xmos-dsp-chip-to-match-official-home-assistant-voice-preview-edition-reference-hardware-specification-inspired-by-onju-voice/860001)
- Lenovo Smart Display (CD-18781Y) reflashed with postmarketOS via EDL and set up with Wyoming voice components (2024) — software replacement on stock hardware, not a board swap — [Michelle Avery, Medium](https://medium.com/@michelle-avery/replacing-google-nest-with-open-source-tools-and-a-bit-of-tinkering-f1352379b9f3)
- Echo Show 5 (2nd gen) teardown: bootloader locked/secure; author's stated future option is "scrap the existing CPU PCB and build a new board with a Raspberry Pi 4 compute module"; commenters ask how to connect speaker, mics, LCD and touch — no completed build reported — [Brian Dorey teardown](https://www.briandorey.com/post/echo-show-5-2nd-gen-smart-display-teardown)
- Discussion threads on putting a Pi inside Echo Show 5/8 (identifying the panel via replacement-part listings is suggested as the starting point) — [RPi forum](https://forums.raspberrypi.com/viewtopic.php?t=343789), [r/hardwarehacking](https://www.reddit.com/r/hardwarehacking/comments/qm2ko8/replacing_the_brains_of_an_echo_show_8_with/)
- Radxa-CM5-based handheld with a custom 5.5" MIPI panel driver board (Retro Lite CM5) shows a hobbyist RK3588S carrier + custom DSI panel is achievable — [StonedEdge/Retro-Lite-CM5](https://github.com/StonedEdge/Retro-Lite-CM5)
- CutiePi tablet (CM4-based) is listed among CM4 boards — [Jeff Geerling's Pi PCIe database](https://pipci.jeffgeerling.com/boards_cm)
- Android as HA dashboard/voice satellite: Ava Pro "turns any Android 5+ device into a voice-first Home Assistant kiosk" — [knoop7/Ava-Pro](https://github.com/knoop7/Ava-Pro); DashVoice, a Fully Kiosk alternative with on-device openWakeWord (Jan 2026) — [HA community](https://community.home-assistant.io/t/i-built-a-fully-kiosk-alternative-with-on-device-voice-control-meet-dashvoice/980581); browser-based "Voice Satellite" integration turning any tablet browser into a smart-display UI — [Reddit](https://www.reddit.com/r/homeassistant/comments/1rr7d4n/my_tablet_finally_feels_like_a_real_smart_display/)

### Inferences
- The software side of "Android smart display for Home Assistant" is mature and app-level (Ava, DashVoice, HA companion, Fully Kiosk, View Assist), needing only a working mic, speaker, touch and WebView — it does not require GMS.
- The absence of a finished Echo Show/Nest Hub board-swap project suggests the blocker is hardware reverse-engineering (panel init, FPC pinouts, backlight) rather than compute; this project would be novel.
- Because ESP32-P4 drop-in boards now exist for the speakers, an ESP32-P4 (MIPI DSI, LVGL/ESPHome) is the obvious non-Android alternative for the display models, but out of this brief's Android scope.

### Gaps
- Two searches aimed at Hackaday/CNX coverage of smart-display board swaps returned no results; no Hackaday or CNX article was located. This may be a search-tool artefact rather than proof of absence.
- No open-hardware Android smart-display reference design (e.g. from Seeed reTerminal, Waveshare CM4/CM5 panel carriers, HA "View Assist" hardware) was examined.
- Did not verify whether any Nest Hub-specific replacement board project exists as of Nov 2026.

## 7. Carrier board design cost/complexity

### Takeaway
A CM4/CM5-class carrier with DSI/CSI/USB is routinely done on 4 layers in KiCad with open templates, and JLCPCB offers impedance-controlled multilayer stackups at no extra charge, so bare-board cost is small; assembly and the connectors/parts dominate.

### Cited Findings
- Raspberry Pi's CM4IO KiCad design is open; Shawn Hymel's template and base-carrier repos reuse its symbols/footprints; mating connector is Hirose DF40C-100DS-0.4V (1.5 mm stack) or DF40HC(3.0)-100DS-0.4V (3.0 mm stack, clears underside components); differential pairs sized with KiCad's PCB calculator — [DigiKey/Shawn Hymel](https://www.digikey.com/en/maker/projects/creating-a-raspberry-pi-compute-module-4-cm4-carrier-board-in-kicad/7812da347e5e409aa28d59ea2aaea490), [rpi-cm4-carrier-template](https://github.com/ShawnHymel/rpi-cm4-carrier-template)
- antmicro cm4-baseboard: Apache-2.0, KiCad 9.x, separate display, CSI and NVMe/microSD sheets, rev 1.1.1 updated Jan 2026, electrically compatible with CM5 (noting CAM0/DSI0 -> USB3 change) — [antmicro/cm4-baseboard](https://github.com/antmicro/cm4-baseboard)
- A 4-layer CM5 carrier in KiCad with automotive power input, USB hub and DSI-to-LVDS bridge has been published by a community designer — [Facebook group post (excerpt)](https://www.facebook.com/groups/altium.designer.vietnam/posts/1245501824236079/)
- 17 public repos under the GitHub `compute-module-5` topic, including KiCad carriers such as xmyoot/rpi-cm5-edge-carrier; Seeed's open-source CM5 MINIMA carrier — [GitHub topic](https://github.com/topics/compute-module-5), [Seeed](https://www.seeed.cc/post/introducing-the-cm5-minima-r3-a-compact-open-source-powerhouse-for-raspberry-pi-cm5)
- JLCPCB: controlled impedance on 4-32 layer boards "at no extra charge"; 6-layer prototypes advertised from about $30-35 for 5 pcs (promotional pricing; excerpt) — [JLCPCB impedance](https://jlcpcb.com/impedance), [JLCPCB 6-layer](https://jlcpcb.com/6-layer-pcb), [JLCPCB capabilities](https://jlcpcb.com/capabilities/pcb-capabilities)
- CM5 datasheet gives MIPI routing guidance ("The MIPI signals should be routed ..." — excerpt truncated) — [CM5 datasheet (Mouser copy)](https://www.mouser.com/datasheet/2/635/cm5_datasheet-3537283.pdf)

### Inferences
- (background, unverified) Expected budget per 5-board run at JLCPCB: 4-layer impedance-controlled boards ~$10-40, SMT assembly setup + parts for a carrier with buck converters, amp, XMOS, FPC connectors roughly $100-300 total, with the 0.4 mm-pitch DF40 connectors and fine-pitch FPC connectors best left to the assembler (standard/"extended" parts fees apply). DSI at 1024x600/1280x800 runs a few hundred Mbit/s per lane; 100 ohm differential pairs, length-matched within a pair to ~0.1-0.2 mm, short runs and a solid reference plane are sufficient on 4 layers.
- The mechanically hardest parts are likely matching the salvaged device's FPC connectors (pitch/pin count/top-vs-bottom contact) and the board outline/mounting bosses, not the high-speed routing.
- A staged approach reduces risk: first prove panel/touch/audio on an off-the-shelf IO board with a small FPC adapter PCB, then commit to the full carrier.

### Gaps
- No verified low-volume assembly quotes; JLCPCB figures are marketing excerpts.
- No open KiCad carrier templates for Radxa CM4/CM5 or i.MX SoMs were located (Radxa publishes IO-board schematics; not verified here).
- CM5 datasheet routing rules (impedance, length-match tolerances) not extracted.
