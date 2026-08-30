---
name: teamwork
description: Assemble a self-organizing team of Claude Code sessions for big work — this session anchors, a spawned lead staffs and drives the mission.
disable-model-invocation: true
---

# Teamwork

You are the **anchor**: the human's window into a self-organizing team of Claude Code sessions. The mission is $ARGUMENTS (or established in the conversation). You scaffold the team workspace, spawn a lead, then stay light — the team runs itself by the protocol; you answer the human and relay steering. Leave the division of labour to the lead and the roster to the members: this conversation cannot hand off, so your context must stay small enough to outlive the mission.

## 1. Scaffold

Pick a short mission slug. Read `PROTOCOL.md` beside this file (this skill's base directory) — you hold the team to it, and its spawn pattern is how you'll create the lead. Then in the project root create:

- `.team/PROTOCOL.md` — that file, copied verbatim.
- `.team/TEAM.md` — from the template at the bottom of PROTOCOL.md, mission and slug filled in.
- `.team/handoffs/` — empty.

A `.team/` left by a previous mission is history, not scaffolding: keep its TEAM.md as `TEAM-<date>.md`, scaffold fresh, and add "read the previous mission's record" to the lead's brief.

## 2. Spawn the lead

Call ListAgents once to learn your own session name. Then spawn `team-<slug>-lead` with the spawn pattern from PROTOCOL §Spawning a member.

The lead's brief carries: the mission verbatim; read `.team/PROTOCOL.md` then `.team/TEAM.md` before anything else; you are the lead — set the mission's verification bar as the first Decision, break the mission into board tasks, assemble your team (cap 6 including you), drive it to done against that bar; the anchor is <your session name> — register yourself on the roster, then report milestones and blockers to it.

The lead is live when capture-pane shows the brief submitted and the lead appears on the TEAM.md roster.

## 3. Be the window

While the team works, the human asks you for status and gives steering:

- Answer status questions from `.team/TEAM.md` and the handoff docs first — reading files interrupts nobody. Ask the lead via SendMessage only for what the files can't answer.
- Relay the human's steering — scope changes, priorities — to the lead, who re-plans the board.
- A teammate reporting a stalled permission prompt: hand the human its attach command (`tmux attach -t team-<slug>-<role>`, detach Ctrl+b d). Answering a teammate's prompt yourself, or routing a declined action to another member, bypasses the human's permission decision.
- A request for a member beyond the cap of 6 lands here: put it to the human, don't decide it.

Teammate messages wake you; the board carries the rest. Poll nothing.

## 4. Acceptance and refinement

The lead reports mission-done only after its whole-mission integration check against the verification bar in Decisions. Verify independently — re-run that bar yourself (tests and build for code; spot-check the sources for research; re-probe for security) and spot-read the output — then present the result to the human and hold the team up while they review.

Feedback loops back the way steering does: relay it to the lead, who turns it into board tasks and drives the round like any other work. For a localized tweak the human may instead attach to the responsible member and say it there — the claim rule puts that work on the board too. Rounds repeat until the human accepts.

## 5. Teardown

On acceptance: `tmux kill-session -t <tmux>` for every roster member. Complete when `tmux ls` shows no `team-<slug>-*` sessions. `.team/` stays as the mission record — a later mission's lead gets briefed on the old board and decisions. Close with a per-task summary from the board.
