# PRD: Superset

**Status:** Draft v4 (2026-11-22), for Nik's review. Not approved.
**Owner:** Nik
**Sources:** idea chat (2026-11-11, private), PRD v3 (`inputs/workout-log-app-prd.md`), design canvas `1795362017-984f` (`flows.md`, design-brief v2), Notion "Workout 3" (read 2026-11-22).
**Next steps:** Nik answers the Open decisions (§13), then option research for P-01 and P-09, then v5. Then design-sync and stage 1b, then PRD-approved.

**Version log**
- v4 (2026-11-22): synced v3 to the canvas, which is newer and much more detailed. New: sign-in, multiple workouts, round-based logging, rating per exercise, equipment with weight lists, sync states, resume and discard. Every place v3 and the canvas disagree is a decision (P-01 to P-08).
- v3 (2026-11-11): first PRD, written from the idea chat.

Decision numbers: **P-xx** are PRD decisions (§13). **D-xx** are design decisions in `flows.md`. Screen and flow numbers like `2.6.1` are canvas numbers.

---

## 1. Overview

A gym logbook for your iPhone that shows exactly what to lift today, logs each round of a superset in one tap, asks how each exercise felt, and moves your targets up on the plan your trainer set. It replaces the per-session tables on the Notion "Workout 3" page.

## 2. Goals

1. **Know what to lift.** Every exercise shows today's sets, reps, and weight before you start, so you never guess or remember.
2. **Log fast.** One tap logs a whole round of a superset when you hit the targets. Fixing a set takes a few taps.
3. **Progressive overload.** Targets go up automatically, a little at a time, toward your trainer's rep range, using the weights you actually have.
4. **Never lose a session.** Logging works with no signal, and everything syncs once you're back online.

## 3. Non-goals

- No program design. The app doesn't choose exercises, sets, or rep ranges; those come from your trainer.
- No social features, sharing, coaching, or plans from others.
- No automatic step back: the app never lowers a target on its own (P-04).
- ~~No default weight jumps by equipment type~~: replaced by equipment weight lists (P-05).

## 4. User and context

- **Single user:** Nik, on an iPhone (the designs target iPhone Air), installed to the Home Screen, mid-workout, one-handed.
- **No sign-up.** The one account is set up in the backend (flows D-11).
- **Connectivity:** the gym can have no signal. Logging and viewing work offline; sync happens later (§6.7).
- **Units:** pounds (P-07).
- **Deadline:** unknown (Q-1).

## 5. Views and flows

| View | What it's for | Flows |
|---|---|---|
| **Sign in** | Face ID, with a backup | 1 |
| **Today** | The current workout's targets, Start or Resume, Switch | 2.1, 5 |
| **Session** | Log rounds, skip, fix a set, end, rate | 2 |
| **History** | Sessions, Workouts, and Exercises tabs; exercise detail | 3, 7.1 |
| **Setup** | Workouts, Exercises, and Equipment tabs; edit sheets | 4, 7 |

- **Flows:** `flows.md` in the canvas is the source of truth for UI behavior: 7 flows, 68 screens, 48 edge cases. Requirements below cite flows and edge cases.
- **Fidelity:** Close for every screen (the canvas default), except `1.1.2` (Rough).
- **Flow status:** every flow is **In progress**, so every requirement from the canvas is provisional until its flow is marked Final.

| Gate | Canvas version | Date |
|---|---|---|
| Synced into v4 (not a gate) | `1795362017-984f` | 2026-11-22 |
| PRD-approved | — | — |

## 6. Functional requirements

### 6.1 Sign in (flow 1)
- **R-6.1.1** Sign in with Face ID first, with a backup way in (`1.1.1`). No sign-up. How Face ID and the backup work is P-01.
- **R-6.1.2** Every screen and every piece of data is behind sign-in.

### 6.2 Today (flow 5, 2.1)
- **R-6.2.1** Today shows one workout: its supersets, and each exercise's target sets, reps, and weight for this session, with Start (`2.1.1`).
- **R-6.2.2** With two or more active workouts, Switch beside the name opens a list. Picking one changes Today only and doesn't start it (5.1, EC-5.1-1). With one workout, Switch is hidden (EC-5.2-1).
- **R-6.2.3** With no active workouts (new install, last one deleted, or all inactive), Today shows the empty state, and Create a workout opens the new-workout sheet (`5.3.1`, EC-4.1-2).
- **R-6.2.4** A session left mid-way (Home button, or the app reopened) shows In progress on Today, with Resume, and Discard to its left. Discard asks first, then deletes the whole session: nothing goes to History and targets don't change (5.4, EC-5.4-2).

### 6.3 Log a workout (flow 2)
- **R-6.3.1** A session runs superset by superset, round by round. One tap logs every exercise in the round as hit, at its target (`2.2.1`).
- **R-6.3.2** Skip one exercise in a round and log the others. Skipped sets don't count against progression (2.3, EC-2.3-1).
- **R-6.3.3** Fix a set: the pencil opens the number pad for reps and weight. A set that differs from its target shows in red with a filled pencil (2.4, EC-2.4-2). The decimal point works in the lb field only, once per number (EC-2.4-1).
- **R-6.3.4** "Machine taken?" lets you start a different superset first, only before the current superset begins (EC-2.2-1).
- **R-6.3.5** A workout has 1 to 5 supersets. With 5, the tiles share one row (`2.2.2`, D-29).
- **R-6.3.6** End workout early asks first. Every round not logged is marked skipped and doesn't count against progression (2.5, EC-2.5-1, D-20).
- **R-6.3.7** After a superset's last round, you rate each of its exercises: Too easy, In range, or Too hard. Continue stays off until every exercise is rated (2.6.1, EC-2.6-1). Rating is per exercise per session, and it's required (P-02).
- **R-6.3.8** When the workout is done, a summary shows next session's targets and what changed (`2.6.2`).

### 6.4 Progression
- **R-6.4.1** Each exercise has sets, a rep range from your trainer (for example 3 × 10, range 8–12), a weight, and equipment (`4.1.7`).
- **R-6.4.2** After each session, each exercise's next target follows its rating and its logged sets (EC-2.6-2):
  - **Too easy:** move to the next heavier weight.
  - **In range:** follow the rep progression (rule in P-04).
  - **Too hard, or any set missed:** hold.
  - **Skipped or unrated** (ended early): hold.
- **R-6.4.3** The next heavier weight comes from the weights listed for that exercise's equipment. On the heaviest weight, the target holds and the summary says "At heaviest". Adding a heavier weight in Equipment lets it move on (2.7, EC-2.7-1, P-05).
- **R-6.4.4** The app never lowers a target on its own. You can override any target, and the override becomes the new starting point (v3 §6.3).
- **R-6.4.5** Bodyweight exercises track reps only (P-06).
- **R-6.4.6** Changing an exercise's setup (rep range, sets, equipment, weight) takes effect next session. Past sessions stay as logged, with the range they were logged under (7.1, EC-7.1-1, EC-7.1-2).

### 6.5 Setup (flow 4, 7)
**Workouts**
- **R-6.5.1** Create a workout from + (any Setup tab). It opens with Superset A and one exercise, using defaults, and never starts empty (`4.1.6`, `4.1.7`, EC-4.1-4).
- **R-6.5.2** Edit a workout in a sheet: rename (the name is always a field), reorder supersets, add and remove exercises, open an exercise to edit it in place. Nothing applies until Save, Save is dimmed until something changes, and leaving with edits asks "Discard changes?" (4.2, 4.5.4, 4.5.6, D-09).
- **R-6.5.3** Delete an exercise or a superset after a confirm. A superset keeps at least one exercise, and a workout keeps at least one superset. Superset letters follow position (4.3, 4.4).
- **R-6.5.4** The workout menu has Duplicate, Make inactive (or Make active), and Delete. Make inactive applies at once: the workout leaves Today and Switch, and its history stays. A new, unsaved workout can't be duplicated or deleted (4.5, D-32, D-36).
- **R-6.5.5** Delete workout asks first. The workout and its sessions are gone for good, and Today moves to the next workout (`4.5.5`). **Conflicts with R-6.6.4; see P-08.**

**Exercises**
- **R-6.5.6** An exercise has one setup and one history wherever it's used. Editing it in any workout changes it everywhere. For a different setup, make a second exercise (D-33). How a change to a shared exercise is confirmed is open (flows D-34, provisional).
- **R-6.5.7** An exercise is Inactive automatically when no active workout uses it (EC-4.1-3).

**Equipment**
- **R-6.5.8** Equipment is a named list of the weights you have. Dumbbells, Cable, and Bodyweight are standard: they can be duplicated and renamed but not deleted. Equipment you add can be deleted only when no workout uses it (4.6–4.8, D-39, D-35).
- **R-6.5.9** Add weights one at a time or as a range (From, To, Step). Duplicates are skipped, and equipment keeps at least one weight (D-43, EC-4.6-2, EC-4.6-3).
- **R-6.5.10** "Used in N active workouts" lists the active workouts that use a piece of equipment (D-37).
- **R-6.5.11** Standard equipment starts with default weights; the values are open (flows D-40).

### 6.6 History (flow 3)
- **R-6.6.1** Sessions tab: sessions by date with total pounds lifted. Expand one to see every set and rating (3.1).
- **R-6.6.2** Workouts tab: exercises grouped by workout, each with its last result and a small bar per session. Exercises tab: every exercise A–Z (3.2).
- **R-6.6.3** Exercise detail: a bar per session (height is weight, fill is reps through the range), with a dashed line where the setup changed (`3.2.3`, `7.1.2`).
- **R-6.6.4** An exercise removed from its last workout keeps its history in the Exercises tab (EC-3.2-1, EC-3.2-2, D-47).
- ~~Personal record callouts~~: not in the designs (P-03).

### 6.7 Offline and sync (flow 6)
- **R-6.7.1** Viewing, logging, and Setup edits work with no signal. Everything is saved on the phone first (`6.1.1`).
- **R-6.7.2** When signal returns, changes sync to the database automatically (`6.1.2`). If sync fails, a banner explains it, with Retry, and data stays on the phone (`6.1.3`, EC-6.1-1).
- **R-6.7.3** History is never stuck on one phone: after sync, signing in on another device shows everything. Conflict rules and storage are P-09.

### 6.8 Import
- **R-6.8.1** The existing Notion "Workout 3" history is in the app at launch (P-10).

## 7. Data and state

| Data | Owner | Lives in | Resets / deletes |
|---|---|---|---|
| Account | Backend (set up by hand) | Database | Never in the app |
| Workouts, supersets | Setup | Phone, synced to the database | Delete workout (R-6.5.5) |
| Exercises (one setup each) | Setup | Phone, synced | P-08 |
| Equipment and weights | Setup | Phone, synced | Delete when unused |
| Sessions, sets, ratings | Session | Phone, synced | Discard (in progress only); delete workout (P-08) |
| Targets | Derived from the setup and the last session | Computed | Recomputed after each session |
| In-progress session | Session | Phone | Resume, Discard, or End |
| Today's chosen workout | Today | Phone | Falls back to the first active workout |

Unverified: what happens to unsynced data if iOS clears a Home Screen app's storage (P-09, §10).

## 8. Design

- The canvas and `elements.json` are the design source. Product-level rules that affect behavior: one accent button per screen (D-04); the accent is what you came to do, and black is the way out (D-02); edits apply only on Save (D-09).
- Built for mid-workout use: big tap targets (72px actions), minimal typing, one hand.

## 9. Success

You walk in, see today's targets, tap through each round, rate each exercise, and your numbers go up steadily without you thinking about it. No session is ever lost to bad signal.

## 10. Risks

| Risk | Status | Fallback |
|---|---|---|
| Face ID in a Home Screen web app (passkeys / WebAuthn) works for sign-in and stays signed in | Unverified (P-01) | Magic link or texted code |
| iOS keeps a Home Screen app's offline data; unsynced sessions survive app kills and storage pressure | Unverified (P-09) | Sync early and often; warn on long-unsynced data |
| Offline edits from two devices conflict | Unverified, low likelihood with one user | Last write wins per record |
| Notion history is sparse (3 dated columns, mostly empty set rows) | Verified (Notion, 2026-11-22) | Import what's there; Nik checks it once |

## 11. Validate before use

- Sign-in and sync on Nik's iPhone, Home Screen install, airplane mode mid-session, then reconnect.
- One real gym session end to end before the app replaces Notion.

## 12. References

| Source | Feeds |
|---|---|
| Idea chat (`private/chat-idea.md`) | §1–3, §6.3 (rating), §6.4, §6.7 |
| PRD v3 | §1–4, §6.4, §6.8 |
| `flows.md` (canvas `1795362017-984f`) | §5, §6, §8 |
| Notion "Workout 3" | §6.8, P-10 |

## 13. Decisions

### P-01 · How sign-in works        Status: Open · Explore
Source: v3 has no sign-in. The canvas has Face ID first, a texted code as the backup, and no sign-up (flows D-11, Decided). Whether the code backup exists and how it works is flows D-12 (Open).
Options: an options brief (stage 1 research) compares passkeys (Face ID through WebAuthn), a texted code, a magic email link, and recovery codes, on an iOS Home Screen app with the default stack (Supabase Auth).
Recommend: run the research now. The likely result is a passkey plus a backup, with a POC in stage 2 on your iPhone.
Nik:
Outcome:

### P-02 · Rating: per set or per exercise        Status: Open · Decision
Source: v3 §6.2 and the chat rate every set right after it ("How did it feel?"). The canvas rates each exercise once, after its superset's last round, and rating is required (2.6.1, D-17).
Options: (a) per exercise per session, required (the canvas); (b) per set (v3).
Recommend: (a). It's newer and fewer taps, and it answers v3's open question ("Should rating be required?": yes).
Nik:
Outcome:

### P-03 · Personal record callouts        Status: Open · Decision
Source: v3 §6.4 calls out new personal records. No canvas screen shows one.
Options: (a) drop for now and add later; (b) design one into the workout summary (`2.6.2`) before PRD-approved.
Recommend: (a).
Nik:
Outcome:

### P-04 · The rep progression rule        Status: Open · Decision
Source: v3 §6.3: add 1 rep per good session up to the trainer's target, then add weight and drop to 2 reps below the target (4×8 → 6). The canvas gives each exercise a rep range (3 × 10, range 8–12) and says In range "follows the progression" without defining it (EC-2.6-2).
Options:
(a) **Range rule:** In range with every set hit adds 1 rep, up to the top of the range. At the top, move to the next heavier weight and reset reps to the bottom of the range. (v3's 4×8 with a 2-rep drop is the same as range 6–8.)
(b) **v3 rule:** a single target with a fixed 2-rep drop, and no range.
Also needed: where does a new exercise start (the canvas default is 10 reps in 8–12), and is "every set hit" required for In range to add a rep?
Recommend: (a), with "every set hit" required.
Nik:
Outcome:

### P-05 · Where the next weight comes from        Status: Open · Decision
Source: v3 non-goal: no default weight steps, and the app asks for the new weight. The chat (18:13): "just ask, app should work without default weight type increments". The canvas: each equipment has a list of the weights you own, the next heavier weight comes from that list, and "At heaviest" holds (2.7, 4.6–4.8). Standard equipment's default weights are flows D-40 (Open).
Options: (a) next weight from the equipment's list (the canvas); (b) ask every time (v3).
Recommend: (a). It's how the canvas works, and it still has no generic "+5 lb" step: the steps are exactly your equipment. v3's non-goal is removed.
Nik:
Outcome:

### P-06 · Bodyweight exercises at the top of the range        Status: Open · Decision
Source: R-6.4.5. Bodyweight has no next weight.
Options: (a) hold at the top and say "At top of range"; (b) keep adding reps past the range; (c) suggest moving to a harder variation (as a note only).
Recommend: (a). It's consistent with "At heaviest".
Nik:
Outcome:

### P-07 · Units        Status: Open · Default
Source: every screen uses lb, and the number pad's decimal works "in lb fields only".
Recommend: pounds only. No kg setting.
Nik:
Outcome:

### P-08 · What deleting removes        Status: Open · Decision
Source: these conflict.
- Delete workout: "the workout and its sessions in History are gone for good" (`4.5.5`).
- An exercise removed from its last workout keeps its history (`3.2.4`, D-47).
- Deleting an exercise from Setup removes its sessions (D-35, not built yet).
- One exercise has one history across workouts (D-33).

So if Goblet Squat is in Workout 3 and Workout 4, and you delete Workout 3, what happens to Goblet Squat's sets logged in Workout 3 sessions?
Options:
(a) **History is kept per exercise:** deleting a workout removes the workout and its session list, but each exercise keeps its logged sets in History › Exercises. Only deleting an exercise (from Setup) removes its sets.
(b) **Deleting a workout removes everything logged in it**, including those exercises' sets (the canvas wording today).
Recommend: (a). It matches "one exercise, one history" and keeps progression data. The `4.5.5` confirm wording would change in the canvas.
Nik:
Outcome:

### P-09 · Offline storage and sync        Status: Open · Explore
Source: R-6.7. The chat: "working without signal but it should connect to a database".
Options: an options brief compares offline-first approaches on an iOS Home Screen app with Supabase: local store choice, a sync queue, conflict rule, and storage eviction. Stage 2 checks the meal-plan POC registry for proven pieces.
Recommend: run the research now. A POC in stage 2 tests airplane mode, app kill, and reconnect on your iPhone.
Nik:
Outcome:

### P-10 · Notion import        Status: Open · Default
Source: R-6.8.1. "Workout 3" has an 8/18/2026 column, an unlabeled column, and 11/12/26 (one summary value per exercise), plus a per-set table for 11/12/26 with only a few sets filled in.
Recommend: a one-time import script, not an app feature. It maps what's there into sessions, and you check the result once. The unlabeled column needs a date from you (Q-2).
Nik:
Outcome:

### Questions

| # | Question | Why it matters |
|---|---|---|
| Q-1 | Is there a date you want to use it in the gym by? | Scope and the build plan |
| Q-2 | What date is the unlabeled column on "Workout 3" (between 8/18 and 11/12)? | Import (P-10) |
| Q-3 | Will you ever use it on a second device (iPad, a new phone), or is "not stuck on one phone" just backup? | Sync design (P-09) |
| Q-4 | Is "Machine taken?" (start a different superset first) the only reordering, or can you also reorder exercises inside a superset mid-workout? | Session logic |

### Linked design decisions that are still open
flows D-12 (code backup, with P-01), D-34 (confirming shared exercise changes), D-40 (default weights), D-48 (delete message wording), D-49 (new equipment in the picker), D-53 (leftover logic). These are answered in `flows.md`.
