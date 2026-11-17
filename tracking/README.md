# Tracking

The documents in `docs/` record what the stock hardware is. The files here record what we are building and how far each item has got.

| File | Tracks |
|---|---|
| `hardware.csv` | What the replacement board must provide, one row per requirement |
| `pinouts/<model>.csv` | Every signal on every connector we mate with, one row per signal |
| `drivers.csv` | What the OS build must contain, one row per device |
| `decisions.md` | Design decisions, open and made |
| `bench-log.md` | Measurements taken on real hardware |

Only the 1st gen is seeded so far.

## IDs

Each peripheral has one short ID (`DISP`, `BL`, `TOUCH`, ...) used in all three tables. Every ID in `hardware.csv` has at least one row in `drivers.csv` (except `MECH`, which has no software side), and every pinout row names the hardware ID it belongs to. A new peripheral gets a row in both tables at the same time.

## Evidence

| Value | Meaning |
|---|---|
| Confirmed | From the exact part's datasheet, a photo, or Google's own board code |
| Sibling | From a related part or another model. Not safe to wire from |
| Inferred | Reasoning from other facts |
| Measured | Taken on a real unit and recorded in `bench-log.md` |

## Status

Hardware and pinout rows:

`Open` → `Measured` → `In schematic` → `Built` → `Verified`

Driver rows:

`Identified` → `Source located` → `Builds` → `Probes` → `Works in Linux` → `Works in Android`

## Rules

1. A connector goes into a schematic only when every one of its pinout rows is `Measured`.
2. Evidence becomes `Measured` only with a matching bench log entry.
3. A status moves forward only with something to point at: a bench log entry, a schematic revision, a commit, a boot log.
4. A row that is waiting on a decision names it in `blocked_by` (`D2`, `D4`, ...).
5. When a measurement contradicts `docs/`, update the model document as well.

## Notes on the seeded data

- The `kconfig_unverified` column in `drivers.csv` holds mainline kernel option names written from memory. Confirm each against the kernel that the chosen module uses.
- Pin numbers are blank throughout: no pinout is known yet.
- The display flex signal list assumes touch shares the display flex and that the backlight has a common anode with three cathodes. Both are inferred.
