---
slug: domain-glossary-file-prevents-drift
title: Settle domain vocabulary in one committed glossary file
summary: >
  Capture a project's domain terms in a single canonical glossary file the
  moment ambiguity surfaces, rather than re-explaining or letting synonyms
  drift across skills, prompts, and sessions.
category: context-management
tags: [memory-files, shared-vocabulary, progressive-disclosure, drift-prevention]
aliases: ["project glossary", "ubiquitous language", "shared vocabulary file"]
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
  - key: skills-context
    type: practitioner
    source: "mattpocock/skills — CONTEXT.md"
    url: https://github.com/mattpocock/skills/blob/main/CONTEXT.md
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal (repo engagement, derivative use) with counts and an observation date."
  - key: skills-grill-docs
    type: practitioner
    source: "mattpocock/skills — docs/engineering/grill-with-docs.md"
    url: https://github.com/mattpocock/skills/blob/main/docs/engineering/grill-with-docs.md
    excerpt: null   # TODO(review): verbatim quote required before merge
    date: 2026-07
    note: "TODO(review): quantify adoption signal with counts and an observation date."
---

## Problem

As agentic work scales across many sessions, skills, and sometimes multiple
agents, the same concept quietly picks up multiple names — one skill's
prompt calls something a "backlog," another calls the same thing a
"queue," and a third uses "backlog" for the *work* rather than the *tool*
holding it. No single conversation is long enough for this to look like a
problem; each skill's author only sees their own usage. But an agent
reading one skill's instructions has no way to know that "backlog" there
means something different from "backlog" in the skill it just ran, and it
resolves the ambiguity silently, in whatever direction its priors push it.
The alternative failure mode is just as costly in a different currency:
re-explaining the same domain concept inline, in full, inside every skill
or prompt that touches it, burns context budget on prose that a shared
reference could have replaced with a name.

## Pattern

Maintain a single committed file — a project glossary — that defines the
closed domain vocabulary once, and have skills and prompts reference it by
term instead of re-explaining the concept inline:

- One term per concept, with an explicit list of aliases that are *not*
  allowed to reappear in docs, skills, or prompts, so a reviewer has
  something concrete to reject a synonym against instead of relying on
  memory of past usage. [^skills-context]
- Write the definition the moment ambiguity actually surfaces — during an
  interview, a design discussion, a code review — rather than trying to
  anticipate the whole vocabulary up front. [^skills-grill-docs] A glossary built reactively from
  real confusion stays grounded; one drafted speculatively tends to define
  terms nobody ends up needing.
- Treat the file as the terminology's single source of truth: when a skill
  needs to use a domain term, it references the glossary rather than
  restating the definition, so there is exactly one place to update when
  the term's meaning changes.
- Keep it scoped to genuinely cross-cutting, recurring terms — the
  vocabulary that multiple skills, prompts, or sessions need to agree on —
  not every local variable name or one-off phrase that only ever appears
  in a single place.

## Example

A `CONTEXT.md` entry resolving a real ambiguity: [^skills-context]

> **Issue tracker** — the hosting system (GitHub Issues, Linear, etc.) where
> work units live. Not: backlog, backend, queue.
>
> **Decision ticket** — an issue containing an unresolved question, distinct
> from an issue tracking implementation work. Not: task, item.

Before this entry existed, one skill's prompt used "backlog" to mean the
tracker itself and another used it to mean the pending work inside it;
after, every skill that needs either concept names it as "issue tracker" or
"decision ticket" and points at this file instead of re-defining either
term.

## Tradeoffs

- The file requires upkeep as the domain evolves; a glossary that goes
  stale is worse than no glossary, because agents will trust and act on a
  definition that no longer matches how the term is actually used.
- It adds a real indirection — a file agents must actually load to benefit
  from — which costs context budget of its own, even though it's smaller
  than the cost of re-explaining every term inline.
- The pattern only pays off for vocabulary that genuinely recurs across
  multiple skills or sessions; a term used in exactly one place gets no
  benefit from being extracted into a shared file, just an extra hop to
  read it.
- A glossary can become a false authority if it's populated speculatively
  rather than from real resolved ambiguity — confident-looking definitions
  for terms nobody actually disputes add ceremony without preventing any
  drift.

## Anti-patterns

- **Synonym accumulation:** letting a near-duplicate term ("errors"
  alongside "error-handling") into the vocabulary because it felt close
  enough at the time it was written — the whole value of the glossary is
  that it's the place synonyms get caught, not waved through.
- **Unread ceremony:** writing the glossary once and never having any skill
  or prompt actually reference it, so it rots into a document that
  describes a vocabulary the project has since drifted away from.
- **Scattering redefinitions:** restating a term's meaning inline in a
  skill or prompt instead of pointing at the glossary, which recreates the
  exact drift risk the shared file exists to close off.
- **Speculative drafting:** writing definitions for terms before any real
  ambiguity has surfaced around them, producing a glossary that looks
  thorough but wasn't grounded in an actual point of confusion.
