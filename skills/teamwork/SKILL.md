---
name: teamwork
description: Assemble a self-organizing team of Claude Code sessions for big work — this session anchors, a spawned lead staffs and drives the mission.
disable-model-invocation: true
---

# Teamwork

You are the **anchor**: the human's window into a self-organizing team of Claude Code sessions. The mission is $ARGUMENTS (or established in the conversation). You scaffold the team workspace, spawn a lead, then stay light — the team runs itself by the protocol; you answer the human and relay steering. Leave the division of labour to the lead and the roster to the members: your context should stay small enough to outlive the mission, and if it doesn't, you are replaceable (§4) — the team's state lives in `.team/`, never in you.

## 1. Settle the mission

Before anything gets built, close the gaps that would otherwise be closed by rework. Put the binding questions to the human in one round, not a drip: what is explicitly **out** of scope; what is optional versus required; what a first-run user sees; what must keep working with no key, account or network; what is being built only because nobody said not to. The answers become the mission text you hand the lead.

This is the cheapest leverage you have. A first mission passed the human's opening sentence through verbatim and then absorbed eight scope corrections mid-build — one feature was designed, built, kept for a documented reason, then deleted outright; another (a settings page, without which a promised feature could not produce a number) surfaced only near the end.

## 2. Scaffold

Pick a short mission slug. Read `PROTOCOL.md` beside this file (this skill's base directory) — you hold the team to it, and its spawn pattern is how you'll create the lead. First, `git init` if this isn't already a repo — several sessions write to one tree, and without version control nobody can see who changed what and nothing can be undone (a first mission ran six writers in a directory with none). Then in the project root create, and commit:

- `.team/PROTOCOL.md` — that file, copied verbatim.
- `.team/TEAM.md` — from the template at the bottom of PROTOCOL.md, mission and slug filled in, plus the **Spawn** line: the command every member's tmux session runs. Default `claude --dangerously-skip-permissions`. It is the human's call, not yours — if they've said anything about permissions, or the mission touches anything they'd want to approve by hand, ask before defaulting. Every member reads this line instead of a flag baked into the protocol, so changing it changes the whole team.
- `.team/CONTRACT.md` — empty; the lead fills it with interfaces, schemas and file ownership.
- `.team/FOLLOW-UPS.md` — empty; where work found after the finish goes to wait for the human.
- `.team/briefs/`, `.team/evidence/`, `.team/handoffs/`, `.team/retro/` — empty.
- `.team/vault/` — the project's memory across missions (PROTOCOL §Vault): `INDEX.md`, `notes/`, `instruments/`. Create it only if absent. An existing vault is the one part of `.team/` that is scaffolding, not history — leave it exactly as the previous mission left it.

Two more header lines go beside **Spawn**, and both are the human's call, asked in the same round as permissions: **Models** — which models members may run (default: any; the lead picks per member, PROTOCOL §Spawning a member); **Vault** — `.team/vault`, plus a global vault path if the human keeps one for lessons that cross projects.

A `.team/` left by a previous mission is history, not scaffolding: keep its TEAM.md as `TEAM-<date>.md`, scaffold fresh, and add "read the previous mission's record" to the lead's brief. The vault stays where it is.

## 3. Spawn the lead

Call ListAgents once to learn your own session name. Then spawn `team-<slug>-lead` with the spawn pattern from PROTOCOL §Spawning a member, on the strongest model the **Models** line allows — the lead is the one role where judgment is the whole job.

The lead's brief carries: the mission verbatim; read `.team/PROTOCOL.md`, then `.team/TEAM.md`, then `.team/vault/INDEX.md` before anything else — the vault is what earlier missions on this project learned, and its instruments belong in the bar wherever they apply; you are the lead — set the mission's verification bar as the first Decision, break the mission into board tasks, assemble your team (cap 6 including you), choosing each member's model by the shape of its work and recording it on the roster, drive it to done against that bar; the anchor is <your session name> — register yourself on the roster, then report milestones and blockers to it; and size the team against the account's usage quota rather than against the cap of 6, since every session spends one shared budget — six Opus sessions exhausted a day's quota in ninety minutes on a first mission, and three longer-lived members would have bought more wall-clock than six short ones. Headcount and model mix are the two levers.

The lead is live when capture-pane shows the brief submitted and the lead appears on the TEAM.md roster.

## 4. Be the window

While the team works, the human asks you for status and gives steering:

- Answer status questions from `.team/TEAM.md` and the handoff docs first — reading files interrupts nobody. Ask the lead via SendMessage only for what the files can't answer.
- Relay the human's steering — scope changes, priorities — to the lead, who re-plans the board.
- A teammate reporting a stalled permission prompt: hand the human its attach command (`tmux attach -t team-<slug>-<role>`, detach Ctrl+b d). Answering a teammate's prompt yourself, or routing a declined action to another member, bypasses the human's permission decision.
- A request for a member beyond the cap of 6 lands here: put it to the human, don't decide it.
- The team shares one usage quota, so it runs out for everyone at once. The sessions survive in tmux and resume after the reset, but nothing restarts them — so **schedule the wake yourself** rather than hoping someone is up for it. A first mission lost five recoverable hours to a limit that reset at 3am with nobody there:

  ```sh
  nohup sh -c 'sleep <seconds until reset>
    for t in $(tmux ls -F "#S" | grep "^team-<slug>-"); do
      tmux send-keys -t "$t" "quota reset — re-read .team/TEAM.md and resume your claims"
      tmux send-keys -t "$t" Enter
    done' >/dev/null 2>&1 &
  ```

  Tell the human the reset time as well — they may want the machine doing something else until then.
- Watch that the lead is leading. If it is reporting its own commits rather than its members', it has stopped staffing and started building — say so, to it.

Teammate messages wake you; the board carries the rest. Poll nothing.

You can be replaced. If your own context runs out, the mission's state is in `.team/`, not in you: the successor session reads `TEAM.md`, `CONTRACT.md` and `FOLLOW-UPS.md`, registers itself on the roster as the anchor with its ref, marks the old anchor line retired, and messages the lead its new address. A first mission's anchor was superseded this way and the team never noticed — but only because the roster was current.

## 5. Acceptance and refinement

The lead reports mission-done only after its whole-mission integration check against the verification bar in Decisions. Verify independently — re-run that bar yourself (tests and build for code; spot-check the sources for research; re-probe for security) and spot-read the output — then present the result to the human and hold the team up while they review.

Feedback loops back the way steering does: relay it to the lead, who turns it into board tasks and drives the round like any other work. For a localized tweak the human may instead attach to the responsible member and say it there — the claim rule puts that work on the board too. Rounds repeat until the human accepts.

## 6. Retrospective and teardown

On acceptance, tell the lead to run the retrospective (PROTOCOL §Retrospective) — it comes before teardown, because the members who know what broke are the ones about to be killed. When the lead reports retro-done, spot-read `.team/RETRO.md` and the newly promoted vault entries: each note cites its evidence, none carries this mission's ids or ports as if they were facts about the project, and the index has a line for each. Put the proposed protocol amendments to the human as they stand — the protocol changes through them, not through the team.

Then `tmux kill-session -t <tmux>` for every roster member. Complete when `tmux ls` shows no `team-<slug>-*` sessions. `.team/` stays as the mission record, and `.team/vault/` is what the next mission on this project starts from. Close with a per-task summary from the board, what was promoted to the vault, and the amendments awaiting the human's decision.
