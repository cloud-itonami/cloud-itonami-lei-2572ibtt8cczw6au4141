# cloud-itonami-lei-2572ibtt8cczw6au4141

> **Independent third-party archive/analysis. Not affiliated with, endorsed by, or sponsored by The Procter & Gamble Company.**

This repository archives publicly published legal/policy documents of
**The Procter & Gamble Company**, with source-url and retrieval-date provenance, per
[ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
and [ADR-2607199960](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607199960-cloud-itonami-lei-top10-universe-expansion.edn)
(`cloud-itonami-lei-corporate-tos-catalog`, `com-junkawasaki/root`). The archive itself
(`blueprint.edn`, `NOTICE`, `80-data/public/tos.journal.edn`) is unchanged by anything
below — it stays a read-only reference archive; this repo does not act, propose, or
execute anything on the company's behalf using the archive data itself.

**As of 2026-07-24 this repo ALSO carries a governed actor layered on top of that
archive** (`src/tosmonitor/*`, ToSMonitor-LLM ⊣ ToSArchiveGovernor), added as an
explicit, owner-approved, single-repo PILOT
([ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn),
repo-local record: [`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md)).
This pilot does NOT change the archive's own read-only nature, and it does NOT extend
to any other `cloud-itonami-lei-*` repository — the other ~154 repos in this family
remain plain archives exactly as ADR-2607110300 describes. See `## Governed actor
(pilot)` below.

## Company identity

- **Legal name**: The Procter & Gamble Company
- **LEI (ISO 17442)**: [2572IBTT8CCZW6AU4141](https://search.gleif.org/#/record/2572IBTT8CCZW6AU4141) (GLEIF-verified)
- **Jurisdiction**: US-OH
- **Website**: https://us.pg.com

## Contents

- `80-data/public/tos.journal.edn` — EDN quad-log of archived documents, each entry
  carrying `:tos/full-text`, `:tos/source-url`, `:tos/retrieved-at`, `:tos/sha256`,
  `:tos/doc-type`, and a `:tos/supersedes` chain for future revisions.
- `NOTICE` — copyright/attribution statement for the archived third-party text.
- `blueprint.edn` — machine-readable company identity record.

## Governed actor (pilot)

**ToSMonitor-LLM ⊣ ToSArchiveGovernor**, built on this workspace's
[`langgraph-clj`](https://github.com/com-junkawasaki/langgraph-clj) StateGraph runtime,
modeled directly on
[`cloud-itonami-commitment-ledger`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger)'s
Store/Registry/Advisor/Governor/Phase/Operation/Sim shape.

Given a candidate ToS/legal-document snapshot, the actor proposes whether it materially
diverges from the archived baseline above, and drafts a change summary — but **it never
writes to the archive itself.** `tosmonitor.store`'s `commit-record!` only writes to
this actor's own Store (an in-process ledger), never to `80-data/public/
tos.journal.edn`. A human archivist decides whether to fold a proposal into the real
archive.

**Single actuation, `:tos/change-proposal`, never autonomous.** Every run — whether the
candidate matches the baseline or diverges from it — carries `:stake :actuation/
archive-update` and is permanently excluded from every rollout phase's auto-commit set.
Six HARD governor checks, each independently re-verified rather than taken on the
advisor's self-report:

| Check | Guards against |
|---|---|
| `grounding-violations` | a cited excerpt not actually present in the candidate's own text (hallucinated quotes) |
| `provenance-incomplete-violations` | a candidate missing full-text/source-url/retrieved-at/sha256 |
| `sha256-mismatch-violations` | a candidate's self-reported sha256 not matching its own content (ground-truth recompute, `:clj`-only in V1) |
| `retrieved-at-not-advancing-violations` | proposing a change from input staler than what is already archived |
| `doc-type-unknown-violations` | a doc-type outside this archive's own vocabulary |
| `source-domain-mismatch-violations` | **this actor's own distinctive check** — a source-url that doesn't belong to the archived company (misattribution risk specific to an LEI-keyed independent archive) |

```bash
clojure -M:dev:run     # clean lifecycle + all six HARD-hold checks + a phase-0 hold + a MemStore->DatomicStore swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

`clojure -M:dev:run` and the test suite always use the deterministic mock-advisor — no
live fetch of the company's current ToS page and no live `kotoba-server`/CACAO publish
happen anywhere in this pilot. `tosmonitor.advisor/llm-advisor` exists as a written,
swappable seam but is not invoked. See
[`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md) for the full
design record.

## Design rationale

See ADR-2607110300 in `com-junkawasaki/root` (`90-docs/adr/`) for why this repo exists,
why it is keyed by LEI rather than GTIN or ticker, and why full-text archival (with
provenance) was chosen over excerpt-only storage. See ADR-2607199960 for why this
company was added as part of the top-10-per-industry / stock-index-universe depth
expansion of the catalog (2026-07-19).
