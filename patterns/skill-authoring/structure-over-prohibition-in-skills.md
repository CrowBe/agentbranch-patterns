---
slug: structure-over-prohibition-in-skills
title: Structure skills around process, not prohibition
summary: >
  Write a skill as a repeatable process skeleton with layered reference
  material, not a growing list of "don't do X" rules — negative constraints
  make the forbidden behavior more salient, and skills that try to cover
  every case sprawl instead of staying legible.
category: skill-authoring
tags: [skill-design, progressive-disclosure, instruction-design, anti-patterns]
aliases: ["positive framing", "process skeleton", "writing great skills"]
applicability: domain-specific
domains: [software-engineering]         # evidence so far is one engineering
                                        #   skills ecosystem
maturity: emerging
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - key: skills-wgs
    type: practitioner
    source: "mattpocock/skills — docs/productivity/writing-great-skills.md"
    url: https://github.com/mattpocock/skills/blob/main/docs/productivity/writing-great-skills.md
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal with counts and an observation date."
  - key: aihero-wgs
    type: practitioner
    source: "AI Hero — The /writing-great-skills Skill"
    url: https://www.aihero.dev/skills-writing-great-skills
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal with counts and an observation date."
---

## Problem

A skill's job is to wrangle a repeatable process out of a stochastic model —
the goal is not the same output on every run, but the same *steps* every run.
Authors chasing that repeatability naturally reach for guardrails, and the
cheapest guardrail to write is a prohibition: "don't skip the tests," "don't
guess at the API," "never touch the migration files." Left unchecked, a
skill accumulates one prohibition per remembered failure until the body is
mostly a list of things not to do, covering every case anyone has ever hit
and full of language repeating things already said elsewhere in the file.
Two things go wrong at once. First, naming the forbidden behavior makes it
more available to the model, not less — telling an agent "don't touch the
migration files" puts "migration files" in its attention when it wasn't
there before. [^skills-wgs] Second, a skill whose body is one clause taller
every time someone hits an edge case stops being legible: the reader (human
or model) can no longer tell the load-bearing steps from the accumulated
exceptions.

## Pattern

Design the skill as a fixed process skeleton — an ordered set of steps
stated as positive actions — and push everything that isn't core sequence
into layered reference material the model reads only when a step actually
needs it. Concretely:

- State what to do, not what to avoid. If a step has a common failure mode,
  encode the correct behavior in the step itself rather than appending a
  prohibition about the wrong one.
- Reach for compact, pre-trained concepts as leading words — "tight,"
  "red," "tracer bullet" — that anchor a lot of behavior in a couple of
  tokens instead of a paragraph of spelled-out constraint. [^skills-wgs]
- Structure disclosure in a hierarchy: the top-level skill file states the
  steps; deeper detail, examples, and edge cases live in reference files
  the skill points to, so the top level stays scannable regardless of how
  much depth exists underneath.
- Treat every sentence with a single-source-of-truth test: if a fact is
  already stated once in the skill or its references, a second restatement
  is pruned, not kept "for emphasis."
- When a skill accumulates enough steps or enough optional sub-flows that
  no one can hold the whole thing in view, split it, or add a router skill
  that indexes the set and tells the caller which one applies — the fix for
  sprawl is decomposition, not a longer document. [^aihero-wgs]

## Example

Before (prohibition-driven, from a hypothetical `deploy` skill):

> Deploy the service. Don't skip the smoke test. Don't push if CI is red.
> Don't deploy on Fridays unless explicitly told to. Don't forget to check
> the migration status. Don't assume the feature flag is off by default.

After (structure-driven):

> 1. Confirm CI is green for the target commit.
> 2. Run the smoke test suite against staging.
> 3. Check migration status; apply pending migrations before deploy, not after.
> 4. Deploy.
>
> See `reference/rollback.md` for the rollback sequence if step 2 fails.

The second version states five behaviors as four ordered steps plus a
reference pointer; nothing forbidden is named, and a reader can see the
whole process at a glance.

## Tradeoffs

- Positive framing takes more authoring effort up front than bolting on a
  prohibition after a failure — it requires figuring out what *should*
  happen, not just naming what shouldn't.
- Progressive disclosure only pays off once a skill has enough depth to
  need layering; a genuinely three-step skill gains nothing from a
  reference-file hierarchy and the extra indirection is pure overhead.
- Leading words that compress well for one model or one team's shared
  vocabulary ("tracer bullet") can be opaque to a different audience,
  trading token economy for a steeper onboarding curve.
- The single-source-of-truth prune can go too far: some restatement at a
  step boundary is a legitimate reminder, not duplication, and pruning it
  purely for length can leave a step under-specified right when the model
  needed the reminder.

## Anti-patterns

- **The elephant problem (negation):** naming the forbidden behavior to
  rule it out ("don't hardcode the API key") raises its salience instead of
  suppressing it; state the correct action instead ("load the API key from
  the config module"). [^skills-wgs]
- **Negative space:** silence on a decision doesn't stay neutral — it
  delegates that decision to whatever the model's priors happen to be.
  Omitting guidance on an important fork is a choice, just an unexamined
  one.
- **Sediment:** old exceptions and one-off fixes pile up as clauses long
  after the situation that prompted them stopped recurring, and no one
  prunes them because each individually looks harmless.
- **Sprawl:** the skill grows to cover every case anyone has ever hit
  instead of the common path, until the core process is buried in edge
  cases.
- **No-op:** a step that looks like it constrains behavior but doesn't
  actually change what the model does, adding length without adding
  reliability.
