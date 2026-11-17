# PRD: Workout Log App

**Status:** Draft (v3, 2026-11-11)
**Owner:** Nik
**Next steps:** Review the PRD, then HLD, then coding-agent prompt. Each step needs Nik's approval before moving on.

---

## 1. Overview
A gym logbook for your phone that shows exactly what to lift today, asks how each set felt, and keeps you progressing on the plan your trainer set. It replaces the per-session tables on the Notion "Workout 3" page.

## 2. Goals
1. **Know what to lift.** Every set has a target for reps and weight, so you never have to guess or remember what you did last time.
2. **Log fast.** Logging takes one tap per set in most cases.
3. **Progressive overload.** Targets go up automatically, a little at a time, toward your trainer's rep targets.

## 3. Non-goals
- No program design. The app doesn't decide your exercises or rep targets; those come from your trainer.
- No default weight jumps by equipment type. When it's time to add weight, the app asks you for the new weight.

## 4. User and context
- Single user: Nik, logging on his phone mid-workout, with the app installed to the Home Screen.
- **Works without signal.** Viewing and logging work offline in the gym. Everything is saved to a database and syncs automatically once you're back online, so history is never stuck on one phone.

## 5. Views

| View | What it's for |
|---|---|
| **Home** | Today's workout and a start button |
| **Workout session** | Log each set against its target |
| **History** | Past sessions by date |
| **Exercise progress** | Weight, reps, and set ratings over time for one exercise |
| **Workout setup** | Build and edit workout templates |

## 6. Functional requirements

### 6.1 Workout templates
- You build a workout once, with its exercises, order, and supersets (A1/A2, B1/B2, C1/C2), plus sets per exercise and your trainer's rep target for each exercise.
- Starting a session pre-fills every set with its target reps and weight.
- Bodyweight exercises (like TRX Row) track reps only.

### 6.2 Logging a set
After each set, the app asks two things:
1. **Did you hit the target?** One tap for yes. If not, you enter the reps and weight you actually did.
2. **How did it feel?** Too easy, in range, or too hard.

### 6.3 Progressive overload
Your trainer's rep target is the goal, and the app builds you up to it one step at a time.

| What happened last session | Next session's target |
|---|---|
| Hit every set, none felt too hard | Add 1 rep |
| At the trainer's rep target on every set | Add weight, and drop reps to 2 below the trainer's target (for example, on 4×8 you'd start at 6 and climb back to 8) |
| A set felt too easy | Add weight (same 2-rep drop) instead of adding a rep |
| Missed a set, or a set felt too hard | Stay the same |

- The app never moves you backward on its own.
- When it's time to add weight, the app asks for the new weight. This works the same way for dumbbells, cables, and kettlebells.
- You can always override a suggestion, and overrides don't break the progression.

### 6.4 History and progress
- Past sessions are viewable by date.
- Each exercise has a trend of weight and reps over time, including how sets felt.
- When you beat your previous best on an exercise, the app calls out the new personal record.

## 7. Data
- Existing workout history from Notion is imported at launch, including the 8/18/2026 and 11/12/2026 sessions on Workout 3.
- After launch, the app is the source of truth.

## 8. Design
- Built for use mid-workout: big tap targets, minimal typing, one-handed use, and no clutter.
- Today's targets are always at the top.
- Look and feel: TBD.

## 9. Success
You walk in, follow the targets, tap through your sets, and your numbers go up steadily without you having to think about it.

## 10. Open questions
- Should rating a set be required, or optional when you hit the target?
- What's the look and feel?

## 11. References (Notion)

| Page | Use for | Link |
|---|---|---|
| Workout 3 | Exercises, trainer targets, and past sessions to import | https://app.notion.com/p/400cf4bc6d1081ed8216caade4156b4d |
