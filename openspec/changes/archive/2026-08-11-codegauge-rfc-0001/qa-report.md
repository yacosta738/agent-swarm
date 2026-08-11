# Acceptance QA Report: `codegauge-rfc-0001`

## 1. Identity

| Item | Value |
|---|---|
| Change | `codegauge-rfc-0001` |
| Mode | Independent capability-driven acceptance QA |
| Phase | `qa` |
| Date | 2026-08-11 |
| QA target | `/Users/acosta/Dev/agent-swarm/codegauge` |
| QA executor | `sdd-qa` |

This report evaluates observable CLI, artifact, and operator behavior. It is not a second static
verification report and does not claim product acceptance for `agent-harness` or for a future
adapter.

## 2. Source artifacts and technical handoff

Read before execution:

- `openspec/changes/codegauge-rfc-0001/proposal.md`
- `openspec/changes/codegauge-rfc-0001/spec.md` (the only delta spec in this change; no `specs/` subdirectory exists)
- `openspec/changes/codegauge-rfc-0001/design.md`
- `openspec/changes/codegauge-rfc-0001/tasks.md`
- `openspec/changes/codegauge-rfc-0001/verify-report.md`
- `openspec/changes/codegauge-rfc-0001/state.yaml`
- `openspec/config.yaml`
- `tmp/RFC-0001 — CodeGauge_ Deterministic Multi-Language Code Metrics Engine.md`
- target `README.md`, Cargo manifests, repository schemas, fixtures, and executable tests

Technical handoff: `verify-report.md` is **PASS WITH WARNINGS**. It reports the two prior
CRITICAL hostile-input findings fixed and re-executed. Its remaining material warning is that an
input over 64 MiB is rejected from file metadata before reading/hashing, so its `INVALID_INPUT`
error has a path but no SHA-256. QA independently exercised that behavior below.

## 3. Target, environment, permissions, and limitations

| Item | Evidence / limitation |
|---|---|
| Working target | `/Users/acosta/Dev/agent-swarm/codegauge` |
| Host | macOS `aarch64-apple-darwin`, Apple Silicon |
| Toolchain | `rustc 1.97.1`, `cargo 1.97.1`, active `1.97.1-aarch64-apple-darwin` overridden by `rust-toolchain.toml` |
| Execution | Local child-process invocation of `target/debug/codegauge`; no service, credentials, or network target required |
| Permissions | Read access to target artifacts and execute access to the locally built binary; temporary QA fixtures were written only under `/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/codegauge-qa-runtime` |
| Source/config mutation | None by QA. No `agent-harness/`, `openspec/config.yaml`, or canonical `openspec/specs/` path was changed. `verify-report.md` was not edited. |
| Target limitation | The checkout has no application server, browser UI, persistence service, or general product test runner. The product surface is the standalone CLI and its file/JSON contract. |
| Release limitation | Current local target worktree is not an immutable release checkout; implementation files and `Cargo.lock` are present in the worktree but not tracked by the target's current Git commit. Release/commit readiness is therefore not claimed. |

## 4. Capability inventory and decisions

| Capability | Decision | Rationale / evidence |
|---|---|---|
| Rust build and workspace tests | **AVAILABLE / SELECTED** | Local pinned Cargo toolchain executed metadata, build, tests, fmt, clippy, bootstrap, and README checks. |
| Standalone CLI process | **AVAILABLE / SELECTED** | `target/debug/codegauge` was invoked as an independent child process across complete, partial, error, hostile, and limit cases. |
| JaCoCo artifact/file input | **AVAILABLE / SELECTED** | Temporary reports and repository fixtures supplied observable XML inputs and exact-byte hashing. |
| JSON contract parsing | **AVAILABLE / SELECTED** | Python standard-library JSON parsing validated every JSON CLI capture, newline termination, schema ID, and required envelope keys. |
| Determinism/provenance | **AVAILABLE / SELECTED** | Same input was run twice with timestamp masking; digest and non-timestamp JSON matched and timestamps ended in `Z`. |
| Exploratory/security input | **AVAILABLE / SELECTED** | Invalid UTF-8, hierarchy violations, DTD/entity, encoding, duplicate, descriptor, count, depth, counter, and size probes were executed. |
| Persistence/state transition | **AVAILABLE / SELECTED (read-only scope)** | The CLI has no mutable application state; input hash and unchanged input bytes were checked across runs. |
| Browser/UI | **UNAVAILABLE / REJECTED as not applicable** | No browser surface or application server exists in the target. |
| API/network | **UNAVAILABLE / REJECTED as not applicable** | This MVP consumes local artifacts and exposes a process CLI, not an HTTP/API service. |
| Accessibility | **REJECTED as not applicable** | No interactive visual UI; JSON/stdout/stderr are the observable interface. Screen-reader and keyboard UI checks do not apply. |
| Responsive behavior | **REJECTED as not applicable** | No visual layout or device viewport exists. |
| Internationalization/locale | **AVAILABLE but NOT SELECTED** | The contract is machine-oriented and the RFC does not define localized output; no locale matrix was run. |
| Coverage tool | **AVAILABLE but REJECTED for this gate** | `cargo-llvm-cov` and `cargo-tarpaulin` are installed, but they measure Rust test coverage, not acceptance of an externally supplied JaCoCo report; no coverage configuration exists. |
| Independent JSON-Schema instance validator | **UNAVAILABLE / NOT TESTED** | `python3` import of `jsonschema` failed with `ModuleNotFoundError`; stdlib JSON parsing is not equivalent to schema validation. |
| Cross-platform release build/checksum | **UNAVAILABLE / NOT TESTED** | Only local macOS arm64 execution was available; no release artifacts for the RFC target platforms were supplied. |
| Dependency scan | **NOT TESTED** | No authenticated/usable dependency-scan capability was part of this local QA run. |
| `agent-harness` adapter | **REJECTED as explicitly out of scope** | The RFC/proposal defer the adapter. CLI/result/error contracts were consumed directly instead. |
| Manual operator acceptance | **AVAILABLE / SELECTED** | QA reviewed captured process exit, stdout, stderr, parsed JSON, and observable provenance rather than relying only on source inspection. |

## 5. Scenario matrix

Every scenario below has one of the required QA outcomes. `PASS` is based on fresh command or
child-process evidence; source inspection alone was not used as acceptance evidence.

### 5.1 Build/test capability

| ID | Scenario | Result | Fresh evidence |
|---|---|---|---|
| B-01 | Locked workspace metadata resolves the independent workspace | **PASS** | `cargo metadata --locked` exit `0`; parsed output reports six workspace members: `codegauge-model`, `codegauge-core`, `codegauge-application`, `codegauge-provider-jacoco`, `codegauge-cli`, `codegauge-conformance`. Cargo emitted only its format-version advisory on stderr. |
| B-02 | Workspace builds with locked dependencies | **PASS** | `cargo build --workspace --locked` exit `0`; `Finished dev profile`. |
| B-03 | Workspace tests execute | **PASS** | `cargo test --workspace --locked` exit `0`; 25 integration/contract tests passed, 0 failed, with unit and doc-test groups also successful. |
| B-04 | Formatting and lint gates execute cleanly | **PASS** | `cargo fmt --all -- --check` exit `0`; `cargo clippy --workspace --all-targets --locked -- -D warnings` exit `0`. |
| B-05 | Bootstrap and README contract checks execute | **PASS** | `python3 tests/bootstrap_checks.py` exit `0`, `BOOTSTRAP CHECKS: PASS`; `python3 tests/readme_checks.py` exit `0`, `README CHECKS: PASS`. |

### 5.2 Standalone CLI capability

The full raw child-process capture was written during QA at
`/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/codegauge-qa-runtime/cli-captures.json`.
The table records the parsed observable contract and exact stderr diagnostics where present.
All 24 JSON cases parsed as one newline-terminated document with the expected schema ID and
required top-level keys. All captured stdout/stderr streams were free of uppercase quality
`PASS`/`FAIL` tokens.

| ID | Invocation / input | Result | Evidence: exit, stdout, stderr |
|---|---|---|---|
| C-01 | `profiles` | **PASS** | Exit `0`; stdout exactly `java-jacoco-v1\n`; stderr empty. |
| C-02 | `version` | **PASS** | Exit `0`; stdout exactly `codegauge 0.1.0\n`; stderr empty. |
| C-03 | Temporary complete JaCoCo report, `CC=7`, full instruction coverage | **PASS** | Exit `0`; stdout schema `codegauge-result/v1`, status `COMPLETE`, 1 symbol, complexity `7`, instruction ratio `1`, CRAP `7`, summary `max=7 mean=7`; stderr empty. Exact input SHA-256: `c7e8a0eea258e3f33b9c3bf88ac56804f094aedd1b26db789b2ef2fb5cb47864`. |
| C-04 | `fixtures/jacoco/valid-methods.xml` partial fixture | **PASS** | Exit `6`; stdout schema `codegauge-result/v1`, status `PARTIAL`, 10 symbols, summary `max=12 mean=3.5323`; stderr contains four bounded diagnostics for missing/invalid/zero-denominator evidence. |
| C-05 | Missing input path | **PASS** | Exit `3`; stdout schema `codegauge-error/v1`, code `INPUT_NOT_FOUND`, path-only details; stderr exactly `input not found or unreadable`. |
| C-06 | `fixtures/jacoco/malformed.xml` | **PASS** | Exit `5`; stdout schema `codegauge-error/v1`, code `INVALID_INPUT`, path plus lowercase 64-hex digest; stderr exactly `invalid JaCoCo input`. |
| C-07 | Unsupported profile `unknown-provider-v1` | **PASS** | Exit `4`; stdout schema `codegauge-error/v1`, code `UNSUPPORTED_PROFILE`, empty details; stderr empty. |
| C-08 | `--format text` | **PASS** | Exit `2`; stdout schema `codegauge-error/v1`, code `CLI_ERROR`; stderr exactly `unsupported output format: text`. |
| C-09 | Invalid arguments: missing `--format` and unknown option | **PASS** | Both invocations exit `2`, emit `codegauge-error/v1` / `CLI_ERROR` JSON on stdout, and write only short argument diagnostics to stderr. |
| C-10 | Zero compatible symbols: required instruction evidence absent | **PASS** | Exit `6`; stdout schema `codegauge-error/v1`, code `INCOMPATIBLE_MEASUREMENTS`, path plus digest; stderr exactly `no compatible measurements`. |
| C-11 | JSON/error envelope and stream discipline | **PASS** | A fresh assertion over 26 captures found 24 JSON cases, all expected schema IDs/required keys, all stdout newline-terminated, and no quality `PASS`/`FAIL` token in either stream. |

### 5.3 Security and untrusted-input capability

| ID | Scenario | Result | Fresh evidence |
|---|---|---|---|
| S-01 | Raw invalid UTF-8 in XML comment | **PASS** | Direct CLI probe exits `5`; `codegauge-error/v1` / `INVALID_INPUT`; path and exact-byte digest present; stderr `invalid JaCoCo input`. |
| S-02 | Raw invalid UTF-8 in processing instruction | **PASS** | Direct CLI probe exits `5`; `codegauge-error/v1` / `INVALID_INPUT`; path and digest present; stderr `invalid JaCoCo input`. |
| S-03 | Unknown wrapper and incompatible parent/child hierarchy | **PASS** | Both direct CLI probes exit `5` with `INVALID_INPUT`, path, digest, and `invalid JaCoCo input`; focused provider regressions also pass (`1 passed` each). |
| S-04 | DTD/DOCTYPE and entity input | **PASS** | `doctype.xml` and `entity.xml` each exit `5` with `INVALID_INPUT`, path, lowercase digest, and `invalid JaCoCo input`; no external resolution occurred. |
| S-05 | Unsupported XML encoding declaration | **PASS** | `unsupported-encoding.xml` exits `5` with `INVALID_INPUT`, path/digest, and `invalid JaCoCo input`. |
| S-06 | Depth, class-count, method-count, and counter-count limits | **PASS** | Direct probes for depth `129`, `100001` classes, `100001` methods, and `17` counters each exit `5` with `INVALID_INPUT` and parseable path/digest details. Focused `malformed_hostile_encoding_bom_and_limits_follow_the_boundary` passed (`1 passed`). |
| S-07 | Required counter value over `1,000,000,000` | **PASS** | Direct probe returns structured `INCOMPATIBLE_MEASUREMENTS` exit `6` because the invalid required evidence is omitted and no compatible symbol remains; the provider test explicitly asserts `InvalidRequiredCounter` and empty symbols. This is the documented indeterminate-evidence path, not an unbounded parse. |
| S-08 | Input larger than 64 MiB | **PASS WITH WARNING** | Sparse `64 MiB + 1` input exits `5`, `INVALID_INPUT`, with `input exceeds 64 MiB`; details contain path but no digest because rejection occurs before read/hash. See finding F-03. |
| S-09 | Duplicate identity and invalid JVM descriptor | **PASS** | Each fixture exits `5` with `INVALID_INPUT`, path/digest, and no metric result. |

### 5.4 Contract and measurement capability

| ID | Scenario | Result | Fresh evidence |
|---|---|---|---|
| M-01 | Complexity total and instruction ratio | **PASS** | Parsed `valid-methods.xml`: `full` has complexity `0+7=7`, ratio `10/(0+10)=1`; `zero` has `0+3=3`, ratio `0/(4+0)=0`; `partial` has `2+5=7`, ratio `7/(3+7)=0.7`. |
| M-02 | CRAP full, zero, and partial edges | **PASS** | Parsed output reports CRAP `7`, `12`, and `8.323` for those three methods respectively; custom `CC=7` full-coverage run reports exactly `7`. |
| M-03 | Missing evidence is not fabricated as zero | **PASS** | Partial output contains 10 compatible symbols and four stderr diagnostics; `missing`, `invalid`, `zero-denominator`, and `optional-only` are omitted. The zero-compatible probe returns a structured error rather than a zero score. |
| M-04 | Overload descriptor IDs remain distinct | **PASS** | Runtime IDs include `java:com/acme/Order#overload()V` and `java:com/acme/Order#overload(I)V`; parsed IDs are bytewise sorted. |
| M-05 | Constructors, `<clinit>`, synthetic, bridge, lambda/generated methods | **PASS** | Runtime partial result includes `<init>`, `<clinit>`, `synthetic`, `bridge`, and `lambda$run$0` symbols. |
| M-06 | Provider emits observations without calculating CRAP | **PASS** | Focused runtime test `valid_full_zero_partial_and_generated_methods_are_observed_without_crap` passed (`1 passed`, 6 filtered); its assertion verifies provider observations have no CRAP metric. |
| M-07 | Application preserves provenance, digest, order, and summary | **PASS** | Fresh assertion confirmed SHA-256 equals the exact input bytes, IDs are bytewise sorted, summary is `{"crap":{"max":12,"mean":3.5323}}`, and output contains provider semantics/path/timestamp provenance. Input hash before/after analysis was unchanged. |
| M-08 | Repeatability excluding timestamp | **PASS** | Two separate complete runs exited `0`; non-timestamp JSON matched exactly, digest matched exact bytes, timestamps were `2026-08-11T11:37:45Z` and `2026-08-11T11:37:46Z`, and both ended in `Z`. |
| M-09 | Schema, golden, formula, and conformance suites | **PASS** | `cargo test -p codegauge-model --locked --test schemas`: 1 passed; `cargo test -p codegauge-model --locked --test contracts`: 3 passed; `cargo test -p codegauge-core --locked --test crap`: 2 passed; `cargo test -p codegauge-conformance --locked --test conformance`: 6 passed. |

### 5.5 Standalone boundary capability

| ID | Scenario | Result | Fresh evidence |
|---|---|---|---|
| A-01 | Invoke without `agent-harness` present | **PASS** | Absolute binary and absolute input were executed from the temporary QA directory with a minimal `PATH`-only environment; exit `0`, result schema `codegauge-result/v1`, status `COMPLETE`, stderr empty, and no harness directory present. |
| A-02 | Preserve the intended no-harness/no-canonical-change boundary | **PASS** | Parent Git path check showed no changes under `agent-harness`, `openspec/config.yaml`, or `openspec/specs`; no QA command wrote those paths. This source-control check supplements the runtime standalone result; it is not used as the sole product acceptance evidence. |
| A-03 | Future adapter acceptance | **NOT TESTED** | No adapter exists by explicit proposal/RFC scope. The released CLI/result/error contract is demonstrably consumable by the direct process probes, but end-to-end harness translation and policy/FSM behavior require a future adapter target. |

## 6. Untested scope, reason, and rerun prerequisite

| Scope | Result | Reason | Rerun prerequisite |
|---|---|---|---|
| Rust coverage measurement | **NOT TESTED** | Coverage tools are installed, but no project coverage configuration exists and Rust test coverage does not validate the artifact-first JaCoCo acceptance behavior. | Decide whether Rust line/branch coverage is a release gate; provide the selected tool/config and run it from the target checkout. |
| Independent JSON Schema instance validation | **NOT TESTED** | Python `jsonschema` is unavailable (`ModuleNotFoundError`); stdlib parsing and Rust DTO/schema equality are not independent instance validation. | Provide/install an approved schema validator, then validate every captured result/error document against both repository schemas. |
| Cross-platform release builds/checksums | **NOT TESTED** | Only local macOS arm64 was available; no immutable release artifacts were supplied. | Provide pinned release revision and CI/build capability for macOS x86_64, Linux x86_64, Linux arm64, and any claimed Windows target, with checksums. |
| Dependency scans | **NOT TESTED** | No authenticated/usable dependency scanning capability was available in this QA run. | Provide an approved dependency/SCA scanner session and scan the target workspace and lockfile. |
| `agent-harness` adapter / end-to-end consumption | **NOT TESTED** | Explicitly deferred by RFC/proposal; no adapter target exists. | Implement or provide a future adapter target that invokes only the released CLI and consumes `codegauge-result/v1` / `codegauge-error/v1`. |
| Locale matrix | **NOT TESTED** | No localized contract is specified for this JSON-only MVP. | Define supported locales and expected machine/diagnostic output before adding locale acceptance. |
| Browser, responsive, accessibility UI | **NOT TESTED / NOT APPLICABLE** | No browser or visual UI target exists. | Supply a browser/UI target if the product surface expands beyond the CLI. |

## 7. Findings

| ID | Severity | Status | Finding / evidence |
|---|---|---|---|
| F-01 | **CRITICAL** | **RESOLVED** | Raw invalid UTF-8 in comments/PIs is rejected as `INVALID_INPUT` exit `5`; both direct probes and the focused provider regression passed. |
| F-02 | **CRITICAL** | **RESOLVED** | Unknown wrappers and incompatible JaCoCo parent/child elements are rejected as `INVALID_INPUT` exit `5`; both direct probes and the focused provider regression passed. |
| F-03 | **P2** | **OPEN — accepted limitation** | The hard 64 MiB pre-read check emits a path-only `INVALID_INPUT` error without SHA-256. This is observable and documented, but consumers requiring a digest for every rejected artifact need a clarified contract or bounded hashing design. |
| F-04 | **P2** | **OPEN — validation gap** | Independent JSON Schema instance validation was not run because `jsonschema` is unavailable. Runtime JSON parsing, schema IDs/keys, Rust schema equality, and conformance tests passed, but those are not a substitute for an independent validator. |
| F-05 | **P2** | **OPEN — release gap** | Cross-platform release artifacts/checksums and immutable revision validation were not tested. The current target worktree is not a release-ready Git revision; no release claim is made. |
| F-06 | **P3** | **OPEN — scope/tooling gap** | Coverage-tool and dependency-scan evidence is absent. This does not invalidate the selected CLI acceptance scenarios, but it limits broader release assurance. |
| F-07 | **P3** | **OUT OF SCOPE** | Harness adapter and end-to-end policy/FSM consumption are deferred intentionally. The CLI/result/error contract is consumable independently; adapter acceptance remains future work. |

No unresolved `CRITICAL`, `P0`, or `P1` finding was observed in the selected runtime acceptance scope.

## 8. Final verdict

**QA verdict: PASS WITH WARNINGS**

The selected acceptance capabilities pass fresh runtime checks: the standalone CLI builds and runs
without the harness, emits the documented exits and versioned JSON contracts, calculates the
JaCoCo-derived values at method granularity, preserves overload identity/provenance/digest/order,
keeps missing evidence indeterminate, rejects hostile and bounded inputs, and produces no quality
`PASS`/`FAIL` result. The warnings are visible and concern the documented oversized-input digest
tradeoff plus untested validator, release, scan, coverage, locale, and future-adapter scope.

This verdict is **not** an archive-readiness approval. The archive phase must apply the configured
policy to acceptance-relevant `NOT TESTED` scope and the open warnings; no code, config, or
verification report was changed by QA.

## 9. Implementation handoff

- Preserve the public boundary exercised here: `codegauge analyze --profile java-jacoco-v1 --input PATH --format json`, `profiles`, and `version`.
- A future harness adapter should invoke the released executable and consume only the v1 result/error contracts; it should not import CodeGauge crates, duplicate CRAP, or own JaCoCo semantics.
- Carry F-03 explicitly into any consumer contract: oversized pre-parse rejection currently has no digest.
- Before a release claim, run the independent schema validator, approved dependency scans, selected coverage gate, and cross-platform immutable-release/checksum checks.
- No source/config fixes were made during QA; findings are handed off for policy/release decision rather than silently downgraded.
