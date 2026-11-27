---
name: boulie-coaching-session
description: Run a returning athlete's coaching session in Boulie. Catch up on what happened since last time, review completed sessions against the plan, answer the coaching question, and save what was agreed. Use when the athlete says "how did my week go", "review my ride", "check in", "what's next", reports a session, or starts any conversation about existing training in Boulie.
---

# A coaching session in Boulie

Coaching is episodic: the athlete checks in a few times a week. Each session should start from what Boulie knows, close any gap between what was planned and what happened, take the athlete's actual question, and leave the record better than it found it.

## 1. Orient with one read

Call `getCoachingContext`. It returns the goal, athlete context, anchors, the week spine through the goal, current-week detail, and the record state.

**Which result is enough:**

- `record.unjudgedPlannedSessionCount` absent and the athlete's question is about the future: you have enough. Go to step 3.
- `record.unjudgedPlannedSessionCount` present: planned days have passed without a judgment. Read `getChangesSince` before writing any future plan. A question about something else doesn't need reconciliation first.
- A review question ("how did my week go", "review my ride") **is** the judgment gap for those dates. Do step 2 for them: judge the sessions and offer to record the review, rather than only describing the week.
- `record.recentDelivery`: hours actually delivered against what was planned. Read it before setting any week's hours.
- For another week's full detail or full `weekNote`, use `getPlan`. For history (past judgments, reviews, why the plan changed, threshold readings), use `getCoachingRecord`: index, then list, then detail. The record nests block review, week review, session judgment, then the day's plan and ride, and each detail offers ready calls to walk down or back up.

## 2. Close the judgment gap (when one exists)

`getChangesSince` returns the unjudged sessions, weeks awaiting review, and plan changes since the cursor.

**When the athlete has a device connected to Boulie** (a Wahoo or Hammerhead head unit), Boulie already holds their completed rides, and `getChangesSince` includes a `completedActivities` pointer. Call `getCompletedTraining` for the gap (up to 14 days per call). It lays each day's planned sessions beside the completed activities, with measured time, power, NP, intensity factor, load, zone time, laps and the power curve. This is the best evidence available:

- Judge from it rather than asking the athlete to repeat figures. Ask only what the data can't tell you, such as how it felt.
- Activities are not paired with sessions and laps are not matched to steps. Pair by date, sport and start time, and for interval sessions read the laps against the prescribed steps.
- Request exactly the dates in question (Monday to Sunday for a week; future dates still show what's planned), and quote the response's range `totals` (planned and completed minutes and load) rather than adding rides up yourself. `activitiesWithoutLoad` counts rides with no power, so say the completed load leaves them out.
- Use `qualityTier` `device_power`, and don't cite peak figures from these rides: Boulie measures device bests itself.

**Otherwise**, or when `getCompletedTraining` returns `available: false`, the evidence comes from you: any activity source connected to this conversation, or what the athlete tells you. Summary figures (duration, distance, climbing, average speed) settle most sessions. Look at detail only when a key session turns on power or pace. An athlete's report is a legitimate basis for a judgment.

Then:

- Key sessions are the ones the plan gave `successCriteria`. Judge those against their prescription, citing `sessionId`, and check each criterion once. Don't promote a session to key because it carried the most load.
- Fold routine sessions into a week review rather than judging each one.
- A date with two sessions has two efforts: match by sport, then by time (slot 1 is earlier).
- Don't mark a session missed only because your activity listing lacks it. Ask.
- `notYetDue` sessions (today) are judged only if the athlete says they happened.

Show the proposed review, get confirmation, then write it as one `recordReconciliation` call with the judgments, any week review and `advanceCursorTo`.

- Put figures under the standard keys (for example `workAvgWatts`, the average over the prescribed work only). Standard keys stay machine-readable, and Boulie computes intensity variance from them. Reasoning and caveats go in the evidence note.
- Use `responseNote` for why the decision follows when it isn't obvious. Later sessions read it beside the decision, so they see the reasoning as well as the conclusion.
- Reuse an `operationId` only to retry the identical write after a timeout, never for a correction. If the athlete explicitly asked you to record a stated outcome, that authorises that outcome only. Anything you've inferred, any anchor change or any athlete-model revision still needs its own confirmation.

**What empty means:** "The judgment frontier is current" means no reconciliation is owed. Go straight to the question.

## 3. Take the coaching question

Answer from the record and the plan as stored. Future plan detail is exactly what Boulie returns. Don't invent sessions or weeks that aren't there.

If the answer changes the plan, follow the plan-and-adjust skill.

## 4. Write back what matters

Before closing, check whether anything agreed should persist:

- An anchor change, such as a new FTP. Propose the value with the reasoning, get explicit confirmation, then write it through `recordReconciliation` `anchorUpdates`.
- A demonstrated best Boulie doesn't measure itself (a race time, a run test): write it as a benchmark, not prose. Benchmarks are queryable, compared against goal targets and shown to the athlete; the same number in athlete-model prose has to be re-read by every future session. Record the conditions, because a fatigued end-of-ride peak and a fresh test are not the same evidence. Goal progress marked `untested` means no benchmark exists yet: a reason to plan a test, not a lack of progress.
- Durable new context (a new constraint, an injury the athlete wants factored in): propose an `athleteModel` revision and confirm it first.
- A correction to a past judgment: pass the `judgmentId` from `getPlan` as `supersedes`.

Don't write just because the conversation is ending or a read showed outstanding work.

## What Boulie can't do

- It doesn't fetch activities from other services. It holds completed rides only for athletes who connected a device in Boulie.
- It doesn't judge sessions on its own. Figures are measured, not judged.
- It doesn't change the plan through a review. Plan changes are a separate, confirmed `writePlan`.
