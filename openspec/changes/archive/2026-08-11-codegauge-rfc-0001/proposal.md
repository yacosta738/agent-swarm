# Proposal: CodeGauge 0.1.0 deterministic JaCoCo engine

## Intent

Ship RFC-0001's standalone Rust CLI for JaCoCo CRAP evidence, not policy/workflow. `codegauge` is an independent repo/submodule, separate from `agent-harness`; MVP integration is its CLI/result contract.

## Scope

### In Scope
- Six crates (`core/model/application/cli/provider-jacoco/conformance`), Rust/Cargo lockfile.
- `java-jacoco-v1`, core `crap-original-v1`, models/schemas, SHA-256, bounded parser, fixtures/golden/CLI tests, `analyze|profiles|version` JSON.

### Out of Scope
- No `agent-harness` or spec/config change; adapter invokes this CLI.
- Languages/providers, policy/thresholds, mutation/DRY, auto-detection, build/test/report execution, downloads/network, LLMs, plugins, project config.

## Capabilities

### New Capabilities
- `codegauge-crap-core`: measurements/CRAP.
- `codegauge-jacoco`: extraction/correlation.
- `codegauge-cli-contract`: JSON, provenance/errors.

### Modified Capabilities
- None; preserve harness capabilities.

## Approach

Boundary: model/core → provider → application → CLI. Specs/design finalize schemas, parser settings, vectors.

- **JaCoCo:** method `COMPLEXITY=missed+covered`; `INSTRUCTION=covered/total`; ignore aggregates/optional counters.
- **Identity:** `(VM class, method, JVM descriptor)` plus `symbol_id`; overloads distinct; duplicate identities reject; include reported methods.
- **Incomplete:** omit invalid/missing required counters; remaining valid symbols → `PARTIAL`/exit 6; none → incompatible/exit 6.
- **Provenance/profile:** never invent paths; keep metadata; hash exact bytes; unknown versions absent; semantic changes require a profile.
- **Determinism/time:** sort by id, POSIX paths, stable finite numbers; RFC3339 UTC timestamp is non-metric and excluded from golden comparison.
- **Errors:** exits `0/2/3/4/5/6/10` = complete/CLI/missing/unsupported/invalid XML/partial-incompatible/internal; JSON stdout, diagnostics stderr.
- **Security/toolchain:** bounded XML, no DTD/entities/network, 64 MiB, counts; pin Rust/Cargo `1.97.1`/lockfile; claim it MSRV unless design lowers it.

## Reference-informed decisions

Non-normative: [crap4java](https://github.com/unclebob/crap4java), [crap4go](https://github.com/unclebob/crap4go), [crap4clj](https://github.com/unclebob/crap4clj). No code copied, dependency added, or reference runner executed.

- **Adopt:** formula edges (full coverage → `CC`; zero coverage; unresolved coverage → indeterminate/no score) and structured diagnostics for unmatched/ambiguous/incomplete observations.
- **Future pattern:** path suffix/range mapping belongs to versioned providers, not core heuristics.
- **Reject:** toolchain execution/mutation, thresholds/policy, and name/line joins when a JVM descriptor exists.
- Scope remains Java/JaCoCo artifact-first; `agent-harness` is unchanged.

## Affected Areas

- `codegauge/` — schemas, fixtures, tests.
- `openspec/changes/codegauge-rfc-0001/` — downstream artifacts.
- `agent-harness/`, `openspec/config.yaml`, `openspec/specs/` — no changes.

## Risks

- **Med:** semantic/timestamp drift → specs, diagnostics, goldens.
- **Med:** XML/toolchain drift → limits, negative tests, lockfile.
- **High:** review size → slices: workspace/core; provider/fixtures; CLI/conformance.

Forecast: ~900–1,300 lines; target ≤400 per slice.

Decision needed before apply: Yes
Chained PRs recommended: Yes
400-line budget risk: High

## Rollback Plan

Revert slices or reset `codegauge` to `d53bdb7`; publish no binary/adapter. Leave harness/global artifacts untouched.

## Dependencies

- Pinned Rust/Cargo plus XML/CLI/serialization/hashing.

## Success Criteria

- [ ] `verify-report.md` records toolchain/isolation, formula, schemas/goldens, hashes, exit/stdout/stderr, and fmt/test/clippy evidence.
- [ ] `qa-report.md` records valid/full, overload, incomplete, malformed, missing, unsupported, hostile XML runs, outputs, hashes, timestamp handling, environment, untested targets, findings, and verdict.
- [ ] Output has no quality PASS/FAIL and uses versioned result/error contracts only.
