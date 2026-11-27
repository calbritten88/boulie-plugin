---
name: boulie-get-started
description: Set up a new athlete's coaching in Boulie, from first conversation to a saved goal, season outline and detailed current week. Use when the athlete is new to Boulie, says "set up my coaching", "help me get started", "build me a plan", or when getCoachingContext returns onboarding state new_athlete or goal_set_no_plan.
---

# Getting an athlete started in Boulie

The aim of a first session: the athlete leaves with a goal they chose, a week-by-week arc toward it, a detailed current week they can train from, and a coaching record that knows who they are. Everything is agreed in chat first, then saved.

This is the default path, not a gate. If the athlete wants only this week, or wants to skip the goal, follow them, say what the skipped step would add, and make it easy to come back to.

## 1. Read what Boulie already knows

Call `getCoachingContext` once. Read these before asking anything:

- `onboarding.state`: `new_athlete` (nothing yet) or `goal_set_no_plan` (goal exists, no plan). If there is no `onboarding` block, the athlete is not new: tell them in one line that they're already set up in Boulie and you'll pick up from their plan, then use the coaching-session skill.
- `athlete.standingConstraints`: their weekly availability pattern and when it took effect. It already reflects their latest edits, so don't ask them to re-confirm it.
- `athlete.anchors`: FTP watts, threshold pace and max HR, with resolved ladders. These are the only basis for percentage targets.
- `capabilities`: the sport and intent vocabulary you will write with.

**What empty means:** a missing anchor means Boulie doesn't have it, not that it's zero. Ask the athlete. If they don't know, plan on HR or RPE and say so. Never estimate an anchor and write targets against the estimate. An empty availability pattern means it hasn't been set: ask how many hours they have and on which days.

## 2. Understand recent training

Call `getCompletedTraining` for the last 14 days. If the athlete has connected a device, it returns their rides with measured figures, which is enough to set a starting load.

If it returns `available: false`, use whatever other evidence you have: an activity source connected to this conversation, or what the athlete tells you. A few weeks of summary figures (how often, how long, what kind) is enough. You don't need per-activity detail for onboarding. With no evidence source at all, ask about the last month: typical week, longest session, anything hard, any time off. That's a legitimate basis.

If the athlete rides with a Wahoo or Hammerhead head unit and hasn't connected it, mention once that they can connect it in the Boulie app (Profile, then Devices). Each completed ride then sits beside its planned session automatically, so future reviews need no copying of numbers.

## 3. Agree and save the goal

Help the athlete name what they're preparing for: an event and date, or an open goal such as building fitness. Add numeric `targets` only if the athlete states a number; Boulie then tracks progress against their benchmarks. A goal with no numbers is normal.

Restate the goal (title, date, priority) and get an explicit yes, then call `setGoal`. Saving a goal doesn't authorise any plan change.

## 4. Agree the arc, then save the spine

Propose the weeks from now to the goal in one compact view: theme and target hours per week, with build, recovery and taper visible. Base the starting hours on what they've actually been doing, not on what they wish they did.

After explicit confirmation, call `writePlanSpine` with `macroTheme`, `targetHours` and a short `weekNote` per week. New weeks use `expectedVersion` 0. Don't add day detail beyond the current and next week.

## 5. Detail the current week

Follow the plan-and-adjust skill for the day-level proposal, `buildWorkout` previews and the confirmed `writePlan`. Respect the 0h days in `unavailableDates` unless the athlete says they've made time.

## 6. Seed the coaching record

Close with one `recordReconciliation` call, shown to the athlete and confirmed first:

- `athleteModel`: durable context they intentionally shared (background, constraints, preferences, open questions), marked provisional.
- `blockReview`: your provisional read of their recent training, marked provisional.
- `advanceCursorTo`: today.

Store only what they shared for coaching. No inferred traits, medical detail or raw activity data.

## Finish

Say what Boulie saved (goal, weeks, current-week sessions, record), using the versions and echoes the writes returned. Tell them where to see it, and that a fresh conversation can pick up from Boulie at any time.

## What Boulie can't do

- It stores no activity streams or GPS, and it can't fetch activities from other services.
- It can't set availability for them through a plan write. A one-week exception goes in `weekNote`.
- It can't compute targets without an anchor. Use HR or RPE until one exists.
