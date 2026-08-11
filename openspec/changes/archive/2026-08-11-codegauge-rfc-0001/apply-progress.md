# Apply Progress: CodeGauge RFC-0001

## Delivery boundary

- Slice: **C**, tasks **2.4–2.6** only; cumulative continuation after Slice B.
- Strategy recorded in `tasks.md`: `single-pr`, local checkout, no branches/commits/PRs.
- Forecast: High; `Chained PRs recommended: Yes`; `400-line budget risk: High`.
- No work was added for application, CLI, or conformance.
- The pre-existing partial checkout was retained and audited; no reset or deletion was performed.

## Cumulative task status

- [x] 1.1–1.4 — Slice A workspace foundation, pinned toolchain, six-crate boundary, and inward dependency checks (previously completed).
- [x] 2.1 — Model/core RED tests for DTOs, IDs, profiles, invariants, formula, and monotonicity.
- [x] 2.2 — Policy-free model DTOs, checked-in v1 schemas, and the sole validated CRAP implementation.
- [x] 2.3 — Schema/formula verification, formatting, linting, and boundary-check refactor.
- [x] 2.4–2.6 — JaCoCo provider slice.
- [x] 2.7–2.9 — Application/CLI slice.
- [x] 3.1–3.3 — Conformance slice.
- [x] 4.1–4.3 — Documentation/release slice.

## TDD evidence

1. **RED:** Added `descriptor_identity_is_a_hashable_join_key` to
   `crates/codegauge-model/tests/contracts.rs`. Before the production change,
   `cargo test -p codegauge-model --locked descriptor_identity_is_a_hashable_join_key`
   failed for the expected reason: `SymbolIdentity` did not implement `Eq` or `Hash`,
   producing Rust `E0277`/`E0599` at `HashSet::insert`/`contains`.
2. **GREEN:** Added `Eq, Hash` to `SymbolIdentity`; the focused test passed, followed by the
   complete model suite (`3` contract tests and `1` schema test) and core suite (`2` tests).
3. **REFACTOR/VERIFY:** Strengthened the checked-in schema test to assert both checked-in `$id`
   values before normalizing generated schemas. All focused tests remained green.

## Verification evidence

- `rustc --version` → `rustc 1.97.1 (8bab26f4f 2026-07-14)`.
- `cargo metadata --locked --no-deps --format-version 1` → pass.
- `cargo test -p codegauge-model --locked` → pass: 3 contract, 1 schema, 0 doctests failing.
- `cargo test -p codegauge-core --locked` → pass: 2 CRAP tests, 0 doctests failing.
- `cargo fmt --check` → pass.
- `cargo clippy -p codegauge-model -p codegauge-core --all-targets --locked -- -D warnings` → pass.
- `python3 tests/bootstrap_checks.py` initially exposed a false failure because the Slice A
  checker treated model's legitimate `schemars`/`serde` external dependencies as forbidden.
  The checker now compares only internal workspace dependencies; the rerun returned
  `BOOTSTRAP CHECKS: PASS`.
- Formula audit: `powi` occurs only in `crates/codegauge-core/src/lib.rs`; no provider/I/O or
  policy implementation was added.
- 400-line guard with `target` and the submodule `.git` entry excluded returned
  `added=0 changed=0` for the guard snapshot comparison. The literal unfiltered command reports
  `added=1338 changed=1338` because it compares the filtered snapshot against generated `target`
  content; that output is not a meaningful source-slice delta.

## External scan evidence

- Socket `depscore` failed twice with `No valid session`.
- Semgrep supply-chain eventually returned no findings with `targets: []`; this was not an
  effective scan of the CodeGauge submodule.

## Previous Slice B scope and handoff

- Changed/retained implementation scope: `codegauge-model`, `codegauge-core`, checked-in result/error
  schemas, focused tests, and the model boundary checker.
- No changes to `agent-harness/`, canonical `openspec/specs/`, JaCoCo/application/CLI/conformance,
  branches, commits, or PRs.
- Previous Slice B handoff retained for downstream phases; this apply phase created no verify or QA report.

## Slice C TDD evidence

1. **RED before production code:** Added `crates/codegauge-provider-jacoco/tests/jacoco.rs` and
   the JaCoCo fixtures listed below while the provider remained the Slice A/B placeholder. The
   first `cargo test -p codegauge-provider-jacoco --locked --test jacoco` exited `101` for the
   expected missing `collect`, diagnostic/error, observation, and model contract symbols. After
   correcting one test-helper lifetime, the rerun still failed only for those missing feature
   symbols; no production provider code existed at either RED run.
2. **GREEN:** Implemented the bounded streaming adapter and existing `SymbolResult`-based
   observation boundary. The final focused run passed 5 integration tests with 0 failures.
3. **REFACTOR:** Consolidated the scenario matrix without changing behavior; the final package
   suite, fmt check, clippy, model/core tests, and bootstrap check remained green.

## Slice C files and fixture matrix

| File | Action | Coverage |
|---|---|---|
| `codegauge/Cargo.toml` | Modified | Pinned minimal `quick-xml` workspace dependency |
| `codegauge/Cargo.lock` | Modified | Locked `quick-xml 0.38.3` and `memchr` |
| `codegauge/crates/codegauge-provider-jacoco/Cargo.toml` | Modified | Provider dependency on pinned XML reader |
| `codegauge/crates/codegauge-provider-jacoco/src/lib.rs` | Replaced | UTF-8/BOM streaming parser, limits, descriptor identity, counters, bounded diagnostics/errors |
| `codegauge/crates/codegauge-provider-jacoco/tests/jacoco.rs` | Added | Valid/full/zero/partial, overloads, generated methods, omissions, hostile XML and limits |
| `codegauge/fixtures/jacoco/valid-methods.xml` | Added | Method evidence, aggregates, overloads, constructors, `<clinit>`, synthetic/bridge/lambda, unresolved counters |
| `codegauge/fixtures/jacoco/duplicate-identity.xml` | Added | Duplicate symbol identity |
| `codegauge/fixtures/jacoco/invalid-descriptor.xml` | Added | Invalid JVM method descriptor |
| `codegauge/fixtures/jacoco/malformed.xml` | Added | Mismatched/malformed XML |
| `codegauge/fixtures/jacoco/doctype.xml` | Added | DTD/DOCTYPE rejection |
| `codegauge/fixtures/jacoco/entity.xml` | Added | Entity/general-reference rejection |
| `codegauge/fixtures/jacoco/unsupported-encoding.xml` | Added | Non-UTF-8 declaration rejection |

Generated test inputs cover depth 129, 100,001 classes, 100,001 methods, 17 method counters,
`1000000001` required counts as indeterminate evidence, and 64 MiB + 1 input bytes.

## Slice C verification evidence

- `cargo test -p codegauge-provider-jacoco --locked` → pass: 5 integration tests, 0 failures;
  unit/doc tests have 0 failures.
- `cargo test -p codegauge-model --locked` → pass: 3 contract + 1 schema test.
- `cargo test -p codegauge-core --locked` → pass: 2 formula/invariant tests.
- `cargo fmt --all -- --check` → pass.
- `cargo clippy -p codegauge-provider-jacoco -p codegauge-model -p codegauge-core --all-targets --locked -- -D warnings` → pass.
- `python3 tests/bootstrap_checks.py` → `BOOTSTRAP CHECKS: PASS`.
- Base snapshot was created **before Slice C edits** at
  `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-c-base.WPjC4n`,
  excluding `.git` and `target`; no post-edit copy was used. The filtered real guard command
  (`git diff --no-index --numstat "$BASE_DIR" codegauge/ -- ':!.git' ':!target'`) returned
  `added=398 changed=399`, satisfying `changed <= 400`.

## Slice C scope and handoff

- The provider emits valid canonical `SymbolResult` observations with `metrics.crap = None`;
  Slice D remains responsible for later status/CRAP orchestration.
- No path/range mapping is attempted; descriptor identity is the only join, so future ambiguous
  path candidates cannot be silently selected.
- No Maven/Gradle/test/JaCoCo runner, network, dependency installation, artifact mutation, LLM,
  path inference, source copying, application/CLI/conformance implementation, or QA/verify claim
  was added.
- This apply handoff is technical implementation evidence only; `sdd-verify` and `sdd-qa` own
  their respective downstream reports.

## Slice D delivery boundary and guard

- Slice: **D**, tasks **2.7–2.9** only; conformance, documentation, and release remain untouched.
- Strategy: `single-pr`, local checkout; no branches, commits, PRs, agent-harness changes, canonical
  OpenSpec changes, or QA/verify report were created.
- Forecast retained from `tasks.md`: High; `Chained PRs recommended: Yes`; `400-line budget risk: High`;
  explicit local slice guard remained authoritative.
- Immutable pre-edit BASE snapshot: `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-d-base.n1UoPb`.
  It was created before Slice D edits with `.git` and `target` excluded and was not overwritten.
- Guard command: `git diff --no-index --numstat "$BASE_DIR" codegauge/ -- ':!.git' ':!target' | awk ...`.
  Final filtered result: `added=353 deleted=22 changed=375`; guard requirement `changed <= 400` passed.

## Slice D TDD evidence

1. **RED:** After adding only the application/CLI tests and updating the lockfile for declared test
   dependencies, the focused locked runs failed before application production implementation:
   `cargo test -p codegauge-application --locked --test application` reported unresolved application
   symbols (`Analyzer`, `ArtifactReader`, registry, canonical writers, and related contracts), and
   `cargo test -p codegauge-cli --locked --test cli` reported the missing `exit_code_for_error` API.
2. **GREEN:** Implemented the generic application port/registry/analyzer, exact-byte reader and SHA-256,
   provider contract adaptation, core-only CRAP join, COMPLETE/PARTIAL/INCOMPATIBLE handling, UTC
   timestamp, canonical JSON writer, and Clap CLI wiring. Final focused GREEN runs passed:
   `cargo test -p codegauge-application --locked` (3 integration tests),
   `cargo test -p codegauge-cli --locked` (3 CLI tests), and the relevant provider/model/core suites
   (5/4/2 tests respectively).
3. **REFACTOR:** Shared `ProviderObservations`/diagnostics now live at the application boundary and
   JaCoCo re-exports them; the provider still emits no CRAP. Canonical JSON preserves DTO field order,
   UTF-8/newline termination, finite 12-decimal round-half-even rendering, trailing-zero removal, and
   `-0` normalization. Final `cargo fmt --all -- --check`, focused `cargo clippy ... -- -D warnings`,
   `python3 tests/bootstrap_checks.py`, `git diff --check` (no diagnostics), and the filtered BASE guard passed.

## Slice D implementation files

| File | Action | Coverage |
|---|---|---|
| `codegauge/Cargo.toml`, `codegauge/Cargo.lock` | Modified | pinned `sha2` and `clap` dependencies |
| `codegauge/crates/codegauge-application/Cargo.toml` | Modified | application JSON/hash dependencies |
| `codegauge/crates/codegauge-application/src/lib.rs` | Replaced | reader, provenance, registry/port, orchestration, status, timestamp, canonical JSON/errors |
| `codegauge/crates/codegauge-application/tests/application.rs` | Added | hash/path, core join, ordering/summary, statuses, timestamp, numeric/JSON contracts |
| `codegauge/crates/codegauge-provider-jacoco/src/lib.rs` | Modified | shared application observation/diagnostic contract adapter; descriptor identity retained |
| `codegauge/crates/codegauge-cli/Cargo.toml` | Modified | `codegauge` binary target, pinned Clap, JSON test dependency |
| `codegauge/crates/codegauge-cli/src/main.rs` | Replaced | `analyze`, `profiles`, `version`, stdout/stderr and exit mapping |
| `codegauge/crates/codegauge-cli/tests/cli.rs` | Added | commands, args, JSON streams, complete/partial/incompatible and exits |

## Slice D scope and handoff

- No conformance vectors, README/release checklist, install/runner/network/configuration, or policy
  logic was added.
- High CRAP remains measurement-only; no PASS/FAIL status exists in application or CLI output.
- This remains implementation evidence only; `sdd-verify` owns technical conformance and `sdd-qa`
  owns independent acceptance QA.

## Slice E delivery boundary and guard

- Slice: **E**, tasks **3.1–3.3** only; README/release documentation remains Slice F.
- Strategy: `single-pr`, local checkout; no branches, commits, PRs, `agent-harness` changes,
  canonical `openspec/specs/` changes, or verify/QA reports were created.
- Forecast retained from `tasks.md`: High; `Chained PRs recommended: Yes`; `400-line budget risk: High`.
  The approved local Slice E guard is the stricter user-requested `changed <= 350`.
- Immutable pre-edit BASE snapshot: `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-e-base.4e0K6e`.
  It was created before Slice E edits with `.git` and `target` excluded and was not overwritten.
- Guard command: `BASE_DIR="/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-e-base.4e0K6e"; git diff --no-index --numstat "$BASE_DIR" codegauge -- ':!.git' ':!target' | awk '{a+=$1;d+=$2} END {print "added="a,"deleted="d,"changed="a+d}'`.
- Final guard result: `added=349 deleted=0 changed=349`; `changed <= 350` passed. No split was required.

## Slice E TDD evidence

1. **RED:** Added `crates/codegauge-conformance/tests/conformance.rs` and a placeholder
   `tests/golden/valid-methods.json` before changing production code. After correcting an initial
   test-helper compile typo, `cargo test -p codegauge-conformance --test conformance` ran 8 tests:
   7 passed and the expected golden assertion failed with `left: Object {...}` versus
   `right: Object {}`. The failure demonstrated that the empty conformance/golden suite did not
   yet provide the required expected document.
2. **GREEN:** Replaced the placeholder with the checked-in canonical golden and kept the behavior
   in existing model/core/application/provider code. `cargo test -p codegauge-conformance --locked
   --test conformance` passed all 6 conformance tests.
3. **REFACTOR:** Consolidated helpers and kept the conformance test readable without changing
   behavior; bounded exhaustive property checks cover the required domains, so no additional
   `proptest` dependency was needed. The final focused package run, formatter, clippy, workspace
   suite, bootstrap, and diff checks remained green.

## Slice E implementation files

| File | Action | Coverage |
|---|---|---|
| `codegauge/crates/codegauge-conformance/Cargo.toml` | Modified | Test-only `schemars`/`serde_json` access for DTO schema and contract assertions; no new third-party package was added |
| `codegauge/Cargo.lock` | Modified | Records the conformance crate's test-only workspace dependencies |
| `codegauge/crates/codegauge-conformance/tests/conformance.rs` | Added | Fixture vectors, schema equality/parseability, property invariants, golden comparison, ordering, summary, digest, boundary ownership, no-policy, and repeatability assertions |
| `codegauge/tests/golden/valid-methods.json` | Added | Stable full result document with only `analysis_timestamp` represented by `<masked>` |

## Slice E coverage and verification evidence

- Fixture vectors cover `valid-methods.xml` full/zero/partial instruction coverage, missing and
  invalid required counters, duplicate identity, malformed XML, DOCTYPE/entity/encoding hostile
  inputs, overloads, constructors, `<clinit>`, synthetic/bridge/lambda methods, and depth/class/
  method/counter/input/count limits.
- Schema tests compare both checked-in v1 schemas to `schemars` output from the Rust DTOs and parse
  canonical result/error JSON back into `ResultDocument`/`ErrorDocument`, including parseable-input
  path and SHA-256 error details.
- Golden tests mask only `provenance.analysis_timestamp`, assert all other JSON fields, bytewise
  symbol ID ordering, unrounded max/mean summary values, exact fixture SHA-256, newline/UTF-8 output,
  and canonical 12-decimal/`-0` number rendering. Repeatability compares the complete non-timestamp
  document and digest from two analyses of the same fixture.
- Boundary assertions prove the provider emits `metrics.crap = None`, the application is the only
  join to `codegauge-core::calculate_crap`, and result/error output contains no PASS/FAIL/policy.
  Existing CLI tests already cover exits/stdout/stderr, so no redundant CLI slice was added.
- `cargo test -p codegauge-conformance --locked` → pass: 6 integration tests, 0 unit failures,
  0 doc-test failures.
- `cargo test --workspace --locked` → pass: application 3, CLI 3, conformance 6, core 2, model
  contracts 3 + schema 1, provider 5; all unit/doc tests had 0 failures.
- `cargo fmt --all -- --check` → pass.
- `cargo clippy --workspace --all-targets --locked -- -D warnings` → pass.
- `python3 tests/bootstrap_checks.py` → `BOOTSTRAP CHECKS: PASS`.
- `git diff --check` → pass with no diagnostics.
- No README/release docs, provider/core/application/CLI production code, harness code, references,
  branches, commits, PRs, verify report, or QA report were added in Slice E.

## Slice F delivery boundary and guard

- Slice: **F**, tasks **4.1–4.3** only; no production code or configuration behavior was changed.
- Strategy: `single-pr`, local checkout; no branches, commits, PRs, `agent-harness/` changes,
  `openspec/config.yaml` changes, canonical `openspec/specs/` changes, or verify/QA reports.
- Immutable pre-edit BASE snapshot: `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-f-base.AOFaAr`.
  It was created before Slice F edits with `.git` and `target` excluded and was not overwritten.
- Filtered guard compared that BASE with the post-edit checkout snapshot
  `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-slice-f-final.rOTwvI` (also
  excluding `.git` and `target`) using `git diff --no-index --numstat`; result:
  `added=176 deleted=5 changed=181`.
  The user limit `changed <= 220` passed; the slice was not stopped or widened.

## Slice F TDD/documentation evidence

1. **RED:** Added `codegauge/tests/readme_checks.py` before modifying the README. The first
   `python3 tests/readme_checks.py` run failed with the expected missing command, contract, limit,
   reference, and release-checklist fragments in the bootstrap README.
2. **GREEN:** Updated `codegauge/README.md` with the exact v0.1.0 JSON-only CLI, status/exit mapping,
   schemas, provenance, JaCoCo semantics, determinism, security boundary, references, future harness
   boundary, and release checklist. `python3 tests/readme_checks.py` returned `README CHECKS: PASS`.
3. **REFACTOR/VERIFY:** Made the documentation check whitespace/backtick tolerant so Markdown wrapping
   does not weaken its required-fragment assertions; reran it successfully. `git diff --check` and
   the filtered BASE comparison reported no whitespace diagnostics; the existing workspace evidence
   also remained green under the locked metadata/test/fmt/clippy/bootstrap commands.

## Slice F files

| File | Action | Coverage |
|---|---|---|
| `codegauge/tests/readme_checks.py` | Added | Red/green contract checks for critical usage, contract, boundary, security, and release text |
| `codegauge/README.md` | Replaced | Purpose/boundary, prerequisites, CLI, JSON contracts, semantics, provenance, limits, references, integration, and release checklist |
| `openspec/changes/codegauge-rfc-0001/tasks.md` | Modified | Marked only 4.1–4.3 complete with evidence |

## Slice F verification evidence

- `rustc --version` → `1.97.1`.
- `cargo metadata --locked --no-deps --format-version 1` → pass.
- `cargo test --workspace --locked` → pass: all workspace unit, integration, conformance, and doc tests.
- `cargo fmt --all -- --check` → pass.
- `cargo clippy --workspace --all-targets --locked -- -D warnings` → pass.
- `python3 tests/bootstrap_checks.py` → `BOOTSTRAP CHECKS: PASS`.
- `python3 tests/readme_checks.py` → `README CHECKS: PASS`.
- `git diff --check` → no diagnostics.

## Slice F scope and handoff

- README documentation matches the RFC, normative spec, and corrected design for evidence-only
  measurement, exact CLI/result/error contracts, JaCoCo semantics, limits, deterministic output, and
  immutable release/checksum practice.
- Release checklist deliberately avoids claiming that cross-platform binaries were produced; target
  triples and checksums are evidence to record only for artifacts actually built.
- This is implementation evidence only. `sdd-verify` owns technical conformance and `sdd-qa` owns
  independent capability acceptance; this slice created neither report.

## Critical verification remediation

- Delivery boundary: only the two confirmed CRITICAL findings from `verify-report.md` were addressed
  in `codegauge-provider-jacoco`; no warning-only application change, feature, branch, commit, PR, or
  forbidden-scope change was added. The existing `single-pr` local strategy remains in force.
- [x] Regressions for raw invalid UTF-8 in XML comments and processing instructions now require
  `collect` to return `Err`.
- [x] Regression coverage for unknown JaCoCo wrappers and incompatible parent-child nesting now
  requires `collect` to return `Err`.

### TDD evidence

1. **RED:** Added the two regression tests to
   `crates/codegauge-provider-jacoco/tests/jacoco.rs` before changing provider production code. After
   correcting a test-only byte-slice helper typo (which was a compile error, not RED evidence), the
   focused runs failed for the intended reasons: the UTF-8 test reported `accepted 189 bytes`, and the
   hierarchy test reported `accepted 200 bytes`; both exited `101` because the existing provider
   accepted the hostile inputs.
2. **GREEN:** Added full-input UTF-8 validation after optional BOM removal and a JaCoCo hierarchy
   allowlist checked before attributes or element-specific processing. Both focused regressions passed
   (`1 passed, 0 failed` each).
3. **REFACTOR/VERIFY:** The existing direct `<class>` fixture and aggregate/method counter behavior
   remained green. The provider suite passed all 7 tests, and the full locked workspace suite passed.

### Remediation implementation

- `crates/codegauge-provider-jacoco/src/lib.rs` now calls `std::str::from_utf8` on the complete input
  after stripping BOM and before `quick-xml` parsing, so Comment/PI bytes cannot bypass UTF-8 checks.
- The parser now accepts only the JaCoCo element set and compatible parent-child edges: `report`,
  `group`, `package`, `sourcefile`, `class`, `method`, `counter`, `line`, and `sessioninfo`. It keeps
  direct `<class>` under `<report>` for the existing contract, rejects unknown wrappers and rejects
  known elements below incompatible parents. `counter`, `line`, and `sessioninfo` remain empty-only.
- `codegauge-application` was intentionally not changed. Inputs over 64 MiB are rejected from file
  metadata before `fs::read` and SHA-256, preserving the security limit. SHA-256 is mandatory here
  only for parseable input errors; no oversized bytes are read or hashed, and the warning-only digest
  behavior was not silently relaxed.

### Commands and results

- `cargo test -p codegauge-provider-jacoco --locked --test jacoco raw_invalid_utf8_in_comment_and_processing_instruction_is_rejected` → RED exit `101`, then GREEN `1 passed, 0 failed`.
- `cargo test -p codegauge-provider-jacoco --locked --test jacoco unknown_wrappers_and_incompatible_jacoco_parents_are_rejected` → RED exit `101`, then GREEN `1 passed, 0 failed`.
- `cargo test -p codegauge-provider-jacoco --locked --test jacoco` → PASS, 7 tests.
- `cargo build --workspace --locked` → PASS.
- `cargo test --workspace --locked` → PASS; application 3, CLI 3, conformance 6, core 2, model 4,
  provider 7 integration/contract tests; unit and doc-test groups had no failures.
- `cargo fmt --all -- --check` → PASS.
- `cargo clippy --workspace --all-targets --locked -- -D warnings` → PASS.
- `python3 tests/bootstrap_checks.py` → `BOOTSTRAP CHECKS: PASS`.
- `python3 tests/readme_checks.py` → `README CHECKS: PASS`.
- `git diff --check` → PASS with no diagnostics.
- Verify-report CLI reproducer, using `target/debug/codegauge analyze --profile java-jacoco-v1
  --input PATH --format json` for raw `0xff` in a comment and for
  `<report><unknown><class>...</class></unknown></report>` → `CLI REPRODUCER: PASS`; both exited `5`,
  emitted `code=INVALID_INPUT` JSON, and wrote `invalid JaCoCo input` to stderr.

### Limitations

- No coverage runner or independent JSON Schema instance validator is configured in this checkout.
- Cross-platform release builds and `agent-harness` consumption remain outside this change; acceptance
  QA remains owned by `sdd-qa`. `verify-report.md` was not edited or falsified.
