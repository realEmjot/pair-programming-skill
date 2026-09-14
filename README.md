# pair-programming

An agent skill that makes a coding agent work *with* you instead of *for* you: it proposes each small step, waits for your approval, writes only that, shows you what it did, checks that you understood, and notices when you edit the result yourself. Pace and depth follow how comfortable you said you are with each technology.

Zero hooks, zero plugins, zero runtime code — one `SKILL.md`. Works in Claude Code, Codex, OpenCode, and any harness that reads agent skills and has a question tool.

## Install

```sh
npx skills add realEmjot/pair-programming-skill      # any supported harness
```

or clone into your harness's skills directory:

| Harness | Directory |
|---|---|
| Claude Code | `~/.claude/skills/pair-programming` |
| Codex | `~/.agents/skills/pair-programming` |
| OpenCode | `~/.config/opencode/skills/pair-programming` |

```sh
git clone https://github.com/realEmjot/pair-programming-skill /tmp/pps && cp -r /tmp/pps/skills/pair-programming <dir>
```

Restart the harness. Say "let's pair on X" or invoke the skill directly.

**Codex:** the skill uses `request_user_input` at every pause. Outside plan mode it needs `default_mode_request_user_input = true` in `~/.codex/config.toml`; without it the skill falls back to ending its turn with numbered options. The checkpoint (`git stash create`, `git hash-object -w`) and staging write inside `.git`, which the `workspace-write` sandbox mounts read-only — expect approval prompts for those, or grant `.git` write access.

## What a session looks like

1. **Comfort check.** The agent lists the technologies the work will touch and asks how comfortable you are with each (1 new → 4 fluent). Unfamiliar tech gets slow, explained steps; fluent tech gets batched.
2. **Step list.** Built with you, or taken from an existing plan. Each step is `boilerplate` (batched) or `critical` (one concept at a time).
3. **Per step:** propose → you approve/reject/split/ask → agent writes → reports with file:line pointers and, for critical steps, one real question about the behavior → you review the unstaged diff in your editor, edit it if you like → agent reads your edits, adopts them, answers, stages.
4. **Shared screens:** your git view (staged = approved, unstaged = the current step) and, if the work has a UI, the running app.
5. **End:** a short recap of decisions, open items, and what to remember. Nothing is ever committed by the agent.

## What it writes into your repo

- `.pair/` — the session state (`session.md`: comfort profile, steps, current phase, checkpoint) plus a self-ignoring `.gitignore`, so nothing in it is ever tracked. Deleted at the end.
- One marker line at the top of your root `AGENTS.md` (or `CLAUDE.md`):
  `<!-- pair-mode: ACTIVE. Before any action, load the pair-programming skill, read .pair/session.md, and continue from "Current step". Remove this line when the session ends. -->`
  Removed at the end. This is what lets the agent resume correctly after a context compaction on every harness: root instruction files are re-read from disk, skill bodies and summaries are not reliable.

If a session dies, you may find a stale marker and `.pair/` — the agent will ask whether to resume or discard on the next start; deleting them by hand is also fine.

## Credits

The approve-before-write loop, git-index-as-shared-screen, running-app screen, and step-sizing rules are adapted from [martinpllu/pair-mode](https://github.com/martinpllu/pair-mode). Design history (including a deleted v1 that used a plugin) is in `docs/`.

## License

MIT
