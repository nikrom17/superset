# Superset · Intake

Stage 0, 2026-11-17. Raw inputs are in `inputs/` and are never edited. Stage 1 reconciles them into the PRD.

## North star (Nik's words)

> "Can you help me structure this table so I can have a target rep and weight for each exercise and then an actual rep weight. I want this for each set" (idea chat, private, 16:56)

> "I want to creat a pWA for this … One thing I want is to ensure progressive overload … Maybe an easy check box or just always go up a rep and then when at target go to weight?" (idea chat, private, 17:11)

> "i should be able to rate the sets like too easy, in range, too hard … i like it working without signal but it should connect to a database" (idea chat, private, 18:09)

> "You walk in, follow the targets, tap through your sets, and your numbers go up steadily without you having to think about it." (PRD v3, section 9, written by Claude from the chat; Nik hasn't reviewed it yet)

## Inputs

| File | What it is | Date |
|---|---|---|
| `private/chat-idea.md` | The idea chat, verbatim; gitignored (personal data, G-17): Notion table → product description v1, v2 → PRD | 2026-11-11 |
| `inputs/workout-log-app-prd.md` | PRD v3 (Draft). A copy; the original stays in `~/Claude/Projects/Workout Log App/`. Identical to the canvas's `uploads/` copy. | 2026-11-11 |
| `inputs/design-canvas.md` | Link and page list for the Claude Design canvas | 2026-11-17 (still changing) |
| Notion "Workout 3" | https://app.notion.com/p/400cf4bc6d1081ed8216caade4156b4d. Exercises, trainer targets, and the 8/18 and 11/12 sessions to import. Read in full at stage 1. | live |

**Newest input:** the canvas. It's six days newer than PRD v3 and already goes past it (for example, the sign-in screen and the Rack page). Stage 1 treats the canvas decisions as the latest intent and confirms each one with Nik.

## Workflow rules

- Gates follow the prd-to-prod pipeline (stages 0–10). Nik approves PRD-approved, HLD-ready, the HLD, and the criteria freeze.
- Approval is the word "approved" (or an explicit go-ahead) in chat or on a `Nik:` line. Praise isn't approval.
- Files are written only in `~/Claude/Projects/Superset/`. The design canvas, Notion, and `~/Claude/Projects/Workout Log App/` are read-only for the skill.
- Decisions and open questions go in the Decisions section of the doc they're about, with a `Nik:` line.

## Hard context

| Item | Value |
|---|---|
| User | Single user, Nik, iPhone, installed to the Home Screen (PRD v3 section 4) |
| Connectivity | The gym can have no signal; logging must work offline and sync later (chat 18:09) |
| Deadline | unknown → stage 1 |
| Reuse | unknown → asked at intake (below) |
| Privacy | Workout history is personal data. Whether the repo or app is public is unknown → stage 1 |
| Content rules | None stated |

## Decisions

### D-01 · Project name        Status: Decided
Source: intake, 2026-11-17.
Nik: "call the project superset"
Outcome: The project is **Superset**, at `~/Claude/Projects/Superset/`. The earlier `Workout Log App` folder is left as is. The canvas keeps its "Workout Log" titles until Nik changes them.

### D-02 · Extra material to read        Status: Decided
Source: intake question, reworded on 2026-11-22 to "existing material to read" (the stack isn't decided at intake; it's committed at the HLD).
Nik: give more info to intake. First update the canvas structure with the UI/UX agent, get notes on what's finalized, and pull that in.
Outcome: 2026-11-22. Nik pastes `design-brief` and then `adopt` into the design chat. When the canvas has `flows.md`, `elements.json`, and a Status table, intake records it (version, flows, and what's Final) and stage 1 starts with syncing PRD v3 to it. Notion "Workout 3" is the only other input. No stack commitment.
Done 2026-11-22: the canvas was restructured (version `1795360630-01fe`); see `inputs/design-canvas.md`.
