<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" alt="teamwork" width="400">
  </picture>
</p>

<p align="center">A Claude Code skill for running big jobs with a team of sessions instead of one.</p>

---

<p align="center">
  <img src="assets/how-it-works.svg" alt="You steer the anchor session; the anchor spawns a lead in tmux; the lead staffs up to six members; everyone coordinates through the TEAM.md board" width="720">
</p>

You type `/teamwork <mission>`. Your session becomes the **anchor** — your window into the team. It sets up a shared workspace, spawns a **lead** session in tmux, and steps back. The lead breaks the mission into tasks, staffs a team of up to six sessions, and drives the work against a verification bar it sets before anything else. You keep talking to your own session the whole time: ask for status, change scope mid-flight, review the result at the end.

There's no orchestrator process and no framework underneath. Coordination is two things:

- **`.team/TEAM.md`** — the single source of truth. A roster, a task board, and a decision log, edited in place by whoever is working.
- **Claude Code's session message bus** — wakeups and questions only, never state. Nobody polls anybody, and members talk to each other directly rather than routing everything through the lead.

Alongside the board, `.team/` collects what the team agrees and proves: `CONTRACT.md` for interfaces and file ownership, `INTEGRATION.md` for the finish line, `evidence/` for how each task was checked, `briefs/` and `handoffs/` for how members arrive and leave.

A few rules make it hold together: members claim a task on the board before touching anything, one owner per task; "done" means checked against the mission's verification bar with the evidence noted on the board, not "looks finished"; and when a member has spent most of its context — at a task boundary, not mid-work — it writes a handoff doc and spawns its own successor, so the team outlives any single context window. The full ruleset is in [`PROTOCOL.md`](skills/teamwork/PROTOCOL.md) — it's short, and every member reads it first.

## Install

With the [skills CLI](https://skills.sh):

```sh
npx skills add unh00k3d/teamwork-skill       # this project
npx skills add unh00k3d/teamwork-skill -g    # everywhere
```

The CLI can install skill folders for other agents too, but the protocol currently assumes Claude Code — it leans on session messaging (`ListAgents`/`SendMessage`) and the `claude` CLI in its spawn pattern. Making it agent-agnostic is on the [roadmap](#roadmap).

As a Claude Code plugin:

```sh
claude plugin marketplace add unh00k3d/teamwork-skill
claude plugin install teamwork@teamwork-skill
```

Or just copy the folder:

```sh
git clone https://github.com/unh00k3d/teamwork-skill
cp -r teamwork-skill/skills/teamwork ~/.claude/skills/
```

Copy into `.claude/skills/` inside a project instead if you only want it there.

## Use

From a Claude Code session in your project root:

```
/teamwork port the API layer to the new client and get the integration suite green
```

<p align="center">
  <img src="assets/terminal.svg" alt="A mission starting: the skill scaffolds .team/, spawns the lead, and tmux ls shows one session per member" width="680">
</p>

Then watch it go. Useful things to know while a mission runs:

- Ask the anchor for status any time — it answers from `TEAM.md` and the handoff docs without interrupting anyone.
- Steering ("actually, skip the admin routes") goes through the anchor too. The lead re-plans the board.
- Every member is a real Claude Code session in its own tmux session. `tmux ls` shows them as `team-<slug>-<role>`, and `tmux attach -t team-<slug>-lead` drops you into any of them. Detach with `Ctrl+b d`.
- When the lead reports done, the anchor re-verifies independently before presenting the result to you. Feedback loops back through the board until you accept, and teardown kills the tmux sessions. `.team/` stays behind as the mission record.

## Permissions — read this part

By default, team members run with `--dangerously-skip-permissions`: every tool call auto-approves, so six sessions can work in parallel without stalling on prompts nobody is watching. The protocol tells members to keep the blast radius small (stay in the mission's directory, route anything destructive or outward-facing through you), but the flag means what it says. Run missions in a repo you can restore, or on a machine you don't mind an agent having.

If that trade isn't for you, edit the spawn pattern in `PROTOCOL.md` to a stricter flag before installing. The protocol already covers that mode: a member that hits a prompt it can't pass marks the task blocked and moves on, and you answer the prompt by attaching to its tmux session.

## Requirements

- Claude Code with skills and session messaging (`ListAgents` / `SendMessage`) — any recent 2.x
- tmux
- Usage headroom. Every member spends the same account quota, so a six-session team burns it several times faster than you're used to. When it runs out the whole team stops at once; the sessions survive and resume after the reset, but nothing moves until then. Size the team accordingly — the skill tells the lead to.
- One machine, one filesystem — members coordinate through files, so this doesn't span hosts

## Why this shape

Most multi-agent setups put an orchestrator in charge and make every agent report up through it. That orchestrator becomes the bottleneck and the single context window that dies first. Here the anchor deliberately stays out of the work precisely so its context lasts the whole mission, and the state lives in a file, so any member — lead included — can be replaced mid-mission without losing the plot.

## Roadmap

Roughly in order:

- [ ] **Configurable spawn command.** The spawn pattern hardcodes `claude --dangerously-skip-permissions`. Make the command and flags a setting so a stricter permission mode doesn't mean hand-editing `PROTOCOL.md`.
- [ ] **Agent-agnostic protocol.** The board is already just a markdown file any agent can edit. What's Claude Code-only is the wakeup channel (`ListAgents`/`SendMessage`) and the spawn command — needs a file-based wakeup fallback for agents without session messaging, then testing against other agents that read skill folders (Codex, Cursor, OpenCode).
- [ ] **Mixed teams.** Once the protocol is agent-agnostic: a Codex member and a Claude Code member claiming tasks off the same board.
- [ ] **Real mission transcripts.** Replace the illustrations with a recorded mission — asciinema plus the final `TEAM.md` as a case study.
- [ ] **Windows.** Everything assumes tmux. WSL works today; native Windows needs a different session substrate.

Ideas and PRs welcome — open an issue if one of these matters to you.

## License

MIT
