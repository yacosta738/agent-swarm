# Design: CodeGauge RFC-0001

## Technical Approach

Independent Rust 1.97.1 workspace: compile-time JaCoCo, one core-owned CRAP formula, versioned JSON
CLI. `model` owns contracts; `core` domain; `application` ports/orchestration; provider/CLI adapters.
No harness or policy.

## Workspace and Decisions

Workspace pins resolver 3, edition 2024, lock and `rust-toolchain.toml` `1.97.1`. Production deps:
`serde`/`schemars`, `quick-xml`, `sha2`, `clap`, `serde_json`; `proptest`/`assert_cmd` test-only.

| Crate | Responsibility | Depends on |
|---|---|---|
| `codegauge-model` | canonical IDs/DTOs/profiles/provenance/schemas | serde, schemars |
| `codegauge-core` | validated sole `CRAP=CC²(1-coverage)³+CC`; no I/O | model |
| `codegauge-application` | orchestration, registry, reader/hash, summaries/errors | model, core, sha2 |
| `codegauge-provider-jacoco` | bounded XML adapter/correlation | model, application, quick-xml |
| `codegauge-cli` | commands/wiring/JSON/exits | application, provider-jacoco, model, clap, serde_json |
| `codegauge-conformance` | shared conformance vectors/tests | model, core, application, provider-jacoco; test deps |

Providers point inward; core never sees providers. Compile-time registration avoids dynamic ABI risk.

## Ports and Data Flow

```rust
pub trait MetricProvider {
    fn descriptor(&self) -> ProfileDescriptor;
    fn collect(&self, request: CollectionRequest) -> Result<ProviderObservations, ProviderError>;
}
pub trait ArtifactReader { fn read(&self, path: &Path) -> Result<Artifact, ArtifactError>; }
pub fn calculate_crap(input: CrapInput) -> Result<CrapScore, MetricError>;
```

`Artifact` = exact bytes/path/length/lowercase SHA-256; `SymbolObservation` = identity plus
measurements. `Analyzer` selects, loads once, collects, invokes core, sorts, summarizes and emits.

```text
CLI -> Analyzer -> ArtifactReader -> JaCoCoProvider -> observations -> core -> Result JSON
Reader/Provider -> typed AnalysisError -> CLI -> error JSON(stdout) + diagnostic(stderr) -> exit
```

## JaCoCo Parser and Semantics

Streaming `quick-xml` over UTF-8 rejects `DOCTYPE`, entities, unsupported encoding, malformed XML,
excessive nesting and external resolution; no network. Limits: 64 MiB, depth 128, 100,000
classes/methods, 16 counters/method, bounded text/attrs, count `<=1e9`.

Require direct method `COMPLEXITY` and `INSTRUCTION`: `CC=missed+covered`, coverage
`=covered/(covered+missed)`. Ignore other/aggregate counters; reject non-decimal, negative,
overflowing or zero-denominator counts. Identity is
`java:{class VM name}#{method name}{JVM descriptor}`; overloads stay distinct; duplicate classes,
methods or counters are `INVALID_INPUT`. Missing evidence stays unknown, is omitted with a bounded
diagnostic, and yields `PARTIAL`/exit 6 if valid symbols remain, otherwise incompatible/exit 6.
Include all reported constructors, `<clinit>`, synthetic, bridge, anonymous and lambda methods;
infer no generated status or repository path.

## JSON v1, Determinism, and CLI

Rust types author `codegauge-result-v1`/`codegauge-error-v1` schemas. Result fields are
`schema/tool/profile/analysis/summary/symbols/provenance`; errors are
`schema/tool/code/message/details`. Field order and `symbol.id` sorting are normative. Numbers use
round-half-even to 12 decimals; strip trailing zeroes and render `-0` as `0`; a canonical writer
enforces this over `serde_json`. Provenance includes profile/provider semantics, display path,
exact-byte hash and RFC3339 UTC timestamp ending `Z`; unavailable values are absent. Mask timestamp
only in goldens.

Commands: `analyze --profile --input --format json`, `profiles`, `version`; no MVP text mode.
`analyze` writes one result or error document to stdout; diagnostics only stderr. Exits: `0` complete,
`2` CLI, `3` missing/unreadable, `4` unsupported, `5` invalid/security, `6` partial/incompatible,
`10` internal. Never emit PASS/FAIL.

## Reference-informed decisions

Verified refs: [`unclebob/crap4java`](https://github.com/unclebob/crap4java),
[`unclebob/crap4go`](https://github.com/unclebob/crap4go), [`unclebob/crap4clj`](https://github.com/unclebob/crap4clj)
(README/spec/formula/coverage code). Evidence only; no copied code or dependencies.

- **Adopt:** unknown coverage is N/A/omitted, never zero; no fabricated CRAP. Formula edges are
  full=`CC`, zero=`CC²+CC`, partial=normal formula. Missing mappings produce bounded diagnostics.
- **Future:** path/range mapping (`path`, start/end, status) is allowed for providers lacking a
  stronger identity; each semantic change gets a versioned profile. `java-jacoco-v1` consumes only
  pre-existing JaCoCo XML and uses descriptor identity.
- **Reject:** Maven/Go/Clojure/test-runner execution, artifact deletion, dependency installation,
  thresholds/policy, and name/line joins when a JVM descriptor exists. Diagnostics use stderr.

## Decision Log

| ID | Decision | Rationale |
|---|---|---|
| REF-001 | References are semantic evidence only | Clean-room Rust and independent release. |
| REF-002 | Unknown is not zero | Avoid false precision; omit and diagnose. |
| REF-003 | Descriptor identity wins | Prevent overload/name/line collisions. |
| REF-004 | Path/range mapping is future-provider only | Preserve stronger Java identity. |
| NORM-001 | `spec.md` is normative | It resolves RFC/design/task conflicts. |
| BOUND-001 | Artifact-first; no policy/harness integration | Determinism and consumer-owned judgement. |

## Testing and Slices

Fixtures cover valid/full/zero, overload/generated/missing/duplicates, malformed and hostile XML.
Tests cover core edges/monotonicity, provider correlation/security, schema/golden numbers/order, and
CLI JSON/exits/streams/repeatability. Slices <=400: (A) workspace/model/core/schemas, (B)
provider/fixtures, (C) application/CLI/provenance/goldens; focused tests/fmt/clippy and rollback.

Decision needed before apply: Yes
Chained PRs recommended: Yes
400-line budget risk: High

## File Changes

| Path | Action | Responsibility |
|---|---|---|
| `codegauge/Cargo.toml`, `rust-toolchain.toml`, `Cargo.lock` | Create | pinned workspace |
| `codegauge/crates/*/src/` | Create | six crates above |
| `codegauge/schemas/*.json`, `fixtures/jacoco/**`, `tests/golden/**` | Create | contracts/fixtures |
| `codegauge/README.md` | Modify | CLI and boundary documentation |

## Migration / Rollout

No migration. Release from an immutable revision. A future `agent-harness` adapter may invoke the
CLI and translate evidence to policy, but must not import crates, duplicate CRAP or change harness
specs in this change.

## Open Questions

None blocking.
