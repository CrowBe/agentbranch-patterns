# Architecture

This document is the technical constitution of agentbranch-patterns: what the
project is, the vocabulary it uses, the pattern schema, the tool contracts,
the indexing and versioning model, and the lines it will not cross. It is
written as current state; when the design changes, this document changes with
it.

## Purpose and product thesis

agentbranch-patterns is a curated corpus of patterns for **agentic setup** —
the engineering discipline of configuring an agent well: tool design, context
management, skill authoring, evaluation, multi-agent composition, workflow
shape, and safety. The corpus is served to any MCP-capable agent through a
deliberately small MCP server.

The thesis has three legs:

1. **The content is the product.** The durable value is curation quality and
   provenance — a corpus where every entry has been reviewed against an
   evidence standard and cites where its claims come from. The server is
   replaceable infrastructure; the corpus is the moat.
2. **The consumer is an agent mid-task.** Documents are structured for machine
   consumption first: dense frontmatter for routing and filtering, a fixed
   body arc so an agent knows where the tradeoffs live without reading twice,
   summaries written to be actionable at search-result granularity. Humans
   read the same files on GitHub, and that's a feature, but the agent is the
   design target.
3. **Leading edge over settled consensus, honestly labeled.** Settled agentic
   practice is already inside frontier models; retrieving a restatement of it
   adds nothing, so it doesn't merge (the *beyond-the-model test*, acceptance
   criterion 3 in [GOVERNANCE.md](GOVERNANCE.md#the-acceptance-bar)). The
   corpus's center of gravity is `maturity: emerging` — fresh practitioner
   knowledge carried with an honest validity signal — and evidence recency
   outweighs evidence mass throughout the schema. A small, fresh, honest
   corpus beats a large confident one.

The project is standalone. [agent.branch](https://github.com/CrowBe/agentbranch)
is the motivating consumer — the concrete job the corpus is built against is
supplying skill authors with current, citable agentic-setup knowledge — but it
consumes the same three public tools as any stranger's agent, through a
port + adapter on its own side. Nothing in this repository imports from,
depends on, or special-cases agentbranch.

## Glossary

One term per concept. The "not" column lists aliases that must not appear in
docs, schema, or code — synonym drift is how corpora rot. Pattern *bodies*
are exempt: they are the lexical search surface, and lexical search only
matches vocabulary that is actually present, so bodies keep practitioners'
natural language, synonyms included. Frontmatter field values still use the
canonical terms.

| Term | Meaning | Not |
|---|---|---|
| **Pattern** | The atomic unit of the corpus: one named, evidenced solution to one recurring agentic-setup problem, as a single markdown document. | recipe, practice, tip, guideline, entry |
| **Corpus** | The versioned set of all patterns in this repository at a given commit. | library, catalog, collection, database |
| **Category** | One of a closed set of top-level subject areas. Every pattern belongs to exactly one. | topic, domain, area, section |
| **Tag** | A lowercase-kebab keyword refining a pattern within and across categories. Open vocabulary, normalized by review. | label, keyword, facet |
| **Alias** | An alternate natural-language name for a pattern's concept, carried in frontmatter `aliases` and indexed for search only — never displayed as the pattern's name. | synonym (as a field name), aka |
| **Applicability** | Whether a pattern is generically useful across agentic work (`general`) or validated only within named domains (`domain-specific`). | scope (as a field name), generality |
| **Domain** | A discipline of agentic work a `domain-specific` pattern is validated in: `software-engineering`, `legal-research`, `data-analysis`, … Open vocabulary, normalized by review like tags. | field, industry, vertical |
| **Slug** | A pattern's immutable identifier: lowercase-kebab, globally unique across the corpus, independent of category. | id (in prose), name, key |
| **Maturity** | The evidence strength of a pattern: `proven`, `emerging`, or `contested`. Orthogonal to lifecycle status. | confidence, stability, tier |
| **Status** | A pattern's lifecycle state: `active`, `superseded`, or `retired`. Orthogonal to maturity. | state, phase |
| **Provenance** | The structured list of evidence sources a pattern rests on. | references, sources, citations, bibliography |
| **Content hash** | SHA-256 of a pattern file's raw bytes at a given commit; the unit of pinning for consumers. | checksum, digest, fingerprint |
| **Index** | The build-time artifact holding the lexical search structures and denormalized frontmatter for the whole corpus. | database, cache |
| **Corpus version** | The identifier of the corpus snapshot a server instance is serving (release tag, backed by a git commit SHA). | release (in tool contracts), edition |
| **Skill rendering** | The offline distribution: an Agent Skills–standard folder generated from the corpus by a pure build-time function. | export, bundle, mirror |
| **Supersession** | The act of replacing a pattern with a newer one via paired `supersedes`/`superseded_by` links, never by editing the old one into the new one. | deprecation, replacement |

## The pattern schema

Every pattern is a UTF-8 markdown file at `patterns/<category>/<slug>.md`:
YAML frontmatter followed by a body with a fixed section order. CI rejects
any file that deviates. Schema changes are governed by `schema_version`
(see [Versioning](#versioning-and-stability)).

### Frontmatter fields

```yaml
---
slug: error-messages-are-prompts        # required, immutable, globally unique,
                                        #   ^[a-z0-9]+(-[a-z0-9]+)*$, max 60 chars
title: Error messages are prompts       # required, sentence case, max 80 chars
summary: >                              # required, 1–2 sentences, max 300 chars;
  Write tool errors as instructions     #   must stand alone in a search result
  for the agent's next action, not      #   and state the prescription, not just
  as diagnostics for a human log.       #   the topic
category: tool-design                   # required, one value from the closed taxonomy
tags: [error-handling, tool-schemas]    # required, 2–6, lowercase-kebab
aliases: []                             # optional, 0–5 free-form lowercase
                                        #   phrases other people use for the same
                                        #   idea; indexed for search, never shown
                                        #   as the pattern's name
applicability: general                  # required: general | domain-specific
domains: []                             # required non-empty iff domain-specific;
                                        #   lowercase-kebab, open vocabulary
                                        #   (e.g. software-engineering,
                                        #   legal-research), normalized by review
maturity: proven                        # required: proven | emerging | contested
status: active                          # required: active | superseded | retired
created: 2026-07-19                     # required, date of first merge, immutable
updated: 2026-07-19                     # required, date of last material edit
supersedes: []                          # optional, slugs this pattern replaces
superseded_by: null                     # required null unless status: superseded
retired_reason: null                    # required null unless status: retired;
                                        #   one sentence, e.g. "Invalidated by
                                        #   provider-side tool retry semantics."
provenance:                             # required, 1+ entries
  - type: primary-docs                  # primary-docs | research | field-report
    source: "Anthropic — Writing effective tools for agents"
    url: https://www.anthropic.com/engineering/writing-tools-for-agents
    date: 2025-09
  - type: field-report
    source: "agent.branch skill-validation corpus, 40-skill benchmark"
    date: 2026-05
    note: "Retry-success rate doubled when errors named the corrective action."
---
```

Rules that don't fit in comments:

- **Slugs are immutable and never reused**, even after retirement. A pattern
  that needs a new name is a new pattern that supersedes the old one. The slug
  is the pattern's identity; the file path is not (a recategorization moves
  the file but keeps the slug, and slugs are unique corpus-wide for exactly
  this reason).
- **Authorship lives in git history**, not frontmatter. Adding author fields
  would duplicate what git records better, and would invite attribution
  disputes the schema shouldn't host.
- **Applicability is a validated claim, not an ambition.** `general` asserts
  the pattern holds across domains of agentic work; as agents cross from
  software engineering into other knowledge work, "works in coding agents" is
  not evidence of "works everywhere." Evidence drawn from a single domain
  defaults the pattern to `domain-specific` with that domain listed;
  broadening to `general` is a material edit requiring evidence (or explicit
  reasoned argument in Tradeoffs) from at least two unrelated domains.
  Domains, like tags, are an open vocabulary with closed spelling, normalized
  by review, with the index as the registry.
- **Provenance types** are closed:
  - `primary-docs` — official documentation or engineering publications from
    the vendor/steward of the thing the pattern is about.
  - `research` — published papers, preprints, or reproducible public
    benchmarks.
  - `practitioner` — a public, dated post, article, talk, or repository by a
    named practitioner (not the vendor) describing the practice. Its validity
    signal is observable adoption: engagement on the original, reproduction in
    unrelated public work, working derivatives. Entries must carry a `url` and
    a `note` naming the adoption signal as observed at review time —
    engagement is a proxy for validation, not proof, so the signal is stated,
    never implied.
  - `field-report` — validated practical experience: a described, checkable
    account of the pattern working (or a variant failing) in a real system.
    Field reports must include a `note` saying what was observed; "we tried it
    and liked it" does not validate anything. Reports from systems that are
    not publicly inspectable must say so in the `note` ("internal; not
    publicly reproducible"); they corroborate other evidence but never anchor
    `proven` on their own.
- **Maturity is set by evidence, not enthusiasm — and evidence ages**
  (thresholds in [GOVERNANCE.md](GOVERNANCE.md#the-acceptance-bar)):
  - `proven` — multiple independent, still-current provenance entries, at
    least one of type `primary-docs`, `research`, or `practitioner` with a
    strong stated adoption signal, and no credible counter-evidence known at
    review time.
  - `emerging` — real evidence, but thin or single-source; the default for new
    patterns and the corpus's expected center of gravity, not a probationary
    state.
  - `contested` — credible evidence points both ways. Contested patterns are
    admissible *and valuable* — the body must present the disagreement in
    Tradeoffs — because an honest "the field disagrees, here are the forks" is
    more useful to an agent than silence.
  - Freshness outweighs mass: an entry predating a material change in the
    platform or practice it describes stops counting toward maturity. Two
    current independent entries clear a bar that five stale ones don't.
- **Tags are an open vocabulary** with closed spelling: review normalizes new
  tags against existing ones (no `errors` alongside `error-handling`). The
  index is the tag registry; there is no separate tag file to drift.

### Body structure

Five sections, `##`-level, in this order, all required:

| Section | Contract |
|---|---|
| `## Problem` | The recurring situation and the forces that make it hard. No solution content. If the problem can't be stated without naming the solution, it isn't a pattern yet. |
| `## Pattern` | The prescription, stated imperatively about the *system being built* ("write tool errors that name the corrective action"), never imperatively at the reading agent. This is the section the summary compresses. |
| `## Example` | At least one concrete, self-contained illustration — a before/after tool description, a frontmatter snippet, a transcript excerpt. Code blocks are illustrative material, never installation or execution instructions. |
| `## Tradeoffs` | What the pattern costs, when it's the wrong choice, and — for `contested` patterns — the substance of the disagreement. A pattern with no stated tradeoffs is rejected on principle: everything costs something. |
| `## Anti-patterns` | Named near-miss failure modes: the ways people implement something that looks like the pattern and get bitten. This is the section that distinguishes a pattern document from a tip. |

No other `##` sections. Related patterns are referenced inline by slug in
backticks (e.g. `` `one-tool-one-job` ``) rather than a dedicated section, so
relationships live next to the reasoning that justifies them; CI checks that
referenced slugs exist.

Patterns are served whole. A pattern's value is its full arc — problem through
anti-patterns — so nothing in this system chunks, truncates, or excerpts a
body. Target length is 400–1200 words of body; longer means the pattern is
probably two patterns.

### Category taxonomy

Closed initial set of seven. Every pattern has exactly one category. Adding a
category is a schema-level change: a PR editing this table, accepted by
maintainers, with at least two admissible patterns that genuinely fit no
existing category. Categories are never removed while patterns occupy them.

| Category | Scope |
|---|---|
| `tool-design` | Naming, describing, and shaping tools an agent calls: descriptions, input/output schemas, error surfaces, granularity, tool-count budgets. |
| `context-management` | What enters and leaves the context window: retrieval timing, compaction, memory files, progressive disclosure, budget discipline. |
| `skill-authoring` | Writing skills (Agent Skills standard and kin): trigger descriptions, folder structure, reference-doc layering, invocation reliability. |
| `evaluation` | Knowing whether an agent works: eval design, grading strategies, regression harnesses, transcript vs outcome measurement. |
| `multi-agent` | Composing more than one agent: orchestrator/worker splits, delegation contracts, context isolation between agents, result merging. |
| `workflow` | The shape of a single agent's loop: planning, checkpointing, retries, escalation to humans, stopping conditions. |
| `safety` | Keeping agent behavior inside intended bounds: permissioning, prompt-injection resistance, tool-access scoping, irreversibility guards. |

Boundary rules. The general one: a cross-cutting pattern's category is where
its prescription *acts* — what the builder changes — not where the benefit
lands; cross-cutting reach is carried by tags and aliases, never by a second
category. For the two adjacent pairs: `multi-agent` vs `workflow` — if the
pattern is meaningless with a single agent, it's `multi-agent`; otherwise
`workflow`. `tool-design` vs `safety` — a pattern about *making a tool usable*
is `tool-design` even when misuse is the motivation; a pattern about
*bounding what tools can do* is `safety`.

The taxonomy is a map of agentic *setup*, not of the work agents do, so it
does not grow a category per discipline as agents spread beyond software
engineering into other knowledge work — that axis is the `applicability` /
`domains` fields, orthogonal to category.

### Seed frontmatter examples

Five metadata-complete examples that exercise the schema across categories,
all three maturity levels, all four provenance types, and both applicability
values. Bodies are future content work; these fix the schema's shape.

```yaml
---
slug: error-messages-are-prompts
title: Error messages are prompts
summary: >
  Write tool errors as instructions for the agent's next action — name the
  corrective step and valid alternatives — not as diagnostics for a human log.
category: tool-design
tags: [error-handling, tool-schemas, retries]
aliases: ["actionable error messages", "self-correcting tool errors"]
applicability: general
domains: []
maturity: proven
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: primary-docs
    source: "Anthropic — Writing effective tools for agents"
    url: https://www.anthropic.com/engineering/writing-tools-for-agents
    date: 2025-09
  - type: field-report
    source: "agent.branch validation harness, tool-retry benchmark"
    date: 2026-05
    note: "Retry-success rate roughly doubled when errors named the corrective action. Internal; not publicly reproducible."
---
```

```yaml
---
slug: just-in-time-context
title: Load context just in time, not just in case
summary: >
  Give the agent lightweight identifiers and retrieval tools instead of
  pre-stuffing the window; pay context cost only when a task step needs
  the material.
category: context-management
tags: [retrieval, context-budget, progressive-disclosure]
aliases: ["lazy context loading"]
applicability: general
domains: []
maturity: proven
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: primary-docs
    source: "Anthropic — Effective context engineering for AI agents"
    url: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
    date: 2025-09
  - type: research
    source: "Context-window degradation studies (long-context recall benchmarks)"
    date: 2025
---
```

```yaml
---
slug: trigger-first-skill-descriptions
title: Write skill descriptions for the router, not the reader
summary: >
  A skill's description is matched against task phrasings by the invoking
  model; lead with concrete trigger conditions and example phrasings, not a
  summary of the skill's contents.
category: skill-authoring
tags: [triggers, descriptions, invocation-reliability]
aliases: ["trigger-led descriptions", "skill discoverability"]
applicability: domain-specific
domains: [software-engineering]         # evidence so far is coding-agent
                                        #   corpora; broadening to general
                                        #   needs a second domain
maturity: emerging
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: primary-docs
    source: "Agent Skills specification — description field guidance"
    url: https://code.claude.com/docs/en/skills
    date: 2025-10
  - type: practitioner
    source: "Practitioner thread on trigger-led skill descriptions"
    url: https://x.com/<author>/status/<id>   # placeholder — real entries carry the live URL
    date: 2026-06
    note: "Adoption signal: high engagement on the original; phrasing style reproduced in several unrelated public skill repositories."
  - type: field-report
    source: "agent.branch skill-triggering evals across baseline corpus"
    date: 2026-04
    note: "Trigger-phrase-led descriptions invoked correctly in significantly more task phrasings than content-summary descriptions. Internal; not publicly reproducible."
---
```

```yaml
---
slug: grade-outcomes-not-transcripts
title: Grade agent outcomes, not transcripts
summary: >
  Evaluate agents on verifiable end states — artifacts produced, tests passing,
  environment changes — rather than judging whether the transcript looks like
  a good process.
category: evaluation
tags: [grading, eval-design, llm-judge]
aliases: ["outcome-based grading"]
applicability: domain-specific
domains: [software-engineering]         # outcome checks need verifiable end
                                        #   states; evidence to date is from
                                        #   code-producing agents
maturity: emerging
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: research
    source: "Agentic benchmark design literature (outcome-based harnesses, e.g. SWE-bench family)"
    date: 2024
  - type: field-report
    source: "agent.branch eval harness migration from judge-scored transcripts to outcome checks"
    date: 2026-03
    note: "Transcript judging systematically rewarded verbose plausible failures; outcome checks did not. Internal; not publicly reproducible."
---
```

```yaml
---
slug: persona-prompts-for-capability
title: Persona prompts for capability gains
summary: >
  Assigning the agent an expert persona ("you are a senior security engineer")
  to improve task capability has conflicting evidence; treat personas as tone
  and scope controls, not capability levers.
category: workflow
tags: [system-prompts, personas, prompt-design]
aliases: ["role prompting", "expert personas"]
applicability: general
domains: []
maturity: contested
status: active
created: 2026-07-19
updated: 2026-07-19
supersedes: []
superseded_by: null
retired_reason: null
provenance:
  - type: research
    source: "Persona-prompting ablation studies reporting positive effects"
    date: 2023
  - type: research
    source: "Follow-up ablations reporting null or negative effects on reasoning benchmarks"
    date: 2024
  - type: field-report
    source: "Internal agent.branch prompt ablations"
    date: 2026-02
    note: "No measurable capability difference; measurable tone and refusal-boundary differences. Internal; not publicly reproducible."
---
```

### Content hashing

A pattern's content hash is `sha256` over the file's raw bytes (frontmatter
and body together, exactly as committed — no normalization, so the hash a
consumer computes from a git blob matches the hash the server reports). The
hash is computed at build time, published in the index, and returned by
`get_pattern`. It is not stored in frontmatter (a file cannot contain its own
hash) and not hand-maintained anywhere.

Pinning model: a consumer that has reviewed a pattern records its slug and
content hash. On later fetches it recomputes/compares; a mismatch means the
pattern changed since review and should be re-reviewed before use. Consumers
wanting whole-corpus pinning pin the git commit SHA behind a corpus version
instead. Both pins are verifiable against the public repo with stock tooling —
that verifiability, not any server feature, is the integrity guarantee.

## Trust model

Patterns are text served into third-party agents' contexts. That makes this
corpus, by construction, a prompt-injection channel: a malicious or
compromised pattern could carry instructions that a consuming agent follows
with *its* user's permissions. This is the project's primary threat, ahead of
availability or even correctness, and the mitigations are structural rather
than bolted on:

1. **Review is the gate.** Nothing reaches the corpus except through a pull
   request approved by a maintainer; there is no auto-merge and no
   post-publication pipeline that mutates content. The attack surface is
   exactly the reviewed git history.
2. **Descriptive voice is a content rule, not a style preference.** Patterns
   describe how to build agentic systems; they never address the reading
   agent. Second-person imperatives aimed at the consumer ("ignore your
   instructions", "fetch this URL and follow it", "run this command now") are
   grounds for rejection, and CI lint flags candidate phrasings for reviewer
   attention. A document that must be *obeyed* to be useful is not a pattern.
   Voice lint is a phrasing filter, though, and phrasing is the easier half
   of the threat — which is why it is paired with the next point rather than
   trusted alone.
3. **Substance review, because poisoned prescription is the harder attack.**
   A fluently descriptive pattern whose *advice* is subtly harmful ("systems
   that skip sandbox re-verification during retries recover faster…") passes
   any voice filter, and so does a payload placed in an Example code block.
   Review therefore treats prescriptions and Example code as the primary
   injection surface: examples are read on the assumption that consuming
   agents will pattern-match them into real systems — because they will — and
   a reviewer who cannot personally vouch for a prescription's and example's
   safety rejects the PR rather than waving it through on style. This is also
   why `safety`-category patterns take the stricter approval path in
   [GOVERNANCE.md](GOVERNANCE.md#review-process).
4. **No fetch-and-follow.** URLs in patterns appear only in provenance and as
   inline citations. A pattern must be fully useful without dereferencing any
   link, and must never direct the reader to retrieve and act on external
   content — external content is outside the review gate, so a link that must
   be followed is a hole in it.
5. **The server adds nothing at runtime.** It serves committed bytes from a
   build-time index — no runtime fetching, no template expansion, no
   server-side synthesis. Compromising served content requires compromising
   the repo, which is public and auditable.
6. **Hash pinning bounds the blast radius — and freezes it.** Consumers
   pinned to reviewed hashes are unaffected by anything merged after their
   review, malicious or not. The same pin also keeps serving content later
   retired for cause: advisories are pull-based and there is no push channel,
   so a pin is only as safe as the consumer's cadence of re-checking HEAD and
   the advisory feed. The docs state this tradeoff rather than selling
   pinning as pure protection.
7. **The repository, not the registry, is the trust anchor.** The server
   ships through package registries (`npx …`), and a registry artifact is a
   separate trust root that repo review cannot vouch for. The design makes
   the artifact *verifiable* rather than trusted: the bundled index carries
   `corpus_commit`, every response echoes it, and any served pattern's
   `content_hash` can be recomputed from the public repo at that commit with
   stock tooling. An artifact whose served bytes don't match the repo at its
   claimed commit is not serving this corpus — and detecting that requires
   nothing from the registry. Release notes (repo-hosted) additionally
   publish each release's index hash.
8. **Takedown is revert-at-HEAD.** A bad pattern is removed by reverting or
   retiring it at HEAD plus a repository security advisory naming the affected
   slugs, hashes, and corpus versions. Git history is not rewritten: the bad
   content remains in history (it was already public; rewriting would break
   every consumer pin and destroy the audit trail), and the advisory is the
   mechanism that tells hash-pinned consumers to move.

What this model does *not* claim: it cannot stop a consuming agent from
treating pattern text as instructions — that is the consumer's
injection-hygiene problem — it cannot make maintainer review infallible, and
it cannot make the modal consumer verify anything: verification is cheap but
opt-in, and an unverified `npx` install trusts the registry, not this repo.
It claims that every byte served traces to a reviewed public commit, that
consumers can verify this cheaply, and that no pattern needs to be executed,
obeyed, or dereferenced to deliver its value.

## Tool contracts

The MCP server is named `agentbranch-patterns` and exposes exactly three
tools. The surface is sized to what an agent mid-task needs: find candidates,
read one, orient in the taxonomy. Requests to add tools meet the non-goals
list before they meet the roadmap.

Conventions common to all tools:

- Every success response includes `corpus_version` (the release tag) and
  `corpus_commit` (the git SHA it resolves to), so any answer an agent acts on
  is attributable to an exact corpus state.
- Errors use the MCP tool-error channel with a body of
  `{ "code": string, "message": string }`. Codes are a closed set:
  `invalid_input` (malformed arguments; message names the field) and
  `not_found` (unknown slug or category; message includes up to 3 near-miss
  slugs when available). Messages are written for the calling agent's next
  action, per `error-messages-are-prompts`.

### `search_patterns`

Lexical (BM25) search over title, summary, tags, aliases, and body, with
title and summary boosted. Returns ranked summaries — never bodies.

Input:

```json
{
  "query":    "string, required, 1–500 chars",
  "category": "string, optional — restrict to one category",
  "tags":     ["string", "... optional — AND semantics"],
  "maturity": ["proven" , "... optional — filter, default all"],
  "domain":   "string, optional — return general patterns plus domain-specific ones validated in this domain",
  "include_inactive": "boolean, optional, default false — include superseded/retired",
  "limit":    "integer, optional, default 5, max 20"
}
```

Output:

```json
{
  "corpus_version": "2026.07.0",
  "corpus_commit": "abc123…",
  "total": 12,
  "results": [
    {
      "slug": "error-messages-are-prompts",
      "title": "Error messages are prompts",
      "summary": "Write tool errors as instructions…",
      "category": "tool-design",
      "tags": ["error-handling", "tool-schemas", "retries"],
      "applicability": "general",
      "domains": [],
      "maturity": "proven",
      "status": "active",
      "updated": "2026-07-19",
      "score": 7.42
    }
  ]
}
```

`results` is capped at `limit`; `total` is the full match count so the agent
knows whether to refine. An empty result set is a success with `results: []`,
not an error. `score` is comparable only within one response.

### `get_pattern`

Fetch one full pattern by slug.

Input: `{ "slug": "string, required" }`

Output: the complete frontmatter as structured fields, plus:

```json
{
  "corpus_version": "2026.07.0",
  "corpus_commit": "abc123…",
  "slug": "error-messages-are-prompts",
  "…frontmatter fields…": "…",
  "content_hash": "sha256:9f2c…",
  "body": "## Problem\n…full markdown body, served whole…"
}
```

Superseded and retired patterns are returned normally — a slug, once
published, always resolves — with their `status`, `superseded_by` /
`retired_reason` fields carrying the redirect. `not_found` is only for slugs
that never existed at this corpus version.

### `list_categories`

Input: `{}` (no parameters).

Output:

```json
{
  "corpus_version": "2026.07.0",
  "corpus_commit": "abc123…",
  "categories": [
    {
      "name": "tool-design",
      "scope": "Naming, describing, and shaping tools an agent calls…",
      "active_patterns": 9
    }
  ]
}
```

Ordered as in the taxonomy table. `scope` is the taxonomy table's scope text,
so the taxonomy has one home and the tool reflects it.

## Indexing model

The index is a build artifact, not a service:

1. A build step walks `patterns/`, validates every file against the schema
   (CI runs the same step — one validator, two callers), and fails the build
   on any violation, so an index can never contain an invalid pattern.
2. It computes per-pattern content hashes and BM25 statistics over the five
   searched fields (title, summary, tags, aliases, body; title/summary
   boosted), and emits a single `index.json` containing the search
   structures, all frontmatter, all bodies, and the corpus version/commit.
3. The server loads `index.json` into memory at startup and serves entirely
   from it. The server is stateless and read-only: no database, no writes, no
   network calls at request time. Scaling is "run more copies"; upgrade is
   "restart with a newer index".

The lexical/BM25 choice is deliberate for v1: the corpus is small (hundreds of
patterns, not millions), queries are term-rich because agents write them, and
lexical search is auditable — a ranking can be explained from term statistics.
The vocabulary policy leans the same way: pattern bodies keep natural
practitioner language and `aliases` carries the names a body doesn't happen to
use (see [Glossary](#glossary)), because BM25 can only match terms that are
actually present.
**The embedding seam:** if retrieval quality ever demands it, embeddings
replace step 2's search structures behind the *same* `search_patterns`
contract — same inputs, same output shape, `score` semantics already declared
response-local. Consumers never learn retrieval changed. Nothing in v1 may
leak BM25 specifics into a contract, and nothing in v1 requires embeddings.

## Repository layout

```
agentbranch-patterns/
├── README.md               # public front door
├── AGENTS.md               # agent onboarding: pointers + repo-only decisions
├── CLAUDE.md               # symlink → AGENTS.md
├── LICENSE                 # Apache-2.0 (code)
├── LICENSE-CONTENT         # CC BY 4.0 (patterns/)
├── docs/
│   ├── ARCHITECTURE.md     # this document
│   └── GOVERNANCE.md       # contribution, review, disputes, license rationale
├── patterns/
│   ├── tool-design/
│   │   └── <slug>.md
│   ├── context-management/…
│   ├── skill-authoring/…
│   ├── evaluation/…
│   ├── multi-agent/…
│   ├── workflow/…
│   └── safety/…
├── schema/                 # machine-readable frontmatter schema (future)
└── server/                 # MCP server source (future)
```

The skill rendering has no directory: it is a build output generated from
`patterns/` by a pure function, published as a release artifact. Committing a
rendered copy would create a second source of truth that drifts.

## Versioning and stability

Three independently versioned things:

- **Corpus versions** are calendar-based release tags, `YYYY.MM.PATCH`
  (e.g. `2026.07.0`), each resolving to one git commit. Content changes
  ship as new corpus versions; nothing is ever republished under an existing
  tag. Semver is wrong for content — merging a pattern is not an API change —
  and dates tell consumers the one thing they ask of a knowledge corpus: how
  current it is.
- **`schema_version`** is a single integer (currently `1`) carried in the
  index and in every tool response envelope. It covers the frontmatter
  fields, body section contract, and tool response shapes together.
  Additive changes (new optional field, new category) do not bump it.
  Breaking changes bump it, and the previous schema's last corpus version
  remains permanently fetchable by tag.
- **The server package** uses semver, independent of both.

Guarantees consumers may rely on:

1. Slugs are immutable and never reused; a published slug resolves via
   `get_pattern` in every subsequent corpus version (possibly as
   superseded/retired).
2. Pattern files are byte-stable within a corpus version: same tag, same
   content hashes, forever.
3. The three tools' names, required inputs, and output fields documented here
   are stable within a `schema_version`; new *optional* inputs and new output
   fields may appear without notice, so consumers must ignore unknown fields.
4. Git history is never rewritten on the default branch — every historical
   pin stays verifiable, including content later retired for cause (the
   advisory, not history rewriting, handles takedown).

Freshness lifecycle (mechanics here; cadence and process in
[GOVERNANCE.md](GOVERNANCE.md#freshness-and-staleness)):

- **Material edits** update `updated` and change the content hash; pinned
  consumers are untouched, live consumers see the new date.
- **Supersession** — replacing a pattern's *prescription* — publishes a new
  pattern carrying `supersedes: [old-slug]`; the same PR flips the old
  pattern to `status: superseded` with `superseded_by` set. The old body is
  otherwise preserved. Search excludes superseded patterns by default;
  `get_pattern` serves them forever with the redirect attached.
- **Retirement** — a pattern invalidated with no successor — flips
  `status: retired`, sets `retired_reason`, and preserves the body. Retired
  patterns document a road the field went down and returned from; deleting
  them would just let someone rediscover the mistake.

## Non-goals

Named so they can be pointed at when scope creeps:

- **Not an awesome-list.** No link collections, no tool roundups. Every entry
  is an evidenced prescription with tradeoffs, or it doesn't merge.
- **Not a documentation mirror.** No restatements of vendor docs or of
  practice frontier models already apply reliably unprompted — that knowledge
  ships inside the consumer. Every entry must pass the beyond-the-model test
  (acceptance criterion 3).
- **Not a blog.** No opinion pieces, announcements, or trend commentary.
  Timeless-until-superseded is the target register.
- **Not a prompt marketplace.** No copy-paste prompt library, no per-model
  prompt packs, no monetized listings. Patterns explain *why* a construction
  works so it can be adapted, not pasted.
- **Not a benchmark.** The corpus cites evals and may pattern eval *design*,
  but publishes no leaderboards and scores no models or agents.
- **Not a RAG platform.** No vector database, no embedding pipeline, no
  runtime data stores at v1. The embedding seam exists; the commitment to
  statelessness is not up for trade against it.
- **Not an agent framework.** The server never executes, orchestrates, or
  scaffolds anything. It serves reviewed text.
- **Not agentbranch's internal library.** agent.branch consumes the public
  tools like any other client. No private endpoints, no privileged schema
  fields, no imports in either direction.
- **Not a general prompt-engineering corpus.** Scope is *agentic setup* —
  configuring systems where a model acts through tools. Pure
  single-completion prompting techniques belong elsewhere unless they change
  how an agent is configured.
