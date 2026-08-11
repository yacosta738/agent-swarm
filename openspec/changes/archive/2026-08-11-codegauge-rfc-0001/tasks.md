# Tasks: CodeGauge RFC-0001

## Review Workload Forecast

Forecast: 1,150–1,550; High; A→B→C→D→E→F; strategy `single-pr`; execution local.

Delivery: `single-pr` future boundary; local checkout; no branches/commits/PRs unless requested.
Decision needed before apply: No
Chained PRs recommended: Yes
Chain strategy: single-pr
400-line budget risk: High

**Local 400-line guard:** Per slice set `BASE_DIR=$(mktemp -d); rsync -a --exclude=.git --exclude=target codegauge/ "$BASE_DIR/"`; then run `git diff --no-index --numstat "$BASE_DIR" codegauge/ | awk '{a+=$1;d+=$2} END {print "added="a,"changed="a+d}'`; require `changed <= 400`. Over 400: stop.

**Norm:** `spec.md` wins: 64MiB, depth128, 100k classes/methods, 16 counters/method, count≤1e9, JSON-only, round-half-even/12. Align `design.md` before production; timestamp `Z`, mask in goldens.
**References:** `exploration.md` verifies `crap4java`/`crap4go`/`crap4clj`; inspiration only, no copied/executed code.

### Slices

A 180–280 workspace; B 260–380 model; C 300–400 JaCoCo; D 320–400 CLI; E 220–350 conformance; F 120–220 docs.

## Phase 1: Infrastructure (Slice A)

- [x] 1.1 **PRECONDITION:** Compare `openspec/changes/codegauge-rfc-0001/{spec,design}.md`; align depth/limits, JSON-only mode and float rules; done: `design.md` has no conflicting claims before production edits.
- [x] 1.2 **RED:** Add failing workspace/lock/toolchain and six-crate isolation checks under `codegauge/tests/`; done: fails on empty bootstrap.
- [x] 1.3 **GREEN:** Create `codegauge/{Cargo.toml,rust-toolchain.toml,Cargo.lock}` and six crate manifests/targets; done: `cargo metadata --locked`.
- [x] 1.4 **REFACTOR/VERIFY:** Enforce model/core inward dependencies and no core I/O; `cargo fmt --check && cargo test --workspace --locked`.

## Phase 2: Implementation (B–D)

- [x] 2.1 **RED:** Add failing model/core tests for DTO/serde, IDs, profiles, invariants, formula, and monotonicity in `codegauge/crates/codegauge-{model,core}/tests/`.
  Evidence: Existing DTO/schema/formula suites were audited; the new `descriptor_identity_is_a_hashable_join_key` test was observed RED with `E0277`/`E0599` because `SymbolIdentity` lacked `Eq`/`Hash`.
- [x] 2.2 **GREEN:** Implement `codegauge/crates/codegauge-model/src/`, result/error v1 schemas, and the sole formula/validation in `codegauge/crates/codegauge-core/src/`; focused cargo tests.
  Evidence: Added `Eq`/`Hash` to descriptor identity, retained the policy-free DTOs/schemas and sole core formula, then `cargo test -p codegauge-model --locked` passed (3 contract + 1 schema test) and `cargo test -p codegauge-core --locked` passed (2 tests).
- [x] 2.3 **REFACTOR/VERIFY:** Validate schemas and one formula; run focused clippy with `-D warnings`.
  Evidence: Checked-in schema equality test now asserts both `$id` values before comparing generated DTO schemas; `cargo fmt --check` and focused `cargo clippy -p codegauge-model -p codegauge-core --all-targets --locked -- -D warnings` passed. `grep` found the CRAP `powi` implementation only in `codegauge-core/src/lib.rs`.
- [x] 2.4 **RED:** Add JaCoCo identity fixtures/tests for full/zero/partial coverage, missing-as-indeterminate (never zero), overload/generated, invalid/duplicate/hostile/limits, and unresolved mappings. Evidence: five focused integration tests and seven XML fixtures were written before provider production code; the first locked run failed for the expected missing provider/model contract symbols.
- [x] 2.5 **GREEN:** Implement `codegauge/crates/codegauge-provider-jacoco/src/`: bounded XML, exact IDs/counters, partial omission, structured unresolved diagnostics, fatal invalid input, no DTD/entities/network; tests pass. Evidence: pinned `quick-xml = 0.38.3` with default features disabled; focused provider suite passes.
- [x] 2.6 **REFACTOR/VERIFY:** Verify constructors/`<clinit>`/synthetic/bridge/lambda, aggregate ignoring, no provider CRAP, and future path/range ambiguity rejection; provider tests. Evidence: final provider suite, model/core suites, fmt, clippy, bootstrap, and the filtered pre-edit snapshot guard all pass; no application/CLI/conformance behavior was added.
- [x] 2.7 **RED:** Add failing application/CLI tests for SHA-256, deterministic id order vs worst-first display order, summary, timestamp, JSON/stdout/stderr and exits. Evidence: `cargo test -p codegauge-application --locked --test application` failed on the expected missing application contract symbols; `cargo test -p codegauge-cli --locked --test cli` failed on the expected missing exit-code mapping before Slice D production code.
- [x] 2.8 **GREEN:** Implement `codegauge/crates/codegauge-application/src/` reader/hash/registry/join/status and `codegauge/crates/codegauge-cli/src/main.rs`; commands/profiles/version exact and exits map 2/3/4/5/6/10; focused cargo tests. Evidence: application and CLI integration suites passed (3 tests each), including complete/partial/incompatible results, while JaCoCo remains the only wired provider.
- [x] 2.9 **REFACTOR/VERIFY:** Fix fixed-order UTF-8/newline JSON, spec floats, no PASS/FAIL, stderr diagnostics; run CLI integration tests. Evidence: package tests, provider/model/core tests, focused `clippy -D warnings`, `cargo fmt --check`, bootstrap checks, `git diff --check` with no diagnostics, and the Slice D guard passed; final guard was `changed=375 <= 400`.

## Phase 3: Testing (Slice E)

- [x] 3.1 **RED:** Add failing conformance/schema/property/golden tests for full/zero/indeterminate coverage, id/display ordering, unresolved diagnostics, and future path/range ambiguity in `codegauge/crates/codegauge-conformance/tests/`. Evidence: the first meaningful `cargo test -p codegauge-conformance --test conformance` run executed 8 tests with 7 passing and the placeholder golden comparison failing with `left: Object {...}` versus `right: Object {}`.
- [x] 3.2 **GREEN:** Implement vectors/goldens for every fixture, identity, contracts, hashes, floats, ordering, stdout/stderr and errors; `cargo test -p codegauge-conformance --locked`. Evidence: added the conformance matrix and timestamp-masked golden; the locked package run passed 6 integration tests, 0 unit failures, and 0 doc-test failures.
- [x] 3.3 **REFACTOR/VERIFY:** Run `cargo test --workspace --locked`, fmt, and clippy; record commands as evidence, never as pre-executed claims. Evidence: workspace tests, `cargo fmt --all -- --check`, workspace clippy with `-D warnings`, bootstrap checks, `git diff --check`, and the final 349-line Slice E guard all passed.

## Phase 4: Documentation/Release (Slice F)

- [x] 4.1 **RED:** Add a failing README/release checklist for commands, schemas/errors, exits, security, determinism, version and checksum. Evidence: added `codegauge/tests/readme_checks.py`; `python3 tests/readme_checks.py` failed against the bootstrap README with the expected missing-contract list.
- [x] 4.2 **GREEN:** Update `codegauge/README.md` with JSON-only CLI, profiles, provenance, exits, limits, no-policy/network, immutable release/checksum. Evidence: README now documents the exact v0.1.0 CLI, contracts, JaCoCo semantics, limits, boundaries, references, future integration, and release checklist; the same check returned `README CHECKS: PASS`.
- [x] 4.3 **REFACTOR/VERIFY:** Reconcile README with RFC/spec/corrected design; `git diff --check`; confirm no changes to `agent-harness/`, `openspec/config.yaml`, or canonical `openspec/specs/`. Evidence: `cargo metadata --locked`, workspace tests, fmt, clippy, bootstrap, README check, and diff whitespace checks passed; Slice F filtered guard measured `changed=181 <= 220`; prohibited paths were untouched.
