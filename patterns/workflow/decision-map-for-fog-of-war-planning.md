---
slug: decision-map-for-fog-of-war-planning
title: Chart big unclear work as a map of decision tickets, not a backlog
summary: >
  When an effort is too large or unclear for one session, maintain a single
  living map issue of decision tickets — created only once a question can
  be stated precisely — resolved one at a time with the answer recorded, so
  a frontier of resolvable work stays visible across sessions.
category: workflow
tags: [planning, issue-tracking, checkpointing, decomposition]
maturity: emerging
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: primary-docs
    source: "mattpocock/skills — docs/engineering/wayfinder.md"
    url: https://github.com/mattpocock/skills/blob/main/docs/engineering/wayfinder.md
    date: 2026-07
  - type: primary-docs
    source: "AI Hero — The /wayfinder Skill"
    url: https://www.aihero.dev/skills-wayfinder
    date: 2026-07
  - type: field-report
    source: "Matt Pocock on X, announcing /wayfinder in personal use"
    url: https://x.com/mattpocockuk/status/2072716979195326905
    date: 2026-07
    note: "Self-reported use planning an entire course with the skill, closing in on ~100 separate grilling/prototyping/research sessions all contributing back to one central, growing map."
---

## Problem

Some efforts are too large for one agent session to plan, let alone
execute, and the standard fix — write a backlog of tickets up front — fails
here for a specific reason: the sub-questions aren't known yet. The work is
"wrapped in fog," where the path from here to the goal isn't visible until
some of it has been walked. Writing tickets before a question can be stated
precisely produces tickets that encode unexamined assumptions instead of
real decisions, and nobody notices until an agent picks one up and
discovers it doesn't actually specify anything actionable. Meanwhile a flat
backlog gives no way to see which of the many open questions can be worked
on *right now* versus which are blocked on answers that don't exist yet —
so sessions either stall waiting on the wrong thing or duplicate work
because two tickets silently depended on the same unresolved question.

## Pattern

Maintain a single index issue (a map) that holds the current state of an
effort as a set of child decision tickets, rather than a flat backlog:

- Name the destination first — state the goal the map is charting a route
  toward, so scope has a boundary from the start.
- Keep a question as "fog" until it can be stated precisely. The test for
  whether something is a ticket yet or still fog is whether you can state
  the question precisely *now* — if not, it isn't ready to become a ticket.
- Convert fog into a ticket only at that point, and classify it: **HITL**
  (needs a live human exchange to resolve) or **AFK** (a research task a
  background subagent can resolve without the human present).
- Use the issue tracker's native blocking relationships between tickets, so
  the current **frontier** — the set of open, unblocked tickets — is
  visible without anyone maintaining it by hand.
- Resolve at most one ticket per session. Record the answer as a resolution
  comment on the ticket before closing it, and append a single-line pointer
  to a running "decisions so far" summary on the map issue.
- If an initial look at the effort reveals it isn't actually foggy — the
  path is already knowable — stop; this pattern is overhead for work that a
  normal backlog already handles.

## Example

A map issue titled `wayfinder: ship the new export format` holds a short
"decisions so far" list and links to child tickets:

- `#142 — Which serialization library?` (HITL, blocked by nothing) — part
  of the frontier, resolvable now.
- `#145 — Does the old format need a migration path?` (AFK, blocked by
  nothing) — assigned to a background research subagent.
- `#150 — Where does the new format get versioned?` (HITL, blocked by
  `#142`) — not yet part of the frontier; can't be answered until the
  serialization choice lands.

Resolving `#142` adds a line to "decisions so far" ("serialization:
protobuf, chosen for schema evolution"), closes the ticket, and moves
`#150` onto the frontier automatically because its blocker cleared.

## Tradeoffs

- The ticket/issue-tracker ceremony is overhead for work that's genuinely
  small or already clear; forcing a map onto a task with no real fog just
  adds ritual.
- It requires discipline to write concrete resolution comments rather than
  closing a ticket with an implicit "done" — a map with unrecorded
  resolutions loses exactly the value it exists to provide.
- The map itself needs pruning attention; without it, a long-running effort
  accumulates enough closed tickets and stale fog that the map becomes as
  hard to navigate as the backlog it replaced.
- HITL/AFK classification is only as good as the judgment behind it — an
  agent that's too willing to mark its own ambiguous questions as AFK
  quietly resolves things that actually needed a human.

## Anti-patterns

- **Fog masquerading as a ticket:** writing a ticket for a question that
  can't yet be stated precisely, because it looks like progress; the
  ticket sits unresolvable and blocks nothing usefully.
- **Resolving more than one ticket per session:** collapses the checkpoint
  value the pattern is built on — the map exists so each session has a
  clear, reviewable unit of decision-making, not so a session can plow
  through the whole frontier at once.
- **Letting an agent self-answer an HITL ticket:** defeats the
  classification's purpose; if a question was marked as needing a human,
  an agent resolving it alone reintroduces the original problem of
  decisions made on unexamined assumptions.
- **Treating the map as a data store instead of an index:** the map issue
  is meant to point at child tickets and summarize resolutions, not to
  accumulate the actual content of every decision inline — that content
  belongs on the tickets themselves.
