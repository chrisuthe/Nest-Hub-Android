# Nest Hub Android

Research for a custom replacement mainboard that runs Android in a Google Nest Hub, reusing the stock display, touch, microphones, speakers, sensors and enclosure. The board is built around an off-the-shelf compute module. Modifying or bypassing the stock firmware is out of scope.

## Documents

One master document per model, plus one for everything shared. Update these in place; do not add per-session files.

| Document | Covers |
|---|---|
| [Nest Hub 1st gen](docs/Nest%20Hub%201st%20gen.md) | H1A, 7-inch, 2018 |
| [Nest Hub 2nd gen](docs/Nest%20Hub%202nd%20gen.md) | GUIK2, 7-inch with Soli radar, 2021 |
| [Nest Hub Max](docs/Nest%20Hub%20Max.md) | H2A, 10.1-inch with camera |
| [Compute modules and board design](docs/Compute%20modules%20and%20board%20design.md) | What the models share, open design decisions, compute module candidates, Android status, building blocks |

Each model document uses the same sections: identification, layout and connectors, display, display connector and pinout, backlight, touch, audio, sensors, buttons/LEDs/thermal, power, driver coverage, what a replacement board must provide, open questions, sources.

## Tracking

What we are building, and how far each item has got, is in [tracking/](tracking/README.md): a hardware requirements table, connector pinout sheets, a driver table, a decisions log and a bench log. All three models are seeded.

## Status

Desk research only. Nothing has been built, measured or tested on hardware. No connector pinout is public for any model.
