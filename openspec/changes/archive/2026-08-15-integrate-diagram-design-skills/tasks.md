# Tasks: Integrate Diagram Design Skills

## Review Workload Forecast

| Field | Value |
|---|---|
| Estimated changed lines | >400; roughly 3,000–10,000 from the snapshot; measure per slice |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Delivery strategy | approved dependency-ordered feature-branch chain |
| Chain strategy | feature-branch-chain |

REVIEW_WORKLOAD_LINES: >400 (estimate 3,000–10,000; confirm per slice)
REVIEW_WORKLOAD_DECISION: Yes — guard approval required before every apply slice
REVIEW_WORKLOAD_EXCEPTION: None — no size exception without explicit maintainer approval
REVIEW_WORKLOAD_ROLLBACK: Revert only the active family; restore its manifest/docs baseline and preserve `opencode.json`

Decision needed before apply: No — maintainer selected stacked commits/PRs
Chained PRs recommended: Yes — implemented locally as a feature-branch chain
Chain strategy: feature-branch-chain
400-line budget risk: High

### Approved chain metadata

- `trunk`: `main`
- `parent_branch`: `feature/diagram-design-commands`
- `base`: `feature/diagram-design-commands`
- `branch`: `feature/diagram-design-docs`
- `position`: `6/6` for the current apply slice; later slices advance to verify/QA
- `publication`: local stacked commits now; GitHub PR creation/push remains a separate explicit action

### Suggested Work Units (dependency order)

| Unit | Goal | Rollback |
|---|---|---|
| 1 | `SKILL.md` plus provenance, license, notices, and manifest identity | Remove only identity/metadata files |
| 2 | Complete `references/**` snapshot and hashes | Remove only `references/**` and entries |
| 3 | Complete `scripts/**` snapshot and hashes | Remove only `scripts/**` and entries |
| 4 | Complete `assets/**` snapshot and hashes | Remove only `assets/**` and entries |
| 5 | Three adapters with their explicit contracts | Revert only `commands/diagram-*.md` |
| 6 | Inventory docs and smoke evidence | Revert docs/evidence; keep prior config |

Before each `sdd-apply` slice, enforce the selected chain/size guard and confirm reviewability.

## Phase 1: infrastructure

- [x] 1.1 Create `skills/design/diagram-design/` from commit `a5e3978088cf89c7caff5c20cabd99fbc2a301de` / version `2.3.5`; preserve `SKILL.md` identity.
- [x] 1.2 Add `PROVENANCE.md`, `LICENSE`, `THIRD-PARTY-NOTICES.md`, and `SNAPSHOT-MANIFEST.sha256` with source, pin, paths, sizes, and SHA-256 hashes.
- [x] 1.3 Copy complete `references/**` as its own slice; preserve targets and update the manifest; prune only explicitly excluded upstream files.
- [x] 1.4 Copy complete `scripts/**` as its own slice, including both extractors and `self_check.py`; update hashes and runtime expectations.
- [x] 1.5 Copy complete `assets/**` as its own slice; finalize hashes and the excluded-file record.

## Phase 2: implementation

- [x] 2.1 Create `commands/diagram-export.md` for HTML/SVG/PNG, safe `$ARGUMENTS`, explicit output, and unavailable Playwright/Chromium status.
- [x] 2.2 Create `commands/diagram-import-drawio.md` with pinned extractor-first untrusted-data flow, dials, and fidelity ledger.
- [x] 2.3 Create `commands/diagram-import-mermaid.md` with the same safety, dials, ledger, no-fetch, and ambiguity contract.
- [x] 2.4 Update `README.md`, `AGENTS.md`, and `.agents/skill-registry.md` consistently; do not modify `opencode.json`.

## Phase 3: testing

- [x] 3.1 Run deterministic frontmatter/path, relative-link, manifest/pin, docs, and unchanged-`opencode.json` checks.
- [x] 3.2 Run `drawio_extract.py`, `mermaid_extract.py`, and `self_check.py` on safe fixtures; record hostile input and missing Python as `UNAVAILABLE`/`BLOCKED`.
- [x] 3.3 Smoke all command contracts: valid output plus invalid args/paths/flags/formats, malformed sources, fidelity ledger, and missing Playwright/Chromium without writes or installation.
- [x] 3.4 Report manual smoke evidence only: no test runner, TDD claim, or product-acceptance claim.

**Dependencies:** `1.1→1.2→1.3→1.4→1.5→2.1–2.4→3.1–3.4`; import adapters depend on `scripts/**`, export depends on local renderer probes, and docs depend on final contracts. **Done:** hashes/notices agree with the pin; docs agree on path/commands/prerequisites; smoke checks pass or record unavailable capability. Optional tools are never installed implicitly.
