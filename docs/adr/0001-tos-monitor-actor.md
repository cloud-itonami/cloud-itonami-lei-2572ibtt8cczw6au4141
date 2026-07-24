# ADR-0001: ToSMonitor-LLM ⊣ ToSArchiveGovernor -- a governed actor layered on this archive

- Status: Accepted (2026-07-24)
- Related: [`com-junkawasaki/root` ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
  (the archive-only design this repo was created under -- unchanged by this
  ADR); [`com-junkawasaki/root` ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn)
  (the fleet-level pilot decision -- this file is this repo's own
  architecture record, the superproject ADR records the pilot-scope and
  registry bookkeeping context); [`cloud-itonami-commitment-ledger`
  `docs/adr/0001-architecture.md`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger/blob/main/docs/adr/0001-architecture.md)
  (the direct template this build follows).

## Context

This repository archives publicly published legal/policy documents of The
Procter & Gamble Company, per ADR-2607110300 -- a read-only reference,
explicitly not a governed Advisor/Governor actor. That design is correct
for the ~155-repo `cloud-itonami-lei-*` family as a whole.

As a pilot (owner-approved, 2026-07-24 session, see the superproject's own
ADR-2607241900 for the full fleet-level context), THIS ONE repo gains a
governed actor layer on top of the unchanged archive, to test whether the
`build-actor` pattern this workspace already uses extensively elsewhere
(most directly: `cloud-itonami-commitment-ledger`, `cloud-itonami-isic-*`)
is worth extending to the `cloud-itonami-lei-*` family.

## Decision

1. **ToSMonitor-LLM (sealed) ⊣ ToSArchiveGovernor (independent).**
   `tosmonitor.*` namespaces under `src/`, modeled directly on
   `commitledger.*`'s Store/Registry/Advisor/Governor/Phase/Operation/Sim
   shape: a `langgraph-clj` StateGraph, mock-advisor by default, six
   independently re-verified HARD governor checks, phase 0-3 rollout,
   append-only audit ledger.
2. **The archive is never touched.** `blueprint.edn`, `NOTICE`, and
   `80-data/public/tos.journal.edn` keep their exact existing shape and
   meaning. `tosmonitor.store`'s `commit-record!` writes ONLY to this
   actor's own Store (MemStore | DatomicStore) -- never to
   `tos.journal.edn`. A human archivist or a future automated pipeline
   decides whether to fold this actor's proposals into the real archive.
3. **One actuation, `:tos/change-proposal`, permanently non-autonomous.**
   Every run -- whether the candidate text matches the baseline or
   diverges from it -- carries `:stake :actuation/archive-update` and is
   excluded from every phase's `:auto` set. Recording any verdict on what
   a company's legal terms say is always a human archivist's call.
4. **Six HARD checks**, every one re-verified independently of the
   advisor's own proposal:
   - `grounding-violations` -- every cited excerpt must be an exact,
     verbatim substring of the candidate's own text. No hallucinated
     quotes.
   - `provenance-incomplete-violations` -- full-text/source-url/
     retrieved-at/sha256 must all be present.
   - `sha256-mismatch-violations` -- ground-truth SHA-256 recompute
     (`:clj`-only in V1; see `tosmonitor.registry` ns docstring for the
     honest `:cljs` scope boundary).
   - `retrieved-at-not-advancing-violations` -- refuses input staler than
     what is already archived.
   - `doc-type-unknown-violations` -- must be a member of this archive's
     own doc-type vocabulary.
   - `source-domain-mismatch-violations` -- **this actor's own
     distinctive check**: the candidate's source-url domain must match
     the company's own `blueprint.edn` website, independently
     re-verified from the store, never the advisor's self-report. Guards
     the risk specific to an LEI-keyed independent-archive family:
     misattributing a document to the wrong company.
5. **Mock-advisor only.** `clojure -M:dev:run` and the test suite never
   make a live network call. `tosmonitor.advisor/llm-advisor` exists as a
   written, swappable seam but is not invoked anywhere in this pilot.

## Consequences

- (+) This repo now has a real, tested, governed actor, proving the
  `build-actor` pattern applies to the `cloud-itonami-lei-*` archive shape
  without guessing.
- (+) `tos.journal.edn`'s existing quad-log format and ADR-2607110300's own
  provenance discipline are untouched and, via `sha256-mismatch-
  violations`/`grounding-violations`, now enforced structurally for any
  candidate this actor is asked to check.
- (-) No live re-fetch of the company's current ToS page exists yet -- a
  human or a future scheduled job must still supply the `:candidate`
  input.
- (-) `sha256-mismatch-violations`'s ground-truth recompute is `:clj`-only;
  a `:cljs` build trusts the candidate-supplied hash for that one check.

## Run

```bash
clojure -M:dev:run     # walk a clean lifecycle + all six HARD-hold checks + a phase-0 hold + a backend swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

## Alternatives considered

| Option | Verdict | Reason |
|---|---|---|
| Write proposals directly into `tos.journal.edn` | Rejected | Would conflate a governed actor's DRAFT verdicts with the archive-of-record ADR-2607110300 defines |
| Auto-eligible "no change detected" reconfirmation path | Deferred | A single, always-escalate actuation is the right scope for a first pilot proving the actor SHAPE itself |
| Portable (`:cljs`) SHA-256 recompute | Deferred | Web Crypto's digest API is Promise-based; no synchronous portable option without a bundled JS dependency this fleet's `.cljc` actors avoid |
