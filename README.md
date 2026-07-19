# cloud-itonami-lei-2572ibtt8cczw6au4141

> **Independent third-party archive/analysis. Not affiliated with, endorsed by, or sponsored by The Procter & Gamble Company.**

This repository archives publicly published legal/policy documents of
**The Procter & Gamble Company**, with source-url and retrieval-date provenance, per
[ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
and [ADR-2607199960](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607199960-cloud-itonami-lei-top10-universe-expansion.edn)
(`cloud-itonami-lei-corporate-tos-catalog`, `com-junkawasaki/root`). It is a read-only
reference/archive repository — it does not act, propose, or execute anything on the
company's behalf, and is not a governed Advisor/Governor actor.

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

## Design rationale

See ADR-2607110300 in `com-junkawasaki/root` (`90-docs/adr/`) for why this repo exists,
why it is keyed by LEI rather than GTIN or ticker, and why full-text archival (with
provenance) was chosen over excerpt-only storage. See ADR-2607199960 for why this
company was added as part of the top-10-per-industry / stock-index-universe depth
expansion of the catalog (2026-07-19).
