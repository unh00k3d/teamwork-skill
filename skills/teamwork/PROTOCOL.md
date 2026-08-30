# Team protocol

You are a member of a team of Claude Code sessions sharing one machine, one filesystem, and one mission. Every member is a full session in its own tmux; the human may attach to yours at any time. Coordination runs on two channels: `.team/TEAM.md` — the single source of truth — and the session message bus (SendMessage, names from the roster), which carries wakeups and questions, never state.

## TEAM.md

Three sections, edited in place (template at the bottom):

- **Roster** — one line per member: session name (from ListAgents), tmux session, role, status (active / handing-off / retired).
- **Board** — one line per task: id, description, status (open / claimed / done), owner, verification (how done was checked, against the mission's bar).
- **Decisions** — dated one-liners for choices that bind everyone: the mission's verification bar (set first), interfaces, conventions, scope calls.

## Claiming

Claim before you work: set yourself as owner on the board, then start. One owner per task; stay inside your claimed tasks' scope — the files, sections, or resources they cover. If you need something outside it, ask its owner or claim an open task that covers it. Work without a claim on the board doesn't exist.

A task is **done** when it meets the mission's verification bar — the standard the lead recorded in Decisions — with the evidence noted on the board. The bar is domain-shaped: green tests and a clean build for code; source-cited coverage for research and analysis; a probed-everything checklist for security. Whatever it is, it must be checkable — an owner can tell done from not-done — not "looks finished". Then wake whoever the board shows is waiting on it.

## Communication

- After a board edit that concerns someone, wake them: one line — "board updated: <what changed>".
- Broadcast = message every active roster name.
- Waiting on a member: subscribe with SendMessage `{notify_when_idle: true}` (no message). Polling or "are you done?" messages mean this wiring is missing.
- Questions for the human go through the anchor (named on the roster).

## Spawning a member

Any member may add one when a real workstream is unstaffed AND the roster (lead included, anchor excluded) is under **6 members**. At the cap, route the need lead → anchor → human.

```
tmux new-session -d -s team-<slug>-<role> -c <workdir> "claude --dangerously-skip-permissions"
tmux send-keys -t team-<slug>-<role> "<brief>"
tmux send-keys -t team-<slug>-<role> Enter
```

Text and Enter are two separate send-keys calls — a single call's trailing Enter is swallowed by paste handling. The brief carries: read `.team/PROTOCOL.md` then `.team/TEAM.md` before anything else; your role; the anchor's and your spawner's session names. The newcomer's first act after reading is registering itself on the roster; the spawner then broadcasts the arrival. A member is joined when it appears on the roster.

## Handoff

When your remaining context budget runs low, hand off before you're forced to:

1. Write `.team/handoffs/<your-name>.md`: current state, decisions made, remaining work, file map, open questions.
2. Spawn your successor — same tmux pattern, exempt from the cap since it replaces you. Brief = your role + read your handoff doc first.
3. Roster: mark yourself retired, add the successor.
4. Broadcast the succession, then stop working — the successor owns your claims.

The lead hands off like anyone else.

## Roles

- **Lead**: sets the mission's verification bar as the first Decision, breaks the mission into board tasks, assembles the team, unblocks members, owns the whole-mission integration check against that bar, reports milestones and blockers to the anchor. How to divide the work is the lead's call — split along whatever seams keep members out of each other's way: file boundaries for code, subtopics or sources for research, targets or surfaces for security.
- **Anchor**: the human's window, and only that — it stays off the board and off the roster's working rotation. Report blockers and milestones to it.
- Everyone else: whatever the brief says, changeable by claiming differently on the board.

## Permissions

Members run with `--dangerously-skip-permissions`: every tool call auto-approves, so work never stalls on a prompt. The human accepted that risk for this mission — keep the blast radius matching it: work inside the mission's directory, and route anything destructive or outward-facing (deploys, force-pushes, publishing, mass deletes) through the anchor to the human first.

When the human spawns a stricter team (a different flag in the spawn pattern), a prompt you cannot pass = mark the task blocked, tell the lead and the anchor, and move to another claimed task — the human answers prompts by attaching to your tmux. A peer never answers a prompt for the human, and never performs an action the human declined for someone else.

## TEAM.md template

    # Team: <slug>
    Mission: <mission>

    ## Roster
    | name | tmux | role | status |
    |---|---|---|---|

    ## Board
    | id | task | status | owner | verification |
    |---|---|---|---|---|

    ## Decisions
    - <date>: <decision>
