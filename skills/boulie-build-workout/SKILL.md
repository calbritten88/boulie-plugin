---
name: boulie-build-workout
description: Build a structured workout in Boulie with buildWorkout, so it compiles first time. Use when the athlete wants intervals, over-unders, nested repeats, sprints, a pyramid, track reps, tempo blocks or any session with a structure, or when buildWorkout returns a TIMELINE_ error.
---

# Building a workout in Boulie

`buildWorkout` takes one flat list of segments in the order they're ridden or run. Boulie finds the repeats itself, resolves targets against the athlete's anchors, and returns a preview and a `workoutDraftId`. It saves nothing to the plan. Committing is a separate, confirmed `writePlan` (see the plan-and-adjust skill).

## The grammar

Write every segment out: no repeat counts, no nesting.

1. **Warm-up first, cool-down last.** `warmup` segments open the list and `cooldown` segments close it.
2. **An interval section** is a run of `work` and `recovery` segments:
   - It starts and ends with `work`. Never end on a `recovery`: add that time to the cool-down, or to the `steady` segment that follows.
   - One `recovery` sits between sets, and every recovery in a section is identical.
   - **Every set in a section is identical.** A set is the `work` run between recoveries.
3. **Patterns inside a set are found for you.** Write an over-under set as over, under, over, under… all as `work`. The under phase is `work`, never `recovery`.
4. **`steady` starts a new section.** Different work in one session (over-unders then sprints, or each rep of a pyramid) needs a `steady` segment with a target between the parts, not a `recovery`.
5. **Units.** Only `work` may use `meters`. Warm-up, recovery, steady and cool-down are `seconds`: convert a 200 m jog to seconds at the expected pace, and keep the distance in the coaching notes. Durations are whole numbers.
6. **Targets** are `[min, max]` in the session's `targetBasis`. Repeat a number for a fixed target. `work` and `steady` need one; warm-up, recovery and cool-down may omit it for Boulie's defaults.
   - `power_pct` is % of FTP.
   - `pace_pct` is % of threshold **speed**: 105 is faster than threshold.
   - `pace_abs` is seconds per km, so a smaller number is faster.
   - `hr_pct` is % of max HR, and `rpe` runs 1–10.

## Recipe

1. Read `athlete.anchors` and the sport's intents from `getCoachingContext`. Pick one `targetBasis`: power when the athlete has it, otherwise HR or RPE; pace for runs.
2. Sketch the session as sections: warm-up, each distinct block, cool-down. Join blocks with `steady`.
3. Expand each block into its segments, check the last segment of every section is `work`, then call `buildWorkout`.
4. Check the preview `summary` reads as the session you agreed (for example `3x 4x (2min @ 347W, 1min @ 297W) / 5min recovery`), and that the total duration or distance is right. Then show the athlete the preview in real watts or paces.
5. On a timeline failure (the codes in the table below), fix exactly what the failure's `suggestion` names and resend. Don't switch to `blocksJson` to get around a rejected structure.

For worked examples that compile (nested over-unders, two blocks in one session, a run pyramid in metres), read `references/examples.md`.

## Common failures

| Code | Fix |
|---|---|
| `TIMELINE_RECOVERY_POSITION_INVALID` | A section ends or starts with recovery, or two recoveries touch. Fold trailing easy time into the cool-down or a `steady` segment |
| `TIMELINE_SETS_NOT_IDENTICAL` | Sets differ: fix a missing segment, or split the different work into its own section with `steady` between |
| `TIMELINE_RECOVERIES_NOT_IDENTICAL` | Make the recoveries match, or split the section with `steady` |
| `TIMELINE_ROLE_REQUIRES_TIME` | A non-work segment is in metres: convert it to seconds |
| `TIMELINE_PATTERN_TRUNCATED` | The last cycle in a set is incomplete: add the missing segment |
| `TIMELINE_TARGET_REQUIRED` | A `work` or `steady` segment has no target |
