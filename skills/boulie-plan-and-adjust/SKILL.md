---
name: boulie-plan-and-adjust
description: Plan or change training in Boulie, from building a structured workout (intervals, over-unders, sprints, pyramids, track reps) to a week or a block, including adjusting for illness, travel, fatigue, weather or a busy week. Use when the athlete says "plan next week", "build me a workout", "move Thursday", "I'm sick", "I only have 5 hours this week", or agrees a change that needs saving to their Boulie plan.
---

# Planning and adjusting in Boulie

Boulie's plan is canonical: what the athlete trains from and sees in the app. Changes are negotiated in chat, then committed once, validated, with the athlete's explicit confirmation of the specific dates and values.

The loop: **read → outline → confirm → build → commit → echo**.

## 1. Read only what the decision needs

- `getCoachingContext` gives the spine through the goal, current-week detail, anchors, `standingConstraints` and `recentDelivery`.
- `getPlan` for a specific date, week or range. Always read a week before replacing days listed in its `daysWithDetail`. Use `detail: "full"` only when you need authored blocks or exact segment targets.
- If `unjudgedPlannedSessionCount` is present and you're about to write the future plan, close that gap first (see the coaching-session skill), unless the athlete wants to go straight to the change.

**Inputs that are already settled:** `standingConstraints` is the athlete's current weekly pattern, so don't ask them to re-confirm it. Days in `unavailableDates` are 0h days. A session there commits with a warning, so propose one only when the athlete says they've made time.

## 2. Outline in chat

Propose the change compactly: each affected date, the session's purpose, its duration, and its targets in real units. Read watts and paces from the `athlete.anchors` ladders. Don't compute them in your head from a remembered FTP.

- Pick one target basis per session. Rides use `power_pct` when the athlete has power, otherwise `hr_pct` or `rpe`. Runs use `pace_pct`, `pace_abs`, `hr_pct` or `rpe`.
- `pace_pct` is a percentage of threshold **speed**: 100 is threshold pace and values above 100 are **faster**.
- Set week hours with `recentDelivery` in view. A target far above what the athlete has actually delivered produces a plan they fall behind on.
- For a key session, up to three short `successCriteria` say what you'll check afterwards. Omit them for routine work.

## 3. Confirm

Restate the full proposal (dates, sessions, durations, targets, week hours) in the confirmation message itself, then wait for an explicit yes. Don't assume the athlete can still see earlier messages. Discussing a plan or reading one isn't authorisation.

## 4. Build structured workouts

`buildWorkout` takes one flat list of segments in the order they're ridden or run, with every repeat written out. Boulie finds the repeats, resolves targets and returns a preview plus a `workoutDraftId`. It saves nothing to the plan.

- **Order:** `warmup` segments first, `cooldown` segments last.
- **Interval sections:** a run of `work` and `recovery` segments that starts and ends with `work`. Every set (the `work` between recoveries) is identical, with one identical `recovery` between sets. Add easy time after the last set to the cool-down, never as a final recovery.
- **Over-unders and other patterns:** write the cycle out as `work` segments (over, under, over, under…). Boulie detects the cycle. The under phase is `work`, not `recovery`.
- **Different blocks in one session** (over-unders then sprints, or each rep of a pyramid): put a `steady` segment with a target between them. `steady` starts a new section.
- **Units:** only `work` may use `meters`. Everything else is `seconds`: convert a 200 m jog at its expected pace, and keep the distance in the coaching notes.
- **Targets:** `[min, max]` in the session's `targetBasis`; repeat a number for a fixed target. `work` and `steady` need one. `pace_abs` is seconds per km, so smaller is faster.

Check the preview `summary` reads as the agreed session (for example `3x 4x (2min @ 347W, 1min @ 297W) / 5min recovery`) before showing it. On a failure, fix exactly what its `suggestion` names. Never switch to `blocksJson` to get around a rejected constraint; use it only for existing prescriptions or shapes `buildWorkout` can't express, with `writePlan` `validate: true`. For worked examples that compile (nested over-unders, two blocks, a run pyramid in metres), read `references/workout-examples.md`.

## 5. Commit

- Day sessions: `writePlan`, with the confirmed sessions, `workoutDraftId` for built workouts (don't recreate their blocks), and the plan version from your read.
  - A day owns sessions in slots 1–4. A double day is two sessions, and `restDay: true` with no sessions is an explicit rest day.
- Week themes, hours and notes beyond the detailed weeks: `writePlanSpine`. Fields merge, so send only approved changes.
- A one-week exception (travel, illness) goes in the plan and that week's `weekNote`. A plan write never changes standing availability.

## 6. Echo what was saved

Compare the returned resolved units and `changes` list with what the athlete confirmed, then tell them the committed version and what Boulie actually saved.

- If something differs, say so, and confirm before any corrective write that changes the approved proposal.
- A version conflict means someone else changed the week. Re-read it, and if the result changes your proposal, re-confirm before retrying. An identical retry may reuse the original confirmation.
- `writePlanSpine` commits weeks in order and stops at the first failure. Report which weeks committed and retry only the rest.

## Adjusting for life

The same loop, shorter. Find out what changed and for how long, read the affected days, and propose a concise replacement that protects the session that matters most that week. If the change follows a session and the athlete has a device connected, read what was actually ridden with `getCompletedTraining` before adjusting the rest of the week. Confirm, then commit. When illness or fatigue is involved, propose less, not a rearranged version of the same load.

## What Boulie can't do

- It doesn't invent future detail: weeks beyond what's stored are themes only until you agree day detail.
- It can't take targets from anywhere except `athlete.anchors`. If an anchor is wrong, propose an update first (coaching-session skill).
- Validation sets a safety floor. It doesn't write the plan for you, and a validation pass isn't the athlete's approval.
