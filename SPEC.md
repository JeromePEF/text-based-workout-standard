# TBWS Specification v1.0

## Table of Contents

1. [Design Principles](#1-design-principles)
2. [Message Structure](#2-message-structure)
3. [Workout Logging](#3-workout-logging)
4. [Nutrition Logging](#4-nutrition-logging)
5. [Water Logging](#5-water-logging)
6. [Body Metrics](#6-body-metrics)
7. [Query Commands](#7-query-commands)
8. [Compound Messages](#8-compound-messages)
9. [Grammar Reference](#9-grammar-reference)
10. [Parser Behaviors](#10-parser-behaviors)

---

## 1. Design Principles

1. **Slash-first, natural-language-tolerant.** A message starting with `/` is a deterministic command. All other text is natural language that parsers attempt to resolve.
2. **Unit-carrying values.** Every measurement carries its unit in the message (e.g., `185lb`, `5k`, `24oz`, `30m`). If omitted, parsers use the user's configured default.
3. **Idempotent logging.** Duplicate detection prevents double-counting within a configurable window (default: 5 minutes).
4. **Extensible metadata.** Unknown key:value pairs are preserved as opaque metadata, never discarded.
5. **Lossless round-trip.** A parsed and re-serialized message must contain all original data.

---

## 2. Message Structure

Every TBWS message is a single line of text (newlines delimit messages). A message contains exactly one **intent** — or multiple intents separated by compound delimiters (see §8).

```
INTENT ::= COMMAND | NATURAL_LANGUAGE
COMMAND ::= "/" COMMAND_NAME [ARGUMENTS]
NATURAL_LANGUAGE ::= free-form text that parsers resolve to an intent
```

### Command Resolution Order

1. Exact slash-command match → dispatched immediately
2. Natural language → walks the resolver chain:
   a. Fast regex patterns (deterministic)
   b. Structured query patterns
   c. ML/LLM fallback

---

## 3. Workout Logging

### 3.1 Slash Command

```
/log EXERCISE_NAME MEASUREMENT [METADATA...]
```

### 3.2 Measurement Formats

| Format | Example | Meaning |
|--------|---------|---------|
| Bare count | `50` | 50 reps |
| Sets × reps | `3x12`, `3,12` | 3 sets of 12 reps |
| Sets × reps × weight | `4x8x225`, `4,8,225` | 4 sets × 8 reps × 225 (default lb) |
| Sets × reps × weight + unit | `4x8x100kg`, `4,8,100 kg` | Explicit kg |
| Duration | `45m`, `90s`, `1h15m` | Time-based exercise |
| Distance | `5k`, `10mi`, `400m` | Cardio distance |
| Distance + duration | `5k 22:30`, `10mi 1:15:00` | Distance with pace |

### 3.3 Exercise Name

Exercise names are normalized to lowercase, trimmed, and matched against a catalog. Unknown names are accepted and stored verbatim.

**Variants & sides:**
- Prefix modifiers: `left`, `right`, `decline`, `incline`, `assisted`, `weighted`, `negative`, `archer`
- Examples: `left leg lunges`, `decline pushups`, `weighted pullups`, `negative pullups`

**Cardio / machine exercises** accept extended metadata:
```
/log treadmill 20m 3.2mph 8 incline brand X
/log stairmaster 15m level 10
/log peloton 30m 245kj output
/log elliptical 45m resistance 12
```

### 3.4 Shorthand Commands

Per-exercise shortcuts for common exercises:
```
/pushups 50        →  /log pushups 50
/pullups 12        →  /log pullups 12
/dpushups 25       →  /log decline pushups 25
/squats 3x10 185   →  /log squats 3x10x185
/bench 4x8 225     →  /log bench press 4x8x225
/deadlift 5x5 315  →  /log deadlift 5x5x315
/run 5k 22:30      →  /log run 5k 22:30
```

### 3.5 Increment Syntax

```
+N                  → add N to last logged exercise
+N EXERCISE_NAME    → add N to named exercise
```

Examples: `+5`, `+10 pushups`, `+13 rows`

### 3.6 Natural Language Examples

All of these must resolve to the same workout log:

```
Did 20 pushups
pushups 50
20 archer pushups
3x10 pushups
did 5 sets of 12 bicep curls
4x8 tricep extensions
bench press 4,8,225
benched 225 for 5
3x10 squats at 185lb
plank for 60 seconds
Ran 3 miles in 25 mins
ran 5k
half marathon in 2:10
sprinted 400 meters
treadmill 20m 3.2mph incline 8
spin class 45 minutes
```

---

## 4. Nutrition Logging

### 4.1 Slash Command

```
/food DESCRIPTION CALORIES [MACROS...]
```

### 4.2 Formats

**Minimal:**
```
/food "Chicken Breast" 280
```

**Full macros:**
```
/food "Chicken Breast" 280 45p 6f 0c
```

**Macro order is flexible:**
```
/food "Salmon Fillet" 367 23p 12f 0c
/food "White Rice" 205 4p 0f 45c
```

### 4.3 Macro Units

| Suffix | Meaning | Notes |
|--------|---------|-------|
| `p` | Protein (grams) | |
| `c` | Carbs (grams) | |
| `f` | Fat (grams) | |
| `fib` | Fiber (grams) | Optional |
| `sug` | Sugar (grams) | Optional |
| `chol` | Cholesterol (mg) | Optional |
| `na` | Sodium (mg) | Optional |

### 4.4 Meal Labels

Prefix a food entry with a meal label to assign it to a meal slot:
```
/food Breakfast: "Eggs & Toast" 350 22p 14f 28c
/food Lunch: "Grilled Salmon Salad" 520 38p 28f 18c
```

### 4.5 Natural Language Examples

```
I had a Celsius
ate grilled salmon
for breakfast i had eggs and avocado
had a protein shake - 180 cal 30p
ate 8oz chicken breast with rice
drank a vanilla whey shake
```

### 4.6 Multi-Item Meals

Items separated by `and`, `,`, `plus`, `with`, or newlines:
```
/food Breakfast: eggs 140 12p 10f 2c and toast 120 4p 1f 24c and coffee 5
```

---

## 5. Water Logging

### 5.1 Slash Command

```
/water AMOUNT [UNIT]
```

### 5.2 Units

| Unit | Example |
|------|---------|
| `oz` (default) | `/water 24` → 24 oz |
| `ml` | `/water 500 ml` |
| `l` | `/water 1.5 l` |
| `cups` | `/water 3 cups` |

### 5.3 Natural Language Examples

```
Drank 32oz water
I drank 80 oz of water
32oz water
500ml water
had 3 cups of water
```

---

## 6. Body Metrics

### 6.1 Slash Command

```
/body METRIC VALUE [UNIT]
```

### 6.2 Supported Metrics

| Metric | Unit | Example |
|--------|------|---------|
| `weight` | lb (default), kg | `/body weight 181` |
| `neck` | in | `/body neck 15.5` |
| `shoulders` | in | `/body shoulders 48` |
| `chest` | in | `/body chest 42` |
| `left bicep` | in | `/body left bicep 15` |
| `right bicep` | in | `/body right bicep 15` |
| `waist` | in | `/body waist 32` |
| `hips` | in | `/body hips 40` |
| `left thigh` | in | `/body left thigh 24` |
| `right thigh` | in | `/body right thigh 24` |
| `left calf` | in | `/body left calf 15` |
| `right calf` | in | `/body right calf 15` |
| `body fat` | % | `/body body fat 15` |
| `height` | ft'in", cm | `/body height 5'11"` |

### 6.3 Natural Language Examples

```
weight 181
neck 15.5 in
right bicep 16 in
thigh is 22
body fat 15 percent
```

---

## 7. Query Commands

Queries retrieve previously logged data. These are read-only.

### 7.1 Period Summaries

```
/today              → all logs for today
/week               → weekly summary
/month              → monthly summary
/daily              → day-by-day breakdown
/report EXERCISE    → full history for an exercise
```

### 7.2 Workout Queries

```
/workout list       → all logged exercises
/plan               → current workout plan/split
/suggest            → next-goal suggestion
/goals              → list active goals
```

### 7.3 Nutrition Queries

```
/food today         → full-day food list with totals
/blood latest       → latest blood work results
```

### 7.4 Natural Language Questions

```
how many pushups did i do this week?
what's my bench press PR?
when did i last do legs?
how many calories today?
hows my week looking?
am I on track for my protein goal?
what was my weight last month?
```

---

## 8. Compound Messages

A single message can contain multiple intents. Parsers split on these delimiters:

**Explicit separators:** newline, `;`

**Implicit separators (natural language):** `and`, `plus`, `with`, `also`, `then`

### Examples

```
/food eggs 140 12p and coffee 5 and /water 16 and /log pushups 50
```

This resolves to three intents:
1. Nutrition: eggs (140 cal, 12g protein) + coffee (5 cal)
2. Water: 16 oz
3. Workout: 50 pushups

```
Weight: 140 Height: 5'7" Breakfast: eggs avocado
```

Resolves to:
1. Body metrics: weight 140, height 5'7"
2. Nutrition: eggs, avocado (breakfast)

### Parsing Rules for Compound Messages

1. Slash commands delimit their own boundaries
2. Body-measure-shaped pairs (`metric: value`) are extracted first
3. Remaining text is scanned for workout patterns, nutrition patterns, water patterns
4. Unresolved fragments trigger a clarification

---

## 9. Grammar Reference

### 9.1 Formal ABNF

```abnf
; Top level
message         = intent *(compound-sep intent)
intent          = slash-command / nl-workout / nl-nutrition / nl-water / nl-body / nl-query
compound-sep    = "." / "\n" / ";" / " and " / " plus " / " with " / " also " / " then "

; Slash commands
slash-command   = "/" command-name [SP args]
command-name    = ALPHA *(ALPHA / DIGIT)
args            = *(any-char)   ; parsed per-command

; Workout
workout-log     = exercise-name SP measurement [SP metadata]
exercise-name   = 1*(ALPHA / DIGIT / "-" / SP)   ; normalized to lowercase
measurement     = count / sets-reps / sets-reps-weight / duration / distance
count           = NUMBER
sets-reps       = NUMBER ("x" / ",") NUMBER
sets-reps-wt    = NUMBER ("x" / ",") NUMBER ("x" / ",") NUMBER [weight-unit]
weight-unit     = "lb" / "lbs" / "kg" / "kgs"
duration        = NUMBER time-unit *(NUMBER time-unit)
time-unit       = "s" / "sec" / "secs" / "m" / "min" / "mins" / "h" / "hr" / "hrs"
distance        = NUMBER distance-unit
distance-unit   = "m" / "meters" / "km" / "k" / "mi" / "mile" / "miles"

; Nutrition
nutrition-log   = food-desc SP calories *(SP macro)
food-desc       = DQUOTE *(any-char) DQUOTE / 1*(any-char)   ; quoted or bare
calories        = NUMBER
macro           = NUMBER macro-unit
macro-unit      = "p" / "c" / "f" / "fib" / "sug" / "chol" / "na"

; Water
water-log       = NUMBER [SP water-unit]
water-unit      = "oz" / "ml" / "l" / "liters" / "cups" / "cup"

; Body
body-log        = metric-name SP NUMBER [SP body-unit]
metric-name     = "weight" / "neck" / "shoulders" / "chest" / "waist" / "hips" /
                  ("left" / "right") SP ("bicep" / "thigh" / "calf") /
                  "body" SP "fat" / "height"
body-unit       = "lb" / "lbs" / "kg" / "in" / "cm" / "%"

; Primitives
NUMBER          = DIGIT *DIGIT ["." *DIGIT]
ALPHA           = %x41-5A / %x61-7A   ; A-Z / a-z
DIGIT           = %x30-39             ; 0-9
SP              = %x20                ; space
DQUOTE          = %x22                ; "
```

### 9.2 Token Priority

When multiple patterns could match, parsers apply these rules:

1. **Quoted strings** (`"Chicken Breast"`) bind tighter than unquoted
2. **Weighted patterns** (sets×reps×weight) consume their span before simpler patterns
3. **Compound measurements** (distance + duration like `5k 22:30`) are treated as one unit
4. **Macro suffixes** (`p`, `c`, `f`) only bind when immediately adjacent to a number

---

## 10. Parser Behaviors

### 10.1 Unit Normalization

| Input | Normalized | Notes |
|-------|-----------|-------|
| `185lb`, `185 lbs`, `185lb.` | `185 lb` | |
| `100kg`, `100 kg`, `100kgs` | `100 kg` | |
| `5k`, `5km`, `5 kilometers` | `5 km` | |
| `10mi`, `10 miles`, `10 mile` | `10 mi` | |
| `24oz`, `24 oz`, `24 ounces` | `24 oz` | |
| `500ml`, `500 mL`, `500 ml` | `500 ml` | |
| `30m`, `30 min`, `30 mins`, `30 minutes` | `30 min` | |
| `1h30m`, `1 hr 30 min` | `90 min` | Normalized to minutes |

### 10.2 Default Units

If a parser's target user has configured defaults, those apply when units are omitted:

| Context | Default | Override |
|---------|---------|----------|
| Weight (exercise) | `lb` | `kg` suffix |
| Weight (body) | `lb` | `kg` suffix |
| Water | `oz` | `ml` / `l` suffix |
| Distance | `mi` | `km` / `m` suffix |
| Body measurements | `in` | `cm` suffix |

### 10.3 Number Parsing

- Integers: `50`, `100`, `315`
- Decimals: `15.5`, `2.5`
- Fractions: `1/2`, `3/4` → converted to decimals
- Time (duration display): `22:30` → 22 min 30 sec
- Time (hour:min:sec): `1:15:00` → 1 hr 15 min

### 10.4 Duplicate Detection

Implementations SHOULD detect and reject duplicate logs:
- Same exercise + same measurement + same user, received within 5 minutes
- Same food entry (description + calories + user) within 10 minutes
- Same water amount within 5 minutes

### 10.5 Error Handling

| Condition | Behavior |
|-----------|----------|
| Unparseable command | Return helpful error with closest-match suggestion |
| Implausible value (e.g., weight 5000 lb) | Challenge before logging |
| Missing required field | Ask targeted clarification question |
| Unknown exercise name | Accept and log verbatim |
| Unknown food | Accept calories; flag for macro enrichment |

### 10.6 Clarification Protocol

When a parser detects intent but has incomplete data:

1. Ask exactly ONE targeted question
2. Hold a pending state (default TTL: 10 minutes)
3. The pending state is released (not trapped) by: any slash command, a new log, a question, or a body-measurement-shaped message
4. Short unrelated messages also release the state

---

## Appendix A: Command Quick Reference

| Command | Example |
|---------|---------|
| `/log` | `/log bench press 4x8 185lb` |
| `/pushups` | `/pushups 50` |
| `/pullups` | `/pullups 12` |
| `/squats` | `/squats 3x10 225` |
| `/bench` | `/bench 4x8 185` |
| `/deadlift` | `/deadlift 5x5 315` |
| `/run` | `/run 5k 22:30` |
| `/food` | `/food "Chicken Breast" 280 45p 6f` |
| `/water` | `/water 24` |
| `/body` | `/body weight 181` |
| `/today` | `/today` |
| `/week` | `/week` |
| `/report` | `/report bench press` |
| `/undo` | `/undo` |
| `/goals` | `/goals` |
| `/suggest` | `/suggest` |
| `+N` | `+10` or `+5 pushups` |

## Appendix B: Example Messages

See the [`examples/`](examples/) directory for annotated example messages covering edge cases, compound intents, and localization.
