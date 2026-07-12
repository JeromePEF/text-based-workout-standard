# Examples — Annotated

Each example shows the raw message and a breakdown of what it resolves to.

---

## Workout Examples

### Basic logging
```
/log pushups 50
```
→ Exercise: pushups, 50 reps, bodyweight

### Sets × reps
```
/log pullups 3x12
```
→ Exercise: pullups, 3 sets × 12 reps, bodyweight

### Sets × reps × weight
```
/log bench press 4x8x225
```
→ Exercise: bench press, 4 sets × 8 reps, 225 lb

### With explicit kg
```
/log squat 5x5x100kg
```
→ Exercise: squat, 5 sets × 5 reps, 100 kg

### Timed exercise
```
/log plank 90s
```
→ Exercise: plank, 90 seconds

### Cardio with distance
```
/log run 5k 22:30
```
→ Exercise: run, 5 km, 22 min 30 sec

### Machine with metadata
```
/log treadmill 20m 3.2mph 8 incline
```
→ Exercise: treadmill, 20 minutes, 3.2 mph, incline 8

---

## Nutrition Examples

### Minimal
```
/food "Chicken Breast" 280
```
→ Food: Chicken Breast, 280 cal, macros unknown

### Full macros
```
/food "Grilled Salmon" 367 23p 12f 0c
```
→ Food: Grilled Salmon, 367 cal, 23g protein, 12g fat, 0g carbs

### With meal label
```
/food Breakfast: "Eggs & Toast" 350 22p 14f 28c
```
→ Food: Eggs & Toast, 350 cal, meal: breakfast, 22p/14f/28c

### Meal with extended macros
```
/food "Protein Bar" 210 20p 7f 25c 9fib 5sug
```
→ 210 cal, 20p/7f/25c + 9g fiber, 5g sugar

---

## Water Examples

```
/water 24
```
→ 24 oz water (default unit)

```
/water 500 ml
```
→ 500 ml water

```
/water 1.5 l
```
→ 1.5 liters water

---

## Body Metric Examples

```
/body weight 181
```
→ Weight: 181 lb

```
/body weight 82.5 kg
```
→ Weight: 82.5 kg

```
/body left bicep 15.5
```
→ Left bicep: 15.5 in

```
/body body fat 14
```
→ Body fat: 14%

---

## Compound Messages

### Slash commands in one message
```
/food "Eggs" 140 12p and /water 16 and /log pushups 50
```
→ 3 intents:
  1. Nutrition: Eggs, 140 cal, 12g protein
  2. Water: 16 oz
  3. Workout: pushups, 50 reps

### Natural language compound
```
Weight: 140 Height: 5'7" Breakfast: eggs avocado
```
→ 2 intents:
  1. Body: weight 140 lb, height 5'7"
  2. Nutrition: eggs + avocado (breakfast)

### Mixed workout + nutrition
```
ate a waffle and drank a celsius and did 50 pushups and 20oz water
```
→ 4 intents:
  1. Nutrition: waffle
  2. Nutrition: celsius
  3. Workout: pushups, 50 reps
  4. Water: 20 oz

### Post-workout log
```
bench 4x8x225, incline dumbbell press 3x10x80s, cable flyes 3x15x50 then 20m treadmill 3.5mph
```
→ 4 intents:
  1. Workout: bench press, 4×8, 225 lb
  2. Workout: incline dumbbell press, 3×10, 80 lb dumbbells
  3. Workout: cable flyes, 3×15, 50 lb
  4. Workout: treadmill, 20 min, 3.5 mph

---

## Naming Equivalence (v1.1)

### Separator equivalence — all the same workout
```
/knee_pushups 50
/knee-pushups 50
/knee pushups 50
/kneepushups 50
```
→ Exercise: KNEE PUSHUPS, 50 reps (one history bucket)

### Bare command ≡ /log
```
/pullups 3x10@45
/log pullups 3x10@45
```
→ Both: pullups, 3 sets × 10 reps @ 45 lb added weight — identical result, one report

### Weighted calisthenics on bare commands
```
/pullups 3x10@45        → 3×10 with 45 lb added
/dips 5 sets of 12      → 5×12 bodyweight
/squats 3,10,135        → 3×10 @ 135 lb
/pushups 10 wv 40       → 10 reps in a 40 lb vest
/pushups -5             → subtract 5 from today's total
```

### Short-form coalescing
```
/log bench 3x10 at 135
/log benchpress 2x8 at 155
/report bench press
```
→ Both sets appear under one BENCH PRESS report.

---

## Modifier-Composed Names (v1.1)

The four ingredient categories (implement × angle × laterality × grip) combine freely in front of any base movement:

```
/log incline dumbbell curls 3x10 at 25
```
→ Exercise: INCLINE DUMBBELL CURL (its own history bucket), 3×10 @ 25 lb

```
/log single arm cable curl 3x12 at 30
```
→ Exercise: SINGLE ARM CABLE CURL, 3×12 @ 30 lb

```
/log reverse ez bar curls 3x10 at 40
```
→ Exercise: REVERSE EZ BAR CURL, 3×10 @ 40 lb

```
/log decline bench press 3x8 at 185
```
→ Exercise: DECLINE BENCH PRESS, 3×8 @ 185 lb

No confirmation prompt, no renaming — the nuance the athlete typed is the exercise that gets tracked.

---

## Domain Coverage (v1.1)

### Yoga (English ≡ Sanskrit)
```
/mountain_pose 60s        ≡ /tadasana 60s
/warrior_one 30s          ≡ /warrior-1 30s
/downward_facing_dog 45s  ≡ /adho_mukha_svanasana 45s
/sun_salutation_a 5       → 5 rounds (counted, not timed)
```

### Duration context — bare number means seconds
```
/plank 90        ≡ /plank 90s
/crow_pose 15    ≡ /crow_pose 15s
```

### Fitness exams
```
/curlups 45                 → FitnessGram curl-ups
/pacer 55                   → PACER laps
/handreleasepushups 45      → Army AFT hand-release pushups
/sprintdragcarry 1          → Army AFT SDC round
/swim500 1                  → Navy PRT 500-yard swim
/sprint300 1                → Cooper test 300 m sprint
```

### Pro combines & track
```
/log 40yard 4.4             → NFL 40-yard dash
/log laneagility 11.2       → NBA lane agility
/log exitvelocity 95        → MLB exit velocity
/log sprint100 12.5         → 100 m dash
/log halfmarathon 1:45:00   → half marathon
```
