# Text-Based Workout Standard (TBWS)

**A simple, human-readable, machine-parseable format for logging workouts, nutrition, and water intake — from any device that can send text.**

## Why?

Fitness logging shouldn't require a smartphone app, a proprietary format, or tapping through dropdown menus. Text is universal — it works over SMS, iMessage, Telegram, WhatsApp, Slack, a terminal, or any messaging platform. This standard defines a format that is:

- **Easy for humans to type** — reads like natural shorthand you'd text a friend
- **Unambiguous for machines to parse** — structured enough for reliable data extraction
- **Platform-agnostic** — works on a flip phone, a smartwatch, or a web app
- **Extensible** — add custom metrics without breaking parsers

## Quick Examples

```
/water 24
/food chicken breast 280 45p 6f 0c
/pullups 3x10@45
/knee_pushups 20
/log incline dumbbell curls 3x10 at 25
/mountain_pose 60s
/run 5k 22:30
```

Same workout, written four ways — separators are interchangeable:

```
/knee_pushups 20
/knee-pushups 20
/knee pushups 20
/kneepushups 20
```

The bare command and `/log` do the same thing:

```
/pullups 3x10@45
/log pullups 3x10@45
```

English and Sanskrit yoga names share one history:

```
/mountain_pose 60s
/tadasana 60s
```

## What's Covered

| Domain | What You Can Log |
|--------|-----------------|
| **Workouts** | Every exercise is its own command: calisthenics (incl. weighted — sets×reps@weight on any bodyweight move), strength with modifier-composed names (implement × angle × laterality × grip — the full "curl codex"), yoga (English + Sanskrit), pilates (mat + reformer), cardio & machines, fitness exams (FitnessGram, Army AFT, Navy PRT, Marine PFT, Cooper), pro combines (NFL/NBA/WNBA/MLB/NHL/soccer), track & field, distance running |
| **Nutrition** | Meals, individual foods, calories, macros (protein/carbs/fat), meal labels |
| **Water** | Daily water intake in oz or ml |
| **Body Metrics** | Weight, measurements, blood work |

## Naming Guarantees

- `_`, `-`, space, and joined forms are the **same command**: `/knee_pushups` ≡ `/knee-pushups` ≡ `/knee pushups` ≡ `/kneepushups`
- Bare commands and `/log` are **equivalent**: `/pullups 3x10@45` ≡ `/log pullups 3x10@45`
- Short forms coalesce: `bench` ≡ `benchpress` ≡ `bench press` — one history bucket
- Modifier-composed names ("incline dumbbell curl", "single arm cable curl") are accepted **as-is**, each with its own history

## Specification

See **[SPEC.md](SPEC.md)** for the formal grammar, parsing rules, unit handling, and edge cases.

## Implementations

- **[TheTrackerApp](https://thetrackerapp.io)** — Full implementation powering real-time coaching
- *Add yours here — open a PR!*

## License

MIT — free for any app, service, or tool to implement.
