# Archive Report: `codegauge-rfc-0001`

## Archive decision

- **Change:** `codegauge-rfc-0001`
- **Date:** 2026-08-11
- **Artifact mode:** OpenSpec filesystem
- **Technical verification:** `PASS WITH WARNINGS` (`verify-report.md`)
- **Acceptance QA:** `PASS WITH WARNINGS` (`qa-report.md`)
- **Archive decision:** **PASS**
- **Product/release readiness:** **Not claimed**

The two-report acceptance gate is satisfied. Both required reports exist; verification and QA are
`PASS WITH WARNINGS`; and no unresolved `CRITICAL`, `P0`, or `P1` finding remains. QA F-01 and F-02
are resolved. F-03 is a visible, accepted `P2` limitation concerning the path-only error for an
input rejected before the 64 MiB read/hash boundary.

## Configured gate evaluation

`openspec/config.yaml` was read before this archive. Its archive policy requires both reports and
blocks `FAIL`, unresolved `CRITICAL`/`P0`/`P1`, acceptance-relevant `BLOCKED`, and
acceptance-relevant `NOT TESTED` results. The configured non-runtime exception is limited to
documentation/configuration and was **not invoked**.

The QA report records `NOT TESTED` scope for the independent JSON Schema instance validator,
dependency scans, coverage measurement, cross-platform release builds/checksums, and the future
`agent-harness` adapter (plus future mapping/locale/UI categories). These are either explicitly
out of scope, unavailable release/tooling assurance, or not applicable to the selected standalone
CLI acceptance target. The selected acceptance scenarios have no acceptance-relevant `BLOCKED` or
`NOT TESTED` result; therefore the configured `acceptance_relevant_NOT_TESTED` blocker is not
triggered. The original QA verdict and every untested limitation remain unchanged in the archived
report.

Open findings retained visibly:

| Finding | Severity | Status | Archive treatment |
|---|---|---|---|
| F-03 oversized input has no SHA-256 | P2 | Open, accepted limitation | Warning preserved; no code/spec change made |
| F-04 independent JSON Schema instance validator | P2 | Open, validation gap | Not tested preserved; no release claim |
| F-05 cross-platform release/checksums | P2 | Open, release gap | Not tested preserved; no release claim |
| F-06 coverage/dependency scans | P3 | Open, tooling gap | Not tested preserved |
| F-07 harness adapter/E2E consumption | P3 | Out of scope | Not tested preserved; future change |

## Canonical spec synchronization

No canonical specs were modified.

The active change contains a single consolidated root `spec.md`; it does not contain the
convention's `specs/{domain}/spec.md` delta tree or explicit `ADDED`/`MODIFIED`/`REMOVED` sections.
The existing `openspec/specs/` tree contains harness capabilities and no exact CodeGauge target.
Splitting the cross-cutting normative requirements into new canonical files would duplicate or
repartition RFC 2119/scenario semantics and invent a new source-of-truth layout. Per the requested
safe-split rule, the full CodeGauge spec is preserved in the archive instead of creating unrelated
canonical specs.

| Canonical path | Action | Details |
|---|---|---|
| `openspec/specs/` | Unchanged | No exact CodeGauge canonical target and no safe delta split |

Protected surfaces were not modified: `agent-harness/`, `openspec/config.yaml`, existing harness
canonical specs, `openspec/changes/deterministic-quality-runners-fsm/`, the `codegauge` production
worktree, and the RFC at `tmp/RFC-0001 — CodeGauge_ Deterministic Multi-Language Code Metrics Engine.md`.

## Archive contents

The complete active change folder, including all reports and apply progress, is moved to:

`openspec/changes/archive/2026-08-11-codegauge-rfc-0001/`

Expected preserved artifacts:

- `exploration.md`
- `proposal.md`
- `spec.md`
- `design.md`
- `tasks.md` — 19/19 checklist items complete
- `apply-progress.md`
- `verify-report.md`
- `qa-report.md`
- `state.yaml`
- `archive-report.md`

No separate artifact-index file exists under `openspec/`; `state.yaml` is updated as the lifecycle
index before the folder move:

```yaml
change: codegauge-rfc-0001
current_phase: archive
completed: [explore, propose, spec, design, tasks, apply, verify, qa, archive]
next: none
updated: 2026-08-11
```

## Archive verification

- Main canonical specs remain unchanged: **PASS**
- Archive destination exists with the complete change folder: **PASS**
- All required reports and `apply-progress.md` are preserved: **PASS**
- Active `openspec/changes/codegauge-rfc-0001/` is absent after the move: **PASS**
- Unrelated active change and protected paths remain untouched: **PASS**

No production code, branch, commit, push, PR, release artifact, cross-platform claim, or harness
adapter claim was created by this archive phase.

## SDD cycle status

The OpenSpec lifecycle for `codegauge-rfc-0001` is complete and archived. Future work for
independent schema validation, dependency scans, coverage policy, cross-platform release evidence,
or harness consumption requires a separate, explicitly scoped change.
