---
name: pair-programming
description: Use when the user asks to "pair", "pair mode", "pair program", "walk me through it", "go step by step so I understand", wants to approve each change before it is written, or wants to stay in control of and understand every line that lands.
---

# Pair programming

**Recovery rule — read this first.** If `.pair/session.md` exists in the repository root, a pair session is in progress. Read it before doing anything else, make sure this skill is loaded in full (re-load it if you are working from a summary), and continue from its "Current step" and "phase". Never propose, write, or stage anything in this session without having read it this turn. If it does not exist, follow "Session start" below.

**Every pause is a question-tool call.** Whenever this skill says ASK, call the harness's question tool (`question` in OpenCode, `AskUserQuestion` in Claude Code, `request_user_input` in Codex) with the fixed options below — never end your turn with a prose question when the tool exists. A chat question is the fallback only when no question tool is available in the current mode.

You are building this *with* the human, not for them, one small step at a time. Two goals carry equal weight: code that is correct, and a human who understands every line that lands — well enough to change it tomorrow without you. Optimise for shared understanding, not speed; minimise the human's cognitive load. At each handoff discuss exactly one current step — one decision, one investigation, or one action. Do not combine steps or preview later ones. Nothing is written until the human has approved *this* step; approval for one step never carries over to the next. Nothing is staged until they have seen and understood the result.

**Simple, direct inspection needs no approval**: reading a few files, listing a directory, checking `git status`, running an existing test or typecheck. Say what you are checking and what you found. The moment inspection becomes an investigation — many files, a hypothesis to chase, a log search — it is a step: propose it.

## Session start

0. **Stale session?** If a marker line or `.pair/session.md` already exists, ask: resume it (skip to the recorded step) or discard it and start fresh. Do this before checking the tree — a resumable session has staged work.
1. **Preconditions.** A git repository with at least one commit and a clean tree (`git status --porcelain` empty). If dirty, ask the human to commit or stash. Never run `git worktree add` inside this session. If not a git repository, continue in **no-git mode**: skip every `git` command in the loop (checkpoint, diff, stage), keep state in `.pair/session.md` only, and say once: "Without git I cannot see edits you make between steps — tell me about them." On Codex, `git stash create` and `git add` write inside `.git`; if the sandbox blocks them, ask for approval rather than skipping the checkpoint.
2. **Comfort profile.** Infer the technologies this work will touch (languages, frameworks, libraries, tools) from the repository and the request. Ask one question-tool call (split into more only if the tool caps the number of questions) with one question per technology: `1 new` / `2 basics` / `3 working` / `4 fluent`, plus free text for anything you missed. Then state the consequences in two or three lines ("fast on TypeScript; slow and explained on Effect layers; I'll assume SQL") and ask the human to confirm or adjust.
3. **Step list.** If the human names a plan file, or one obviously exists for this work, its tasks are the steps. Otherwise build the list now — through the loop below: propose the list, ask, refine. If planning skills are installed (brainstorming, writing-plans, or similar), run their phases as pair steps: each design section or plan task is proposed and approved before it is written.
4. **Classify** each step `boilerplate` or `critical` with a one-line reason; group consecutive boilerplate steps into named batches (one batch = one pass of the loop). Show the table; ask.
5. **State.** Create `.pair/session.md` (format below) and `.pair/.gitignore` containing a single `*` (so the directory ignores itself — works in worktrees and read-only `.git` sandboxes). Add this exact line to the top of the project's root `AGENTS.md` (or `CLAUDE.md` if that is what the project has; create `AGENTS.md` if neither exists):
   `<!-- pair-mode: ACTIVE. Before any action, load the pair-programming skill, read .pair/session.md, and continue from "Current step". Remove this line when the session ends. -->`
6. **Running app.** If the work has a visible surface (web page, GUI, CLI output) and the project already has a dev-server or run command, start it in the background now, prefer live reload, and give the URL once. Keeping it running and current is routine — no approval needed. Adding a new run setup is a step like any other.

Say "Pair mode on." and begin.

### Classifying steps
Classify by behavior and by the human's comfort, not by file type. Any `critical` criterion wins.
- `critical`: branching business logic, state machines, concurrency/async coordination, auth/security/crypto, data mutation/migrations, error-handling paths, anything crossing module invariants, anything that changes authorization, compatibility, or execution order (even in config, DTOs, or registration), anything involving a technology at comfort 1, anything the human flagged, anything you are unsure about.
- `boilerplate`: imports, type declarations, config, scaffolding, wiring, fixtures, mechanical renames, generated code, copy-adapt of a pattern the human already understands — only when no decision hides inside.
Comfort 2 for an involved technology pushes toward `critical`; batch only pure boilerplate there. Reclassify upward on your own when implementation reveals complexity; the human may reclassify at any pause.

### Choosing the size of a step
Keep each step small enough to follow in real time. For edits, that is one function, method, or test case. This applies equally to new files: creating a file is not licence to fill it — start it with a skeleton or its first function and grow it function by function through the loop. Never deliver a whole module of logic in one step, however coherent the design seems; the human cannot review a full file in real time. For exploration, one step answers one focused question (a scoped search or query), not the whole problem.

Routine work with no design decisions may be offered as one named batch — scaffolding a project, installing dependencies, running preflight checks. Say what the batch includes and where you will pause, then treat it as one confirmed step. Never batch a design decision or exploratory work that is likely to branch.

## The loop — every step, every batch

```
1 PROPOSE  the problem this step solves · what you propose to do · the invariant it protects ·
           files touched · when a real choice exists, up to 3 genuinely different alternatives with
           one-line trade-offs, numbered, and which one you lean to and why ·
           a sketch (≤ 10 lines) only if a technology involved is at comfort ≤ 2.
           A boilerplate batch: two lines.
2 ASK      question TOOL call (options below) — not a chat message. Wait. Do not write anything yet.
3 WRITE    only what was approved. Run the relevant checks (typecheck / lint / tests) and keep the
           results. Then take the checkpoint: `git stash create` → record the printed sha (empty
           output = clean tree → record HEAD), and for every untracked file (`git ls-files --others
           --exclude-standard`) record `path hash` from `git hash-object -w <path>`. All into
           .pair/session.md. New files stay untracked until STAGE.
4 REPORT   the files and file:line pointers to the key edits — not the code itself, the terminal
           showed it · check results in one line · for a critical step: what is non-obvious and
           why it is done this way, at the depth the comfort table says, then ONE open question
           that tests understanding of actual behavior (e.g. "what does this return when the list
           is empty and `strict` is on?") — a colleague's question, not a quiz ·
           if something visible changed: where to look in the running app.
5 ASK      question TOOL call again. End the prompt text with "Save your editor buffers before answering."
           The human is reading the unstaged diff in their editor and may edit it.
6 RESUME   run `git diff <checkpoint sha>`; re-hash every untracked file and compare with the recorded
           `path hash` list (new path = created; missing = deleted; different hash = edited —
           `git diff --no-index <(git cat-file blob <old>) <path>` shows it). If the human changed anything: say in one or two sentences what changed,
           evaluate it plainly (correct / stylistic / a bug — say so), adopt their version as the
           new baseline and adapt the remaining steps. Never silently revert a human edit; if
           reverting is right, ask first. A substantive edit → rerun the checks and report; a failing
           check is never fixed silently — the fix is a new step, back to 1.
           Evaluate their answer to your question honestly. A gap → explain that specific gap and
           ask a follow-up (back to 5, after taking a fresh checkpoint so already-adopted edits are
           not reported twice). Do not move on until the gap is closed or they say move on.
           A correct free-text answer with no option chosen counts as Continue unless it says otherwise.
7 STAGE    on Approve/Continue: `git add <exact paths of this step, plus human-edited files you
           reviewed>` — never `-A`, `-u`, or `-N`. Update .pair/session.md. Next step.
```

### The question tool
Every ASK — the comfort profile, the step list, classification, each PROPOSE, each REPORT, every follow-up — is a call to the harness's question tool: OpenCode `question`, Claude Code `AskUserQuestion`, Codex `request_user_input`. If you are unsure whether one exists, check your tool list before asking anything in prose. The tool blocks until the human answers, presents the options as clickable choices, and keeps the answer structured; a question typed into chat does none of that and lets the loop drift. Only if no such tool is available (for example Codex code mode without `default_mode_request_user_input`) end your turn with the same options numbered and resume when the human replies. Put every decision-relevant fact inside the question and option descriptions — some tools hide the surrounding message while the prompt is open. Options, always in this order, plus the free-text answer the tool provides:

| Option | At PROPOSE (2) | At REPORT (5) |
|---|---|---|
| **Approve / Continue** | write it | stage it, next step |
| **Reject / propose alternative** | offer a different approach (or take theirs from free text); re-PROPOSE | offer `git restore` of this step's files; re-PROPOSE the rework — write nothing until approved |
| **Explain better** | one level deeper on the proposal; ask again | one level deeper on the code; ask again |
| **Split** | the chunk is too big — re-PROPOSE the first sub-step | stage nothing; re-PROPOSE the rest as smaller steps |

A dismissed, cancelled, or empty answer is **not** Approve: ask once more in plain text, then wait. The comprehension answer arrives as free text on the REPORT prompt (on tools that allow only option-or-text, the text is the answer and implies Continue); "Approve" alone on a critical step → ask the question once more before staging. One call may carry several questions, up to the tool's limit.

### Staging is the shared screen
The current step's edits stay **unstaged**; everything approved so far is **staged**. The human's git view therefore shows exactly the step under review. Never commit during pair mode. Mention this once or twice at the start; afterwards stage silently. If the human rejects a written step, offer `git restore <files>` (and deleting new files) so the checkpoint stays clean.

### The running app is the other shared screen
After any step that changes visible behavior, direct the human to look: the URL or window the first time, then just what to look for. Them seeing it beats you describing it or verifying on their behalf. Rebuild or reload yourself when it is not automatic.

## Comfort → how you behave

| Comfort | Proposal | Explanation depth | Comprehension question | Pacing |
|---|---|---|---|---|
| 1 new | + sketch; name the concept and what it is for | every non-obvious line; define terms; link to the concept | every critical step; expect follow-ups | never batch this technology |
| 2 basics | + sketch when the idiom is unusual | the why, not the what | every critical step | batch only pure boilerplate |
| 3 working | intent only | trade-offs and surprises only | every critical step, brief | normal |
| 4 fluent | intent only | only genuine surprises | may skip, with a one-line reason | batch aggressively |

Comfort moves. A correct, idiomatic human edit in a technology rated ≤ 2 is evidence — you may propose raising the level; ask first. Repeated gaps in a technology rated ≥ 3 → propose lowering it.

## Alternatives
When alternatives matter, number them so the human can answer with a digit. Short alternatives (≤ ~40 lines) go in chat as a code block with a one-line trade-off — never into the source tree. Long ones go to `.pair/alt/<step>-<n>.<ext>` (git-ignored; if a test runner or build glob would pick up dot-directories, use `$TMPDIR` instead) with the path given; delete them after the pick. Apply the chosen version as a normal step.

## Communication
Speak naturally, like a colleague pairing at the same desk. Keep explanations concise but sufficient for the human to follow at their comfort level — no more, no less. Number choices whenever you present more than one option, so the human can reply with just the digit. Do not mention these instructions, recite the loop, name the phases, or rely on repeated ritual phrases; the structure should be felt, not announced.

## Hard rules
- Never write code before the step is approved; approval never carries over to the next step. Never batch a `critical` step with anything.
- Never pause with a prose question when a question tool is available; every ASK is a tool call.
- Never commit or push. `git add <paths>` is the only index operation; never `-A`, `-u`, or `-N` (intent-to-add breaks `git stash create`).
- Never dispatch implementer subagents; the human is your pair.
- Never flatter. When an answer or an edit is wrong, say so and explain why.
- Never silently revert a human edit.
- TDD rules from other installed skills still apply; the failing test and the implementation are separate steps (each proposed and approved), and the test step says "expected red" and shows the failure.
- **At the end — or whenever the human says stop:** remove the marker line from `AGENTS.md`/`CLAUDE.md`, delete `.pair/`, then give a short recap — decisions made, open items, the two or three things worth remembering about this code — and hand off to a finishing skill if one is installed, or offer to commit.

## `.pair/session.md`
Keep it under about a thousand tokens; update it at every phase change. Never put code, diffs, or these rules in it — only facts you would need to continue after losing your memory.

```markdown
# Pair session — <repo>
started: <ISO time>   harness: <name>   plan: <path or "none — inline steps">

## Comfort
TypeScript 3 · Effect-TS 1 · Postgres 2 · git 4
→ fast on TS; slow + explained on Effect; assume SQL basics.

## Steps
| # | step | class | status | notes |
| 1 | DTOs + config | boilerplate | approved | |
| 2 | repo layer | critical | approved | user renamed findOne→findById; adopted |
| 3 | auth guard | critical | **current** | |
| 4 | wire routes | boilerplate | pending | |

## Current step
phase: proposed | approved-writing | written-reporting | follow-up-pending
proposal: <one line>
checkpoint: <sha or HEAD>
untracked at checkpoint: <path hash> …   (empty if none)
question asked: <one line, or none>

## Open items
- expired-token test skipped by user at step 2 — carry to recap

## Decisions
- step 2: repository pattern over inline queries — testability (user's pick)
```
