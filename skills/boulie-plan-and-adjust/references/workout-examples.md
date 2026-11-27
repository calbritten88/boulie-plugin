# Worked buildWorkout examples

Every example below compiles: a test in Boulie's harness sends each one through the same builder `buildWorkout` uses. Copy the shape, then change the numbers. The rules behind them are in the plan-and-adjust skill, section 4.

## 1. Simple intervals: 5 × 4 min at 110% with 3 min easy

One section. Five identical sets, one identical recovery between each, and it ends on `work`.

```json
{
  "sport": "ride",
  "intent": "vo2max",
  "targetBasis": "power_pct",
  "segments": [
    { "role": "warmup", "duration": { "unit": "seconds", "value": 900 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 240 }, "target": [108, 112] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 180 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 240 }, "target": [108, 112] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 180 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 240 }, "target": [108, 112] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 180 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 240 }, "target": [108, 112] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 180 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 240 }, "target": [108, 112] },
    { "role": "cooldown", "duration": { "unit": "seconds", "value": 600 } }
  ]
}
```

## 2. Nested over-unders: 3 sets of 4 × (2 min at 105%, 1 min at 90%), 5 min easy between sets

Each set is the over-under cycle written out four times, all as `work`. Boulie detects the cycle and reports `3x 4x (...)`.

```json
{
  "sport": "ride",
  "intent": "threshold",
  "targetBasis": "power_pct",
  "segments": [
    { "role": "warmup", "duration": { "unit": "seconds", "value": 900 }, "target": [55, 70] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 300 }, "target": [50, 60] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 300 }, "target": [50, 60] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "work", "duration": { "unit": "seconds", "value": 120 }, "target": [105, 105] },
    { "role": "work", "duration": { "unit": "seconds", "value": 60 }, "target": [90, 90] },
    { "role": "cooldown", "duration": { "unit": "seconds", "value": 600 }, "target": [50, 60] }
  ]
}
```

## 3. Two different blocks: 2 × 10 min at threshold, then 4 × 30 s sprints

The blocks differ, so a `steady` segment separates them. The easy spin after the last sprint goes into the cool-down, not a final recovery.

```json
{
  "sport": "ride",
  "intent": "threshold",
  "targetBasis": "power_pct",
  "segments": [
    { "role": "warmup", "duration": { "unit": "seconds", "value": 900 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 600 }, "target": [95, 100] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 300 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 600 }, "target": [95, 100] },
    { "role": "steady", "duration": { "unit": "seconds", "value": 300 }, "target": [50, 60] },
    { "role": "work", "duration": { "unit": "seconds", "value": 30 }, "target": [120, 130] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 30 }, "target": [120, 130] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 30 }, "target": [120, 130] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "seconds", "value": 30 }, "target": [120, 130] },
    { "role": "cooldown", "duration": { "unit": "seconds", "value": 690 } }
  ]
}
```

## 4. Run pyramid in metres: 400, 800, 1200, 800, 400 m at 105% of threshold speed, 200 m jog after each

Each rep is a different length, so each is its own section, and the jogs between them are `steady` in seconds (200 m at an easy pace is about 66 s). The last jog goes into the cool-down.

```json
{
  "sport": "run",
  "intent": "intervals",
  "targetBasis": "pace_pct",
  "segments": [
    { "role": "warmup", "duration": { "unit": "seconds", "value": 600 } },
    { "role": "work", "duration": { "unit": "meters", "value": 400 }, "target": [105, 105] },
    { "role": "steady", "duration": { "unit": "seconds", "value": 66 }, "target": [70, 80] },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [105, 105] },
    { "role": "steady", "duration": { "unit": "seconds", "value": 66 }, "target": [70, 80] },
    { "role": "work", "duration": { "unit": "meters", "value": 1200 }, "target": [105, 105] },
    { "role": "steady", "duration": { "unit": "seconds", "value": 66 }, "target": [70, 80] },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [105, 105] },
    { "role": "steady", "duration": { "unit": "seconds", "value": 66 }, "target": [70, 80] },
    { "role": "work", "duration": { "unit": "meters", "value": 400 }, "target": [105, 105] },
    { "role": "cooldown", "duration": { "unit": "seconds", "value": 546 } }
  ]
}
```

## 5. Equal track reps: 6 × 800 m at 3:20/km with 90 s jog

Identical reps can share one section: distance `work`, timed `recovery`. `pace_abs` is seconds per km.

```json
{
  "sport": "run",
  "intent": "intervals",
  "targetBasis": "pace_abs",
  "segments": [
    { "role": "warmup", "duration": { "unit": "seconds", "value": 900 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "recovery", "duration": { "unit": "seconds", "value": 90 } },
    { "role": "work", "duration": { "unit": "meters", "value": 800 }, "target": [198, 202] },
    { "role": "cooldown", "duration": { "unit": "seconds", "value": 600 } }
  ]
}
```
