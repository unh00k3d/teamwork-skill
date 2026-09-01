# Case study: the first real mission

The protocol in this repo isn't a design sketch — most of its rules were written after a
team of Claude Code sessions ran a full build and hit the walls in person. This is that
mission: what it produced, what worked, and what broke badly enough to change the protocol.

The [`TEAM.md`](TEAM.md) beside this file is the real board, verbatim. It is the most
useful thing here — a roster, 45 tasks with their evidence, and a decision log the team
wrote for itself. Skim its Decisions section before anything else.

## The mission

> Build a self-hosted, locally-run fitness tracker web app for a single user. AI-generated
> training programs that adapt to logged performance; fast keyboard-driven set/rep/weight
> logging; body metrics with weight trend and body-fat estimate; cardio logging; progress
> charts; and an interactive muscle heatmap — click a muscle group to see its exercises, its
> recent volume, and whether it is lagging.

One sentence from the human, typed into an anchor session. No further instruction about how
to divide it up.

## What it produced

| | |
|---|---|
| Sessions | 11 across the mission (anchor, 2 leads, 8 members) |
| Board | 45 tasks |
| Code | 173 TypeScript files, ~29,000 lines |
| Surfaces | 10 pages, 20+ API routes, SQLite with hand-written migrations |
| Evidence | 197 files, 142 of them screenshots |
| Tool calls | 2,949 |
| Messages between sessions | 400 |

Every one of those 45 tasks was checked against a bar the lead set as its first act, and
cited its evidence file on the board before it could be marked done.

## What worked

**The board outlived every session that wrote it.** Partway through, a VM restart killed all
six tmux sessions at once. A respawned lead read `.team/`, reconstructed the roster, reopened
the dead members' claims and restaffed. Nothing was lost, because nothing important lived in
a context window.

**Members caught each other's defects.** The sharpest findings in the run came from one member
reading another's surface, not from the author's own tests:

- fit-design found that `/body` and `/cardio` typeset weights in a proportional face while
  identical numbers in the logging grid were mono and tabular — invisible on any single page.
- fit-log found that narrowing a kit `Select` stranded its dropdown chevron ~500px from its
  control. Every scripted assertion passed while it was on screen. It was caught by opening
  the PNG.

**The team distrusted its own green checks.** It ended up cataloguing six ways an
instrument lies — assertions that can't fail, checks that pass vacuously, setup that silently
no-ops — each one found by doubting a passing check rather than by a failing one. That
taxonomy is in the Decisions log and is the best writing the mission produced.

## What broke — and what it changed

| What happened | What it changed |
|---|---|
| Six sessions exhausted the account's usage quota in ~90 minutes. Two members then sat dead at the limit for **eight hours**, still marked `active`, still holding claims. The quota had reset at 3am with nobody awake to restart anything. | The §Liveness section: a usage limit *pauses* a session, it doesn't retire it. Plus quota-based team sizing, and the anchor scheduling its own wake at reset. |
| Five members ran to 350–470k of context. **Not one handoff happened all mission** — the trigger was "when your context runs low", which nobody can observe or act on. | A 200–300k band, checked at task boundaries, with a command that reads your own status line. |
| Every browser check queued behind one shared Chrome profile. One member eventually wrote its own headless driver and the bottleneck dissolved — but only for them, until the lead noticed. | §Shared resources: isolation over turn-taking, and instruments are shared property. |
| After the restart, peer-to-peer messaging fell from 31% of traffic to 9%. The new lead answered everything itself — and its shell calls outran its messages 4.2:1, versus 1.5:1 for the first lead. It started editing `src/` directly. | "Talk to each other, not through the lead", briefs that name your neighbours and their seams, and the lead-discipline rules: the lead's hands are `.team/`. |
| Eight scope corrections arrived mid-build. One feature was designed, built, kept for a documented reason, then deleted outright. A settings page — without which a promised feature couldn't produce a number — surfaced near the end. | An intake round with the human *before* the team is staffed. |
| Three consecutive polish rounds each discovered further polish, while the human waited on an app they hadn't seen. The anchor had to call a scope freeze the lead should have called. | The lead calls the finish, and `FOLLOW-UPS.md` catches everything found afterwards. |
| Six writers, one working tree, **no git repository at all**. | `git init` in scaffolding, and commit discipline in the protocol. |

## Caveats

This is a snapshot of a mission that was still running when it was taken, so the board has
open rows. It is one mission, in one domain, on one machine — the numbers are evidence, not
benchmarks. And it ran under an earlier version of the protocol: the rules it argued for are
in the current `PROTOCOL.md`, which means a rerun should fail differently.
