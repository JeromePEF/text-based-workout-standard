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
/log bench press 4x8 185lb
log pullups 12
```

## What's Covered

| Domain | What You Can Log |
|--------|-----------------|
| **Workouts** | Exercises, sets, reps, weight, duration, distance, cardio, machine settings |
| **Nutrition** | Meals, individual foods, calories, macros (protein/carbs/fat), meal labels |
| **Water** | Daily water intake in oz or ml |
| **Body Metrics** | Weight, measurements, blood work |

## Specification

See **[SPEC.md](SPEC.md)** for the formal grammar, parsing rules, unit handling, and edge cases.

## Implementations

- **[TheTrackerApp](https://thetrackerapp.io)** — Full implementation powering real-time coaching
- *Add yours here — open a PR!*

## License

MIT — free for any app, service, or tool to implement.
