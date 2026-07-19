# agentbranch-patterns

A curated, open-source knowledge base of **patterns for agentic setup** — how to
design tools, manage context, author skills, run evals, compose sub-agents, and
keep agents safe — served to any MCP-capable agent through a small, stable MCP
server.

> **Status: design phase.** The pattern schema, tool contracts, and governance
> model are defined (see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and
> [docs/GOVERNANCE.md](docs/GOVERNANCE.md)). The server implementation and the
> first pattern corpus are in progress. Usage instructions below describe the
> committed design.

## What this is

Building agents produces hard-won, transferable knowledge: how to word a tool
description so a model picks the right tool, when to split work across
sub-agents, how to structure an eval that catches real regressions. The leading
edge of that knowledge lives in practitioners' posts, threads, and repos —
moving faster than official documentation and far faster than model training
data — with no provenance attached and no stable way to cite or serve it.

agentbranch-patterns exists to solve one concrete problem first: people
authoring and validating skills with
[agent.branch](https://github.com/CrowBe/agentbranch) need current, citable
agentic-setup knowledge to build against. The corpus is that source, and it is
public — the same knowledge serves any MCP-capable agent or human reader:

- **Patterns, not prose.** Each entry is a structured document with a fixed
  arc — problem → pattern → example → tradeoffs → anti-patterns — served whole,
  because the tradeoffs and anti-patterns are the part most collections omit
  and agents most need.
- **Leading edge, not received wisdom.** The corpus deliberately excludes what
  frontier models already reliably know: restatements of vendor documentation
  and long-settled practice don't merge. The target is fresh practitioner
  knowledge that carries an observable validity signal — adoption, engagement,
  reproduction in other people's public work.
- **Provenance, always.** Every pattern cites its evidence: primary
  documentation, published research, practitioner publications with observable
  adoption, or validated field experience. Every pattern declares its maturity
  honestly — `proven`, `emerging`, or `contested` — and maturity is
  freshness-weighted: recent evidence outweighs accumulated stale citations,
  because the field moves faster than bibliographies do.
- **Scoped honestly.** Every pattern declares whether it is generically
  useful or specific to a domain of agentic work. As agents cross from
  software engineering into other knowledge work, "works in coding agents"
  stops being evidence of "works everywhere" — the schema keeps the two
  claims apart.
- **Git-native and hash-pinnable.** The repository *is* the database. Patterns
  are markdown files, contributions are pull requests, CI validates the schema,
  and every pattern carries a content hash so consumers can pin exactly what
  they reviewed.
- **Stateless server.** Search is lexical (BM25 over title, summary, tags,
  body) against an index precomputed at build time. No runtime database, no
  embedding pipeline, nothing to operate but the process itself.

The content is the product. The server is a thin, boring way to reach it.

## Who it's for

- **Skill builders.** The motivating consumer:
  [agent.branch](https://github.com/CrowBe/agentbranch) users authoring and
  validating skills draw on the corpus — directly, and through agent.branch's
  own use of the public tools — for current, cited answers to setup questions.
- **Agents mid-task.** Documents are structured so an agent scaffolding a new
  tool, writing a skill, or designing an eval can query the knowledge base and
  get ranked, cited answers it can act on immediately.
- **Agent builders.** Humans designing agentic systems can browse the same
  corpus directly on GitHub — every pattern is a readable document.

agent.branch has no privileged integration path: it builds on the corpus
through the same three MCP tools everyone else uses.

## Using it from an agent

The server exposes three MCP tools — deliberately no more:

| Tool | Purpose |
|---|---|
| `search_patterns` | Lexical search over the corpus; returns ranked summaries with ids |
| `get_pattern` | Fetch one full pattern document by id, with content hash |
| `list_categories` | Enumerate the category taxonomy with counts |

A typical agent interaction: `search_patterns` with a task-shaped query
("how should tool errors be worded"), scan the returned summaries, then
`get_pattern` for the one or two worth reading in full. Exact input/output
contracts, error shapes, and stability guarantees are specified in
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#tool-contracts).

### Adding the server

Once released, the server will be installable as a standard MCP server
(stdio and streamable HTTP). For Claude Code, that will look like:

```sh
claude mcp add agentbranch-patterns -- npx -y agentbranch-patterns
```

Other MCP clients add it through their usual server configuration. The server
takes no credentials and stores no state; it serves a versioned, read-only
corpus.

### Offline / portable use

The same content renders to a skill folder (per the
[Agent Skills](https://code.claude.com/docs/en/skills) open standard:
`SKILL.md` plus reference documents) for agents that work offline or prefer
files over tools. Rendering is a pure function of this repository — the MCP
server and the skill are two views of one corpus, never two sources of truth.

## Reading a pattern

Every pattern is a markdown file with YAML frontmatter, under
`patterns/<category>/<slug>.md`. The frontmatter carries identity, category,
tags, aliases, applicability, maturity, lifecycle status, and provenance; the
body follows a fixed
section order (Problem, Pattern, Example, Tradeoffs, Anti-patterns). The full
schema is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#the-pattern-schema).

Trust matters here: patterns are text that gets injected into third-party
agents' contexts, which makes a malicious pattern a prompt-injection vector.
The threat model and mitigations — review gate, descriptive-voice content
rules, hash pinning, provenance, takedown-by-revert — are specified in
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md#trust-model) and enforced through
the process in [docs/GOVERNANCE.md](docs/GOVERNANCE.md).

## Contributing a pattern

Contribution is open; acceptance is curated. In short:

1. Check the pattern doesn't already exist (`search_patterns`, or browse
   `patterns/`).
2. Write it against the schema — all required frontmatter fields, all five
   body sections, provenance for every load-bearing claim.
3. Open a pull request. CI validates the schema; maintainers review for
   evidence, scope, and voice.

The acceptance bar (what counts as evidence, what's in scope, what gets a
pattern rejected) and the review process are defined in
[docs/GOVERNANCE.md](docs/GOVERNANCE.md). Read it before writing — the bar is
deliberately high, and the two most common rejections are "true but not a
pattern" and "true but already in the weights" (knowledge frontier models
apply reliably without help).

## License

Pattern content is licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); server code (when
it lands) under Apache-2.0. Rationale in
[docs/GOVERNANCE.md](docs/GOVERNANCE.md#license).

## Related projects

- [CrowBe/agentbranch](https://github.com/CrowBe/agentbranch) — skill
  authoring and validation tool; the motivating consumer of this knowledge
  base.
- [CrowBe/agentbranch-tap](https://github.com/CrowBe/agentbranch-tap) — public
  distribution tap for agent.branch.

This repository is standalone: it does not depend on either sibling, and they
consume it only through the public MCP tools.
