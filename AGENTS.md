# agentbranch-patterns — agent notes

Curated corpus of agentic-setup patterns, served via a small MCP server.
Knowledge has one home — read these before changing anything, and put new
knowledge in the doc that owns its subject rather than adding docs:

- [README.md](README.md) — public front door: what/who/how.
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — glossary, pattern schema,
  category taxonomy, tool contracts, index model, trust model, versioning,
  non-goals. The technical constitution; the glossary's terminology is
  binding in all docs, code, and pattern content.
- [docs/GOVERNANCE.md](docs/GOVERNANCE.md) — contribution flow, acceptance
  bar, review process, disputes, freshness sweeps, licenses.

Decisions that live nowhere else:

- Docs are written as **current state** — no decision-history tables, no
  ADRs, no "we considered X" narratives. When a decision changes, rewrite
  the section and keep the live rationale in place.
- No application code, package scaffolding, or CI config exists yet; the
  first implementation session builds the validator + index builder first
  (they define what a valid pattern is in code), then the server.
- Content is licensed CC BY 4.0 and code Apache-2.0, split by path
  (`patterns/` + `docs/` vs `server/` + `schema/`) — keep new files on the
  right side of that line.
