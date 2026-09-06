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

One Claude Code session is bounded by one context window and one pair of hands. Some jobs — a multi-surface app, a large port, a research sweep, a security review across many targets — are bigger than that. **teamwork** turns a single `/teamwork <mission>` into a team of real Claude Code sessions, each in its own tmux window, that split the mission between them, verify each other's work, hand off to successors when their context runs low, and write down what they learned for the next team.

There is no orchestrator process and no framework underneath. The whole thing is a markdown protocol that every session reads, a board file they all edit, and Claude Code's built-in session messaging for wakeups. You can `tmux attach` to any member at any time and watch it work.

## What happens when you run it

1. **Intake.** Your session becomes the **anchor**. Before anything is built it asks you the questions that would otherwise become rework: what is explicitly out of scope, what is optional, what must keep working with no key or network, which models the team may use, what permission mode members run in. The answers become the mission text.
2. **Scaffold.** The anchor makes sure the project is a git repo, then creates `.team/` — the board, the contract, the vault, and the folders for briefs, evidence and handoffs — and commits it.
3. **The lead.** The anchor spawns one session, the **lead**, and steps back. The lead's first act is to write the mission's **verification bar** — what "done" means, checkably — as the first Decision on the board. Then it breaks the mission into tasks, decides where the seams are, and writes the interfaces and file ownership into `CONTRACT.md`.
4. **Staffing.** The lead spawns members, up to six including itself, each on a model chosen for the shape of its work. Every member gets a brief naming its seam, its neighbours, and what each neighbour owns. A member's first act is to register on the roster and introduce itself to those neighbours.
5. **Work.** Members claim tasks on the board, build inside their seam, commit their own paths, and talk to each other directly — a question about the API goes to whoever owns the API, not through the lead. A task is done only when it meets the bar and cites its evidence file on the board.
6. **Handoffs.** When a member has spent most of its context — at a task boundary, not mid-work — it writes a handoff doc, promotes what it learned into the vault, spawns its own successor, and retires. The team outlives any single context window, the lead included.
7. **Acceptance.** The lead runs a whole-mission integration check against the bar and reports done. The anchor re-verifies independently, then shows you the result. Your feedback becomes board tasks; rounds repeat until you accept.
8. **Retrospective.** Before anyone is killed, every member writes what broke on its seam and what it would tell its replacement, and the anchor writes what only it could see from outside — who ran past the handoff band, who sat blocked, which rules went unheld. The lead promotes the transferable parts into `.team/vault/` and writes `RETRO.md`, including proposed amendments to the protocol itself, which the anchor puts to you.
9. **Teardown.** The tmux sessions are killed. `.team/` stays behind as the mission record, and the vault is what the next mission on this project starts from.

Throughout, you talk only to your own session. Ask it for status, change scope mid-flight, review at the end.

## The roles

| Role | What it does | What it deliberately doesn't do |
|---|---|---|
| **Anchor** | Your window. Runs intake, scaffolds `.team/`, spawns the lead, answers your status questions from the files, relays your steering, logs every member's context and liveness, re-verifies the result, writes the outside view in the retrospective. | Take a task, staff the team, or hold anything in its context that isn't also in `.team/` — so it lasts the whole mission, and can be replaced if it doesn't. |
| **Lead** | Sets the bar, splits the work, writes the contract, staffs the team, picks each member's model, unblocks, sweeps the roster every twenty minutes for dead sessions and spent context and orders handoffs, runs the integration check and the retrospective, calls the finish. | Edit the work. The lead's hands are `.team/`. A lead that starts fixing things itself has stopped leading — the protocol gives it a test for that. |
| **Member** | Claims tasks, builds inside its seam, verifies against the bar, commits its own paths, talks to its neighbours, hands off when spent. | Touch another member's files, answer a permission prompt on the human's behalf, or work on anything that isn't claimed on the board. |

## How they coordinate

Two channels, and only two:

- **`.team/TEAM.md`** — the single source of truth. A roster (who is alive, on what model, holding which claims), a task board (one row per task, edited in place), and a decision log of one-liners. Anyone can edit it; everyone re-reads it, which is why the protocol is strict about keeping it small — settled decisions rotate out, done rows collapse to a pointer.
- **Claude Code's session message bus** — `SendMessage` between sessions, for wakeups and questions only, never state. "Board updated: T7 done" is a message; what T7 was is on the board. Nobody polls anybody.

Around the board, `.team/` holds everything the team agrees, proves, and remembers:

```
.team/
├── PROTOCOL.md      the rules, copied in verbatim — every member reads it first
├── TEAM.md          Spawn / Models / Vault header lines, roster, board, decisions
├── CONTRACT.md      interfaces, schemas, file ownership — binding, changed only via the lead
├── DECISIONS.md     settled decisions rotated out of the board, so TEAM.md carries only what still binds
├── INTEGRATION.md   the finish line, written when the bar is set, not when the work ends
├── FOLLOW-UPS.md    everything found after the finish, waiting for you to triage
├── RETRO.md         what happened, what it changed, proposed protocol amendments
├── briefs/          how each member arrived
├── evidence/        how each task was checked — the board cites these
├── handoffs/        how each member left
├── retro/           each member's closing notes, plus the anchor's view from outside
└── vault/           memory across missions — survives when everything above is archived
    ├── INDEX.md     one line per entry; members read this, then open what touches their seam
    ├── notes/       one lesson, pattern, or codebase fact per file, with what it was verified against
    └── instruments/ reusable scripts — drivers, probes, fixtures — each with a README saying what it proves
```

## The rules that make it hold

The full ruleset is [`PROTOCOL.md`](skills/teamwork/PROTOCOL.md). It is short, and most of its rules were paid for — see [the case study](#a-real-mission). The load-bearing ones:

- **Claim before you work.** One owner per task. Work without a claim on the board doesn't exist. A finding belongs to whoever owns the surface it lives in, the moment it's reported.
- **Done means checked against the bar**, with the evidence cited on the board — not "looks finished". The bar is set first and it is domain-shaped: green tests and a production build for code, source-cited coverage for research, a probed-everything checklist for security.
- **Commit your own paths, explicitly.** Six writers share one working tree; `git add -A` sweeps up someone's half-edit. A task isn't done until it's committed. Nobody rewrites shared history.
- **Talk sideways.** Members message the owner of what they need, and the lead hears about it only when the answer moves the board or the contract. A team routed through its lead is a star, and stars fail at the centre. A sent message is checked as delivered, and an escalation obliges a reply.
- **Prefer isolation over turn-taking.** Your own port, your own browser profile, your own copy of the database. A team that queues to verify runs one member at a time. And `.team/` is a coordination surface, never a build input — exclude it from anything that traces or bundles the tree.
- **Hand off while there is still room to write a good handoff.** The band is 200–300k of a 1M window, and a working member gets there in about twenty minutes. Members check at task boundaries and before long tool sequences; past the top of the band the lead orders it, because a member with warm context is the worst-placed session to call it on itself.
- **Silence isn't progress.** A session stops four ways — blocked on an unanswered question, usage limit, killed, handoff — and only one is planned. The lead sweeps the roster with `tmux ls` and `capture-pane` on a cadence, and the anchor keeps a liveness log of every member's context, rather than anyone assuming.
- **The lead calls the finish.** Polish rounds don't terminate on their own. Anything found after the finish goes to `FOLLOW-UPS.md` for you to triage, and whether another round is worth it is your call.

## Memory across missions

Everything above is scoped to one mission and archived when the next one starts — except the **vault**. `.team/vault/` is the project's memory: short indexed notes and reusable scripts that every member reads before it starts, so a lesson learned or a probe written once is never rebuilt.

It fills by promotion, not by free writing. Two moments put things in: the *For the next mission* section of every handoff, and the retrospective at the close. Each entry carries what it was verified against, because it is a claim by a session that is gone; a member that finds one wrong corrects it. It is plain markdown with `[[wikilinks]]`, so an Obsidian vault opens it as-is. A `Vault:` line in `TEAM.md` can point at a second, global vault for lessons that cross projects.

The retrospective also produces **proposed protocol amendments** — rules the mission had to learn mid-flight, phrased as the line `PROTOCOL.md` should carry. They go to you, not into the protocol; the first mission's amendments are how most of the current protocol got written.

## Models and quota

Every member spends the same account quota, and models differ several-fold in what they cost per hour of work. So the team has two sizing levers: how many members, and which model each one runs. The lead picks a model per role within the `Models:` line you set at intake — the strongest for judgment (the lead itself, design, auditing another member's surface, anything where the contract is still being discovered) and a cheaper one for well-specified work against a settled contract (porting, wiring a route to a schema, evidence re-capture). The choice is recorded on the roster so you can see the mix.

When the quota runs out, the whole team stops at once. The sessions survive in tmux and resume after the reset, and the anchor schedules its own wake-up call for the reset time so nobody has to be awake for it.

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

Answer the intake questions, then watch it go. Useful things to know while a mission runs:

- Ask the anchor for status any time — it answers from `TEAM.md` and the handoff docs without interrupting anyone.
- Steering ("actually, skip the admin routes") goes through the anchor too. The lead re-plans the board.
- `tmux ls` shows every member as `team-<slug>-<role>`; `tmux attach -t team-<slug>-lead` drops you into any of them, `Ctrl+b d` detaches. Attaching is also how you answer a permission prompt if you chose a stricter mode.
- Three header lines in `.team/TEAM.md` configure the whole team and can be changed mid-mission: `Spawn:` (the command each member runs), `Models:` (which models the lead may use), `Vault:` (where the memory lives).
- When the lead reports done, the anchor re-verifies before showing you. Your feedback becomes board tasks. When you accept, the retrospective runs, then teardown.

## Permissions — read this part

By default, team members run with `--dangerously-skip-permissions`: every tool call auto-approves, so six sessions can work in parallel without stalling on prompts nobody is watching. The protocol tells members to keep the blast radius small (stay in the mission's directory, route anything destructive or outward-facing through you), but the flag means what it says. Run missions in a repo you can restore, or on a machine you don't mind an agent having.

If that trade isn't for you, say so at intake: the anchor writes the spawn command onto the `Spawn:` line in `.team/TEAM.md`, every member reads it from there, and a member that hits a prompt it can't pass marks the task blocked and moves on until you answer the prompt in its tmux session.

## Requirements

- Claude Code with skills and session messaging (`ListAgents` / `SendMessage`) — any recent 2.x
- tmux
- git — the mission runs in a repo, and the anchor will `git init` one if needed
- Usage headroom. A six-session team burns quota several times faster than you're used to; see [Models and quota](#models-and-quota).
- One machine, one filesystem — members coordinate through files, so this doesn't span hosts

## A real mission

The protocol's rules mostly aren't guesses — they were written after a team of eleven
sessions built a 29,000-line app across 45 board tasks, survived a VM restart, and failed in
several ways worth documenting. [**The case study**](case-study/) has the numbers, what
worked, and what broke, alongside the mission's real `TEAM.md`. A second mission has since
added its own lessons — on board size, message delivery, and how fast a member reaches the
handoff band — and they are in the protocol with their numbers.

## Why this shape

Most multi-agent setups put an orchestrator in charge and make every agent report up through it. That orchestrator becomes the bottleneck and the single context window that dies first. Here the anchor deliberately stays out of the work precisely so its context lasts the whole mission, and the state lives in files, so any member — lead included — can be replaced mid-mission without losing the plot. The vault extends the same idea across missions: the next team starts from what this one learned, not from zero.

## Roadmap

Roughly in order:

- [x] **Configurable spawn command.** The command lives on the `Spawn:` line of `.team/TEAM.md`, set once per mission by the anchor.
- [x] **A vault across missions.** `.team/vault/` persists; handoffs and a closing retrospective promote lessons and instruments into it, and every member reads its index first.
- [x] **Per-member models.** The lead chooses a model per role within the `Models:` line, recorded on the roster.
- [ ] **Scripts for the mechanical parts.** Spawn, roster sweep, and quota wake-up are prose in the protocol today; a script can't forget the second `send-keys`.
- [ ] **Agent-agnostic protocol.** The board is already just a markdown file any agent can edit. What's Claude Code-only is the wakeup channel (`ListAgents`/`SendMessage`) and the spawn command — needs a file-based wakeup fallback for agents without session messaging, then testing against other agents that read skill folders (Codex, Cursor, OpenCode).
- [ ] **Mixed teams.** Once the protocol is agent-agnostic: a Codex member and a Claude Code member claiming tasks off the same board.
- [ ] **An asciinema recording.** The written case study is in [`case-study/`](case-study/) with the real `TEAM.md`; what's still missing is a screen recording of a mission actually running.
- [ ] **Windows.** Everything assumes tmux. WSL works today; native Windows needs a different session substrate.

Ideas and PRs welcome — open an issue if one of these matters to you.

## License

MIT
