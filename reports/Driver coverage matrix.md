# Driver coverage matrix

Checked 2026-11-17. Android runs on the Linux kernel, so "Android driver" here means a Linux kernel driver, plus an Android HAL where one is needed.

No drivers have been built or tested. This records where source exists for each part we know is in the three Nest Hubs.

## How this was checked

- **Mainline**: read directly from `torvalds/linux` master (driver source and its compatible strings), or confirmed absent by a code search of that repository returning zero hits.
- **Vendor GPL**: Google's GPL kernel release for the 2nd gen, as mirrored at https://github.com/frederic/elaine-linux (Linux 4.9.244, last pushed 2022-06). Only directory listings and one file were inspected. It is a third-party mirror; the same code should be obtainable from Google's open-source release page for the product.
- **Other**: GitHub code search.

Status key: **Mainline** = in current upstream kernel. **Vendor** = GPL source exists outside mainline and needs porting to a modern kernel. **None** = no kernel driver found.

## Parts that stay in the enclosure

| Part | Models | Status | Driver / source | Work needed |
|---|---|---|---|---|
| JD9364 / JD9365 panel driver IC | all | Mainline | `drivers/gpu/drm/panel/panel-jadard-jd9365da-h3.c` | Add a panel entry with our timings and init sequence from the Fuchsia tables. The driver already carries a K&D panel (`kingdisplay,kd101ne3-40ti`) and Radxa displays as models to copy. |
| FocalTech FT5726 touch | Max | Mainline | `edt-ft5x06.c`, compatible `evervision,ev-ft5726` | Device-tree node. The stock firmware uploads touch firmware; mainline does not, so it relies on the controller keeping its flashed firmware. |
| FocalTech FT3x27 touch | 1st gen (BOE) | Unlisted | `edt-ft5x06.c` has `focaltech,ft3518` and a generic FocalTech path, but no FT3x27 entry | Test with an existing compatible; may need an ID added. Vendor `focaltech_touch` driver exists in the GPL tree as a fallback. |
| Goodix GT9293 touch | 1st gen (Innolux) | Unlisted | `goodix.c` has no `gt9293` compatible, but falls back to its GT9xx chip data for unknown IDs | Likely works with a neighbouring compatible such as `goodix,gt928`; untested. |
| Goodix GT6853 touch | 2nd gen | Vendor | Goodix GTX8 driver, GPL v2, compatible `goodix,gt6853`: `goodix_touch_gtx8` in the GPL tree; also seen in `Supernote-Ratta/kernel_a6x_a5x` | Port an out-of-tree driver to the target kernel. Mainline `goodix_berlin` only lists GT9916. Highest touch risk. |
| ams TCS3400 light sensor | all | Vendor | Mainline `tcs3472.c` does not support it. Google's GPL tree patches `drivers/iio/light/tcs3472.c` with `CONFIG_USE_TCS3400` | Forward-port Google's patch to the mainline TCS3472 driver. Zephyr also has a TCS3400 driver, useful as a register reference. |
| Sensirion SHTC3 | 2nd gen | Mainline | `drivers/hwmon/shtc1.c` | Device-tree node. |
| Infineon BGT60TR13C radar | 2nd gen | None (kernel) | Infineon `sensor-xensiv-bgt60trxx`, Apache-2.0, platform-independent C | Run in userspace over `spidev` with GPIO for IRQ and reset. Presence detection logic is ours to build. |
| Sony IMX227 camera | Max | None | Not in mainline. Old kernels (4.13–4.14) had it inside the Intel-only atomisp staging driver, which is not reusable as-is. Fuchsia's driver has the register tables. | Write a V4L2 sensor driver from the Fuchsia tables, plus ISP tuning on the chosen SoC. Highest risk overall. |
| PDM microphones | all | Platform | Generic `dmic` codec is mainline; the PDM receiver is the compute module's | Depends on module choice. |
| Buttons, mute switch | all | Mainline | `gpio-keys` | Device-tree node. |
| Status LEDs | all | Mainline | `pwm-leds` | Device-tree node. |
| NTC thermistors | all | Mainline | `ntc_thermistor` via an ADC | Needs an ADC on the new board. |

## Parts on the stock mainboard (lost; listed to guide part selection)

| Part | Stock use | Status | Driver |
|---|---|---|---|
| TI LP8556 backlight | all | Mainline | `drivers/video/backlight/lp855x_bl.c` |
| TI TPS65132 bias supply | Max (inferred identity) | Mainline | `drivers/regulator/tps65132-regulator.c` |
| TI TAS2770 amp | 1st gen | Mainline | `sound/soc/codecs/tas2770.c` |
| TI TAS5805M amp | 2nd gen | Mainline | `sound/soc/codecs/tas5805m.c` |
| TI TAS5720 amp | Max | Mainline | `sound/soc/codecs/tas5720.c` |
| Maxim MAX98357A amp | candidate | Mainline | `sound/soc/codecs/max98357a.c` |
| TI TAS5825M amp | candidate | Vendor | Not in mainline; a `tas5825m.c` is in Google's GPL tree |

Reusing the LP8556, TPS65132 and the stock amplifier part numbers on the new board keeps every one of them on a mainline driver, with Google's register values already published.

## Android layer above the kernel

Not verified; from general knowledge of AOSP.

- **Display, touch, buttons**: no HAL work. DRM and evdev are picked up directly.
- **Backlight**: needs the lights HAL pointed at the backlight sysfs node.
- **Light and humidity sensors**: need a sensors HAL that reads IIO/hwmon devices. The community Raspberry Pi build already has an I2C sensor HAL to extend.
- **Audio**: needs an audio HAL configuration (tinyalsa/mixer paths) for the amplifier and PDM capture.
- **Radar**: no Android framework concept; expose it through a custom service or app.
- **Camera**: needs a camera HAL on top of V4L2, and depends on the SoC's ISP support.

## Also found: Google's GPL kernel tree for the 2nd gen

The mirrored tree contains more than drivers. Device trees named `elaine-*.dts` and `elaine-sm1-panel.dtsi` are present. These should hold the stock panel power sequencing and pin assignments on the SoC side, which the pinout work still lacks. **Update:** these have now been read, and the same tree also holds the 1st gen (`estelle`) and Max (`newman`) device trees and a vendor FocalTech driver with an FT3427 config. See `Stock kernel device tree findings.md`.

## Gaps and next steps

1. Done: device trees read for all three models (see `Stock kernel device tree findings.md`).
2. Done: the 1st gen and Max are in the same tree.
3. Decide the compute module, since kernel version decides how much porting the vendor drivers need.
4. Bench-test the unlisted touch controllers (FT3x27, GT9293) against the mainline drivers before porting anything.
