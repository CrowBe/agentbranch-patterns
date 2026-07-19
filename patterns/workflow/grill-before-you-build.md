---
slug: grill-before-you-build
title: Interview the brief before implementing it
summary: >
  Before writing code from an ambiguous brief, run a one-question-at-a-time
  interview down the decision tree — grounded in the existing codebase or
  docs, each question paired with a recommended answer — and don't proceed
  until the human confirms shared understanding.
category: workflow
tags: [elicitation, planning, requirements, decision-tree]
aliases: ["grilling", "requirements interview", "one question at a time"]
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
  - key: skills-grilling
    type: practitioner
    source: "mattpocock/skills — docs/productivity/grilling.md"
    url: https://github.com/mattpocock/skills/blob/main/docs/productivity/grilling.md
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal with counts and an observation date."
  - key: skills-grill-docs
    type: practitioner
    source: "mattpocock/skills — docs/engineering/grill-with-docs.md"
    url: https://github.com/mattpocock/skills/blob/main/docs/engineering/grill-with-docs.md
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal with counts and an observation date."
---

## Problem

An agent handed an ambiguous brief will usually produce something
plausible-looking rather than stopping to ask — it fills gaps with silent
assumptions because assumptions are cheaper to generate than questions.
The mismatch between what was meant and what got assumed often isn't
visible until an implementation already exists, by which point turns and
tokens are sunk into a direction that has to be unwound. Asking questions
well is itself hard: a bulk list of questions dumped on the human at once
is bewildering and invites shallow, rushed answers, and generic
questions that the codebase or existing docs could already answer just
waste the human's attention on things the agent should have checked
itself.

## Pattern

Run a structured interview before any implementation step, shaped by the
decision tree the brief actually implies:

- Ask one question at a time and wait for the answer before asking the
  next — never a bulk list. [^skills-grilling] Sequential questioning walks
  down the decision tree branch by branch, resolving dependencies between
  decisions in order, so later questions can build on earlier answers
  instead of being asked in a vacuum.
- Pair every question with a recommended answer, so confirming is often
  faster than composing a fresh response, and disagreement is cheap to
  signal. [^skills-grilling]
- Check what the existing codebase or docs can already answer before
  asking the human — a question the code settles on its own is noise, not
  diligence. [^skills-grill-docs]
- Don't proceed to implementation until the human confirms that shared
  understanding has actually been reached; a brief interview that ends
  without confirmation is indistinguishable from no interview at all.
- Persist the resolved answers rather than letting them evaporate at the
  end of the session — write sharpened terminology to a shared glossary
  (per `domain-glossary-file-prevents-drift`) and reserve a decision record
  for the genuinely hard-to-reverse, surprising calls, so the next session
  doesn't re-derive (or re-ask) the same ground. [^skills-grill-docs]

## Example

Given the brief "add rate limiting to the API," a grounded interview might
run:

> **Agent:** Should rate limits be per-user or per-IP? Recommend per-user,
> since the API already requires auth on every route.
> **Human:** Per-user, agreed.
>
> **Agent:** What should happen when a client exceeds the limit — reject
> with 429, or queue and delay? Recommend 429, since the codebase has no
> existing queueing infrastructure.
> **Human:** 429 is fine.
>
> **Agent:** Confirming shared understanding: per-user limits, 429 on
> excess, using the existing auth middleware to key the limiter. Proceeding
> to implementation.

Each question is answerable in one line because it's paired with a
recommendation grounded in what the codebase already does, and the
explicit confirmation step is what licenses moving to code.

## Tradeoffs

- The synchronous back-and-forth is real overhead on tasks where the brief
  genuinely is unambiguous; forcing an interview onto a one-line, obvious
  request is friction with no payoff.
- A weak interviewer becomes its own source of noise — too many questions,
  or questions pitched too generically, tire out the human faster than a
  wrong first implementation would have.
- The technique is only as good as its grounding: an interview run without
  consulting the codebase or docs first degenerates into generic
  questions the agent should have answered itself.
- Recommended answers can anchor the human toward agreement even when a
  different answer was actually better — a suggestion presented alongside
  every question carries some risk of rubber-stamping.

## Anti-patterns

- **Skipping straight to implementation "to save time":** treats a wrong
  first attempt as cheaper to fix than a clarifying question, which is
  rarely true once the wrong direction has been built on.
- **Ungrounded grilling:** asking questions the existing docs or codebase
  already answer, because the interview step ran without checking them
  first.
- **Bulk questioning:** presenting every open question as one long list
  instead of walking the decision tree one branch at a time, producing
  rushed or shallow answers.
- **Discarding the resolution:** running the interview, getting answers,
  and using them only in the moment without writing the sharpened
  terminology or the hard decisions anywhere — the next session re-asks
  what this one already settled.
