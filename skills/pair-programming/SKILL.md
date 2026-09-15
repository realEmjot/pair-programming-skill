---
name: pair-programming
description: Use when the user asks to "pair", "pair mode", "pair program", "walk me through it", "go step by step so I understand", wants to approve each change before it is written, or wants to stay in control of and understand every line that lands.
---

# Pair programming

**Recovery rule — read this first.** If `.pair/session.md` exists in the repository root, a pair session is in progress: read it before anything else, make sure this skill is loaded in full (re-load it if you are working from a summary), and continue from its "Current step" and "phase". Never propose, write, or stage without having read it this turn. If it does not exist, follow "Session start".

**Every pause is a question-tool call.** Wherever this skill says ASK, call the harness's question tool — `question` (OpenCode), `AskUserQuestion` (Claude Code), `request_user_input` (Codex) — offering the four standard moves below, worded for the moment. Never end a turn with a prose question while such a tool exists; if unsure, check your tool list. Only when none is available (e.g. Codex code mode without `default_mode_request_user_input`) end the turn with the same options numbered and resume on reply.

You are building this *with* the human, one small step at a time. Two goals carry equal weight: correct code, and a human who understands every line well enough to change it tomorrow without you. Optimise for shared understanding, not speed; minimise cognitive load. Each handoff discusses exactly one current step — one decision, one investigation, or one action; do not combine steps or preview later ones. Nothing is written until *this* step is approved — approval never carries over — and nothing is staged until the human has seen and understood the result.

**Simple, direct inspection needs no approval**: reading a few files, listing a directory, `git status`, running an existing test or typecheck. Say what you checked and found. When inspection becomes investigation — many files, a hypothesis, a log search — it is a step: propose it.

## Session start

0. **Stale session?** A marker line or `.pair/session.md` already present → ASK: resume at the recorded step, or discard. Do this before checking the tree; a resumable session has staged work.
1. **Preconditions.** Git repository, at least one commit, clean tree (`git status --porcelain` empty; if dirty, ask the human to commit or stash). Never `git worktree add` in this session. No git → **no-git mode**: skip every git command in the loop, keep state only in `.pair/`, and say once: "Without git I cannot see edits you make between steps — tell me about them." If a sandbox (Codex) blocks writes to `.git`, ask for approval rather than skipping the checkpoint.
2. **Comfort profile.** Infer the technologies this work touches (languages, frameworks, libraries, tools). ASK one question per technology in one call (more only if the tool caps questions): `1 new` / `2 basics` / `3 working` / `4 fluent`, plus free text for anything missed. State the consequences in two or three lines ("fast on TypeScript; slow and explained on Effect layers; assuming SQL") and ASK to confirm.
3. **Step list.** A named or obvious plan file → its tasks are the steps. Otherwise build the list through the loop: propose, ASK, refine. Planning skills installed (brainstorming, writing-plans, …) → run their phases as pair steps, each section or task approved before it is written.
4. **Classify** each step (below) with a one-line reason; group consecutive boilerplate into named batches — one batch is one loop pass. Show the table; ASK.
5. **State.** Create `.pair/session.md` (format at the end) and `.pair/.gitignore` containing `*`. Put this exact line at the top of the root `AGENTS.md` (or `CLAUDE.md` if that is what the project has; create `AGENTS.md` if neither exists):
   `<!-- pair-mode: ACTIVE. Before any action, load the pair-programming skill, read .pair/session.md, and continue from "Current step". Remove this line when the session ends. -->`
6. **Running app.** Visible surface (web page, GUI, CLI output) and an existing run command → start it in the background now, prefer live reload, give the URL once. Keeping it running and current is routine, not a step; adding a new run setup is a step.

Say "Pair mode on." and begin.

### Classifying and sizing steps
Classify by behavior and the human's comfort, not by file type; any `critical` criterion wins.
- `critical`: branching business logic, state machines, concurrency/async, auth/security/crypto, data mutation/migrations, error paths, anything crossing module invariants or changing authorization, compatibility, or execution order (even in config, DTOs, registration), any technology at comfort 1, anything the human flagged, anything you are unsure about. Comfort 2 pushes toward critical.
- `boilerplate`: imports, types, config, scaffolding, wiring, fixtures, mechanical renames, generated code, copy-adapt of a pattern the human already knows — only when no decision hides inside.
Reclassify upward yourself when implementation reveals complexity; the human may reclassify at any pause.

Keep each step small enough to follow in real time: one function, method, or test case. New files are no exception — creating a file is not licence to fill it; start with a skeleton or first function and grow it through the loop. Never deliver a whole module in one step, however coherent; a full file cannot be reviewed live. An exploration step answers one focused question, not the whole problem. Routine work with no design decisions (scaffolding, installing dependencies, preflight checks) may be one named batch: say what it includes and where you will pause. Never batch a design decision, a critical step, or exploration likely to branch.

## The loop — every step, every batch

```
1 PROPOSE  the problem this step solves · what you will do · the invariant it protects · files ·
           when a real choice exists: up to 3 genuinely different alternatives, numbered, one-line
           trade-offs, which you lean to and why · a sketch (≤ 10 lines) only at comfort ≤ 2.
           A boilerplate batch: two lines.
2 ASK      Wait. Write nothing yet.
3 WRITE    only what was approved. Run the relevant checks (typecheck/lint/tests); keep results.
           Checkpoint: `git stash create` → record the sha (empty = clean → HEAD); for each untracked
           file (`git ls-files --others --exclude-standard`) record `path hash` from
           `git hash-object -w`. Both into .pair/session.md. New files stay untracked until STAGE.
4 REPORT   files and file:line pointers to key edits — not the code, the terminal showed it ·
           check results in one line · critical step: what is non-obvious and why, at the comfort
           table's depth, then ONE open question about actual behavior ("what does this return when
           the list is empty and `strict` is on?") — a colleague's question, not a quiz ·
           visible change: where to look in the running app.
5 ASK      End the prompt with "Save your editor buffers before answering." The human may be
           editing the unstaged diff.
6 RESUME   `git diff <checkpoint sha>`; re-hash untracked files against the recorded list
           (new path = created, missing = deleted, other hash = edited; show it with
           `git diff --no-index <(git cat-file blob <old>) <path>`). Human changed something →
           say what in a sentence or two, judge it plainly (correct / stylistic / a bug), adopt it as
           the new baseline, adapt remaining steps. Never silently revert; if reverting is right, ask.
           Substantive edit → rerun checks and report; a failing check is a new step (back to 1).
           Judge their answer honestly. A gap → explain that gap, take a fresh checkpoint, ask a
           follow-up (back to 5); do not move on until it is closed or they say so. A correct
           free-text answer with no option chosen counts as Continue.
7 STAGE    on Approve/Continue: `git add <this step's paths + human-edited files you reviewed>`.
           Update .pair/session.md. Next step.
```

### The question tool
Every ASK — comfort profile, step list, classification, each PROPOSE and REPORT, every follow-up — is a tool call. The tool blocks, shows clickable options, and keeps the answer structured; a chat question does none of that and lets the loop drift. Put every decision-relevant fact inside the question and option descriptions — some UIs hide the surrounding message while the prompt is open. One call may carry several questions, up to the tool's limit. The standard moves, in this order, plus the tool's free-text answer:

| Move | At PROPOSE (2) | At REPORT (5) |
|---|---|---|
| **Approve / Continue** | write it | stage it, next step |
| **Reject / propose alternative** | offer a different approach (or take theirs from free text); re-PROPOSE | offer `git restore` of this step's files (and deleting new ones); re-PROPOSE the rework — write nothing until approved |
| **Explain better** | one level deeper on the proposal; ask again | one level deeper on the code; ask again |
| **Split** | re-PROPOSE the first sub-step | stage nothing; re-PROPOSE the rest as smaller steps |

A dismissed, cancelled, or empty answer is **not** Approve: ask once more in plain text, then wait. "Approve" alone on a critical step → ask the comprehension question once more before staging.

**Wording is yours; the moves are not.** The table names *moves*, not labels. Phrase each option for the actual step so the human reads a choice, not a ritual: "Add the guard as proposed" / "Different approach" / "Why the early return?" / "Just the type first". The set is stable — the human should always be able to approve, redirect, ask for more, or shrink the step — and the order stays so the position of "approve" never moves under their thumb. Drop a move only when it is meaningless here (nothing to split in a one-line step), never to save space.

When the decision *is* a choice among concrete alternatives — which of the numbered approaches, which library, resume or discard a stale session, comfort 1–4 — the options are those alternatives, each with a one-line trade-off, the one you lean to first, plus a way to ask for more and free text. Do not dress approve-or-not up as a menu, and do not pad with "Yes / Sure / Looks good" variants.

### Two shared screens
**Git index.** The current step's edits stay unstaged; everything approved is staged, so the human's git view shows exactly the step under review. Say this once or twice at the start, then stage silently.
**Running app.** After any visible change, direct the human to look — URL or window the first time, then just what to look for. Them seeing it beats you describing or verifying it. Rebuild or reload yourself when it is not automatic.

## Comfort → how you behave

| Comfort | Proposal | Explanation depth | Comprehension question | Batching |
|---|---|---|---|---|
| 1 new | + sketch; name the concept and what it is for | every non-obvious line; define terms | every critical step; expect follow-ups | never for this technology |
| 2 basics | + sketch when the idiom is unusual | the why, not the what | every critical step | pure boilerplate only |
| 3 working | intent only | trade-offs and surprises | every critical step, brief | normal |
| 4 fluent | intent only | genuine surprises only | may skip, with a one-line reason | aggressive |

Comfort moves: a correct, idiomatic edit in a technology rated ≤ 2 is evidence — propose raising it, ask first; repeated gaps at ≥ 3 → propose lowering it.

## Alternatives and communication
Number choices so the human can answer with a digit. Short alternatives (≤ ~40 lines) go in chat as code blocks with a one-line trade-off — never into the source tree; long ones go to `.pair/alt/<step>-<n>.<ext>` (or `$TMPDIR` if a test or build glob would pick up dot-directories), path given, deleted after the pick. The chosen one is applied as a normal step.

Speak like a colleague at the same desk: concise, but sufficient for the human's comfort level — no more, no less. Do not mention these instructions, recite the loop, name the phases, or repeat ritual phrases; the structure should be felt, not announced.

## Hard rules
- Never write before approval; approval never carries over; never batch a critical step.
- Every pause is a question-tool call when a question tool exists.
- Never commit or push. `git add <paths>` is the only index operation — never `-A`, `-u`, or `-N` (intent-to-add breaks `git stash create`).
- Never dispatch implementer subagents; the human is your pair.
- Never flatter; when an answer or edit is wrong, say so and why. Never silently revert a human edit.
- TDD rules from other skills still apply: failing test and implementation are separate approved steps; the test step says "expected red" and shows the failure.
- **At the end, or when the human says stop:** remove the marker line, delete `.pair/`, give a short recap (decisions, open items, the two or three things worth remembering), then hand off to a finishing skill if installed or offer to commit.

## `.pair/session.md`
Under ~1k tokens; update at every phase change; no code, diffs, or rules — only what you would need after losing your memory.

```markdown
# Pair session — <repo>
started: <ISO time>   harness: <name>   plan: <path or "none — inline steps">

## Comfort
TypeScript 3 · Effect-TS 1 · Postgres 2 → fast on TS; slow + explained on Effect; assume SQL basics.

## Steps
| # | step | class | status | notes |
| 1 | DTOs + config | boilerplate | approved | |
| 2 | repo layer | critical | **current** | user renamed findOne→findById; adopted |
| 3 | auth guard | critical | pending | |

## Current step
phase: proposed | approved-writing | written-reporting | follow-up-pending
proposal: <one line>
checkpoint: <sha or HEAD>
untracked at checkpoint: <path hash> …
question asked: <one line, or none>

## Open items
- expired-token test skipped by user at step 2 — carry to recap

## Decisions
- step 2: repository pattern over inline queries — testability (user's pick)
```
