# Governance

How content enters, changes, and leaves the corpus, and who decides. The
technical shape of patterns (schema, taxonomy, trust model, versioning
mechanics) is defined in [ARCHITECTURE.md](ARCHITECTURE.md); this document
defines the human process wrapped around it.

The operating principle is **open contribution, curated acceptance**. Anyone
may propose a pattern; maintainers decide what merges. This is not a
democracy of content — the corpus's value is the acceptance bar, and the bar
is only worth something if someone is accountable for holding it.

## Roles

- **Maintainers** hold merge rights, own the acceptance bar, run review
  sweeps, and arbitrate disputes. The maintainer list is the set of people
  with merge permission on this repository — GitHub is the registry; no
  parallel list is kept here to drift. New maintainers are added by consensus
  of existing maintainers, based on a track record of accepted contributions
  and review participation.
- **Contributors** are anyone opening a pull request: pattern authors,
  reviewers-by-comment, staleness reporters. No sign-up, no CLA — the
  license grant in a PR (see [License](#license)) is the whole agreement.

**Bootstrap phase.** The project currently has a single maintainer, and every
rule below that presumes a group — second approvals, dispute escalation,
consensus — activates only once a second maintainer exists. Until then, the
second-approval requirement for `safety`-category patterns and for changes to
the trust model or schema is replaced by an extended open-comment window of
seven days, and process disputes have no appeal beyond the sole maintainer.
That is a real cost, stated rather than papered over; it ends when the
maintainer count does.

## Contributing a pattern

1. **Search first.** Check `patterns/` (or `search_patterns`) for the idea
   under other words. If a pattern half-covers it, prefer a PR improving that
   pattern over a near-duplicate; overlapping patterns dilute search and
   force agents to reconcile them.
2. **Write against the schema** in
   [ARCHITECTURE.md](ARCHITECTURE.md#the-pattern-schema): all frontmatter
   fields, all five body sections, provenance for every load-bearing claim.
   New patterns enter at `maturity: emerging` unless the evidence already
   clears the `proven` bar; claiming `proven` is a reviewable assertion, not
   a self-assessment.
3. **Open a PR** — one pattern per PR, so review quality doesn't degrade
   across a batch. CI validates schema, slug uniqueness, cross-reference
   integrity, and voice-lint; a red check means the PR isn't reviewable yet.
4. **Review happens in the open**, on the PR thread. Expect substantive
   pushback on evidence and scope; that's the product working.

Edits to existing patterns follow the same flow. Material edits (anything
changing the prescription, tradeoffs, or evidence) update `updated` and are
held to the same evidence standard as new patterns; typo-level fixes are
merged on sight.

## The acceptance bar

A pattern merges when a reviewing maintainer can answer yes to all six:

1. **Is it a pattern?** A recurring problem plus a named, reusable
   prescription. A common rejection is *true but not a pattern*:
   one-off war stories, product tips tied to a single vendor's UI, and
   observations with no prescription all fail here — regardless of quality.
2. **Is it in scope?** Agentic setup, per the category taxonomy. A pattern
   that fits no category is either out of scope or evidence for a taxonomy
   PR — the reviewer says which.
3. **Is it beyond the model?** A pattern that restates vendor documentation
   or practice a current frontier model already reliably applies unprompted
   is rejected regardless of quality — that knowledge ships inside the
   consumer, and retrieving it adds nothing. The quick test: give a frontier
   model the Problem section cold; if it reliably produces the prescription,
   the pattern fails. Vendor docs may still appear in provenance — as
   corroboration for a prescription that goes beyond them, never as its
   source of novelty. The rejection phrase here is *true but already in the
   weights*.
4. **Is the evidence real, current, and honestly scoped?** Every provenance
   entry must be checkable: `primary-docs` and `research` entries have
   working, relevant URLs; `practitioner` entries have working URLs and a
   stated adoption signal a reviewer can observe (engagement, reproduction in
   unrelated public work); `field-report` entries describe what was observed
   concretely enough that a reader could attempt the same observation, and
   are marked "internal; not publicly reproducible" when they are — internal
   reports corroborate but never anchor `proven` alone. Freshness outweighs
   mass: evidence predating a material change in what it describes stops
   counting. Maturity must match the evidence per the thresholds in
   [ARCHITECTURE.md](ARCHITECTURE.md#the-pattern-schema); conflicting
   credible evidence *requires* `contested` with the disagreement presented
   in Tradeoffs. Applicability must match it too: single-domain evidence
   means `domain-specific` with the domain named — claiming `general`, like
   claiming `proven`, is a reviewable assertion, not a self-assessment.
5. **Are the costs stated?** A Tradeoffs section that says "none significant"
   is a rejection. If the author can't name what the pattern costs, the
   pattern isn't understood well enough to publish.
6. **Is the voice safe — and the substance?** Descriptive voice throughout,
   no instructions aimed at the reading agent, no fetch-and-follow
   constructions, no code presented as something to execute rather than
   something to learn from — per the trust model in
   [ARCHITECTURE.md](ARCHITECTURE.md#trust-model). Voice violations are fixed
   or rejected, never waived: this is the injection-surface control, and one
   waived exception sets the precedent that ends it. Voice covers phrasing
   only, so the reviewer also owns the substance check: a prescription or
   Example the reviewer cannot personally vouch as safe for consumers to
   pattern-match into real systems is rejected, however well it reads.

## Review process

- **One maintainer approval merges** a pattern PR, with a required 72-hour
  open-comment window from PR readiness (CI green) — long enough for other
  maintainers and the public to object, short enough not to strangle
  contribution. Two exceptions in opposite directions: typo-level fixes skip
  the window; changes to `safety`-category patterns and anything touching the
  trust model or schema require a second maintainer approval, because errors
  there compound into consumers (during the single-maintainer bootstrap
  phase, the substitute in [Roles](#roles) applies).
- **Reviews leave a trail.** Acceptance and rejection reasons live on the PR,
  stated against the six acceptance criteria. "Not a good fit" without a
  criterion is not a valid rejection.
- **Author responsiveness:** PRs with unanswered review feedback for 30 days
  are closed as stale — reopenable any time; closure is queue hygiene, not
  judgment.

## Disputes

- **Content disputes** (is this pattern right?) are resolved by evidence,
  through the schema. The corpus's native answer to a credible disagreement
  is `maturity: contested` with both sides in Tradeoffs — a dispute is
  content. Only when a maintainer judges one side's evidence decisively
  stronger does the pattern stay/become `proven` or get superseded.
- **Process disputes** (was this rejection fair?) go to the maintainer group;
  any contributor may request a second maintainer's review of a rejection.
  The second review is final. If maintainers deadlock, the status quo holds —
  the corpus defaults to *not* publishing.
- **No relitigating by attrition.** A rejected pattern may return only with
  materially new evidence. Reviewers may close repeat submissions citing the
  original rejection.

## Freshness and staleness

Agentic best practice moves fast; a pattern corpus that doesn't plan for its
own decay becomes misinformation with provenance. Mechanics (supersession,
retirement, hash-pin behavior) are defined in
[ARCHITECTURE.md](ARCHITECTURE.md#versioning-and-stability); the process:

- **Anyone can report staleness** by opening an issue naming the pattern and
  the evidence that undermines it. A staleness issue is triaged like a PR:
  against the evidence, in the open.
- **Scheduled review sweeps:** every three months, maintainers re-examine
  every `active` pattern whose `updated` date is older than six months, asking
  one question — *would this merge today?* Outcomes: reconfirm (bump
  `updated` with a provenance addition if new supporting evidence exists),
  demote maturity, supersede, or retire. Sweeps are tracked in a public
  issue per sweep. The cadence is aggressive on purpose: a corpus that lives
  at the leading edge decays faster than one that curates settled consensus,
  and the beyond-the-model test cuts both ways — a pattern that was beyond
  the model at merge time gets absorbed into later model generations, at
  which point it retires as *absorbed*, its job done.
- **Event-driven sweeps:** a major release or behavior change in a
  widely-used platform (model family, agent runtime, protocol) triggers a
  targeted sweep of the categories and domains it touches, without waiting
  for the calendar.
- **Supersession is preferred over heavy revision.** When the prescription
  itself changes, publish a successor and flip the old pattern to
  `superseded` rather than rewriting it in place — consumers who reviewed
  the old prescription keep a stable object to point at, and the corpus
  keeps an honest record of what it used to recommend.
- **Consumers pinned to a hash are never broken** by any of this: their bytes
  don't change. Freshness signals (`status`, `superseded_by`, `updated`)
  reach them when they next look at HEAD, and security advisories reach them
  when it matters urgently.

## Security and takedown

Suspected malicious or injection-carrying content is a security report, not a
PR comment: report privately via GitHub's security-advisory flow for this
repository. Confirmed reports get the takedown treatment defined in the
trust model — retire or revert at HEAD, publish an advisory naming affected
slugs, content hashes, and corpus versions — typically within 72 hours of
confirmation. History is not rewritten; the advisory is the signal pinned
consumers act on.

## License

Two licenses, split by what they cover:

- **Pattern content** (`patterns/`, `docs/`): [Creative Commons Attribution
  4.0](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).
  The corpus only matters if it travels — into agent contexts, into rendered
  skills, into commercial products — so the license must permit commercial
  reuse and derivative works with attribution as the only obligation.
  ShareAlike was rejected deliberately: patterns are served into third-party
  agents' contexts and blended into their outputs, and a copyleft obligation
  in that path creates compliance ambiguity that would make cautious
  organizations simply not use the corpus. Attribution is enough: it routes
  reputation back to the corpus, which is the incentive that funds curation.
- **Server code** (`server/`, `schema/`, build tooling):
  [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0). Permissive to
  keep adoption frictionless, chosen over MIT for the explicit patent grant —
  appropriate for infrastructure that organizations embed.

Both license texts live at the repository root (`LICENSE` for Apache-2.0,
`LICENSE-CONTENT` for CC BY 4.0). One boundary clarification: the tool
contracts and schema shapes *described* in `docs/` exist to be implemented —
implementing them creates no derivative work of the documentation, and their
machine-readable form in `schema/` is Apache-2.0 like the rest of the code.

By opening a PR, contributors license their contribution under the license
covering the files it touches. That sentence is the project's entire
contributor agreement.
