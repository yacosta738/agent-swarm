# Verification Report: `codegauge-rfc-0001`

## Overall status

**PASS WITH WARNINGS**

This is a fresh verification after the remediation recorded in `apply-progress.md`; the previous
hostile-input verdict is superseded. The locked workspace, all tests, formatting/lint checks,
conformance/golden checks, README checks, and the complete CLI matrix pass. Raw invalid UTF-8 in
comments/PIs and unknown/incompatible JaCoCo nesting now produce `INVALID_INPUT` exit `5`.

One warning remains: an input larger than 64 MiB is rejected from file metadata before reading or
hashing, so its `INVALID_INPUT` error has a path but no SHA-256. Per `spec.md`, the mandatory
path/digest rule applies to **parseable-input** errors; this input is rejected before parsing to
preserve the hard cap. This is recorded as `PASS WITH WARNINGS`/an explicit limitation, not a
CRITICAL finding.

This report is technical conformance only. It does not claim product/operator acceptance; `sdd-qa`
owns acceptance scenarios and `qa-report.md`.

## Verification context

| Item | Evidence |
|---|---|
| Change | `codegauge-rfc-0001` |
| Target | `/Users/acosta/Dev/agent-swarm/codegauge` |
| Persistence mode | OpenSpec filesystem artifacts |
| Verification mode | Standard verification; `openspec/config.yaml` sets strict TDD to false |
| Toolchain | `rustc 1.97.1`, `cargo 1.97.1`, active toolchain from `rust-toolchain.toml` |
| Host | `aarch64-apple-darwin` |
| Fresh apply handoff | `apply-progress.md`, Critical verification remediation section, lines 298–357 |
| Report | `openspec/changes/codegauge-rfc-0001/verify-report.md` |

## Completeness and task reconciliation

All 19 task checklist items are marked complete (`1.1–1.4`, `2.1–2.9`, `3.1–3.3`, `4.1–4.3`).
The remediation added focused tests and provider code only; no warning-only application change was
added.

| Task range | Checklist state | Fresh evidence | Verdict |
|---|---:|---|---|
| `1.1–1.4` workspace/infrastructure | Complete | Metadata, build, tests, bootstrap boundary check | PASS |
| `2.1–2.3` model/core | Complete | Core/model tests, schema equality, clippy, formula inspection | PASS |
| `2.4–2.6` JaCoCo provider | Complete | Provider suite now 7 tests; both remediation regressions and CLI probes pass | PASS |
| `2.7–2.9` application/CLI | Complete | CLI matrix and application tests pass; oversized digest tradeoff documented | PASS WITH WARNINGS |
| `3.1–3.3` conformance/golden | Complete | Six conformance tests and golden comparison pass | PASS |
| `4.1–4.3` README/release documentation | Complete | README check and whitespace check pass | PASS |

## Commands and execution evidence

All requested commands were executed freshly from `/Users/acosta/Dev/agent-swarm/codegauge`:

| Command | Result |
|---|---|
| `cargo metadata --locked` | PASS, exit `0`; exactly six workspace packages: application, CLI, conformance, core, model, provider-jacoco |
| `cargo build --workspace --locked` | PASS, exit `0` |
| `cargo test --workspace --locked` | PASS, exit `0`; 25 integration/contract tests passed, 0 failed; unit/doc-test groups passed |
| `cargo fmt --all -- --check` | PASS, exit `0` |
| `cargo clippy --workspace --all-targets --locked -- -D warnings` | PASS, exit `0` |
| `python3 tests/bootstrap_checks.py` | PASS: `BOOTSTRAP CHECKS: PASS` |
| `python3 tests/readme_checks.py` | PASS: `README CHECKS: PASS` |
| `git diff --check` | PASS, exit `0` |
| `rustc --version && cargo --version && rustup show active-toolchain` | PASS: Rust/Cargo `1.97.1`, pinned `aarch64-apple-darwin` toolchain |

No coverage runner or coverage configuration exists in this checkout. Coverage is **NOT TESTED**,
not inferred from passing tests.

## Fresh CLI matrix

The built `target/debug/codegauge` was exercised as a separate process. Each JSON stdout document
was parsed with Python's standard `json` parser and checked for its expected schema ID and required
top-level keys. The full capture is at the temporary path
`/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/codegauge-verify2-cli-capture.json`.

| Case | Exit | Stdout | Stderr | Verdict |
|---|---:|---|---|---|
| `profiles` | `0` | Exact `java-jacoco-v1\n` | Empty | PASS |
| `version` | `0` | Exact `codegauge 0.1.0\n` | Empty | PASS |
| complete temporary report (`CC=1`, full instruction coverage) | `0` | JSON `codegauge-result/v1`, `COMPLETE`, 1 symbol | Empty | PASS |
| complete temporary report (`CC=7`, full instruction coverage) | `0` | JSON `codegauge-result/v1`, profile `java-jacoco-v1`, CRAP `7`, newline-terminated | Empty | PASS |
| `fixtures/jacoco/valid-methods.xml` partial report | `6` | JSON `codegauge-result/v1`, `PARTIAL`, 10 symbols | Four bounded missing/invalid/zero-denominator diagnostics | PASS |
| missing input | `3` | JSON `codegauge-error/v1`, `INPUT_NOT_FOUND`, no result | `input not found or unreadable` | PASS |
| `fixtures/jacoco/malformed.xml` | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path + lowercase 64-hex digest | `invalid JaCoCo input` | PASS |
| unsupported profile `unknown-provider-v1` | `4` | JSON `codegauge-error/v1`, `UNSUPPORTED_PROFILE` | Empty | PASS |
| `--format text` | `2` | JSON `codegauge-error/v1`, `CLI_ERROR` | `unsupported output format: text` | PASS |
| raw `0xff` in XML comment plus valid method | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path + digest | `invalid JaCoCo input` | PASS |
| raw `0xff` in processing instruction plus valid method | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path + digest | `invalid JaCoCo input` | PASS |
| `<report><unknown><class>...</class></unknown></report>` | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path + digest | `invalid JaCoCo input` | PASS |
| incompatible `<class><package/></class>` parent/child | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path + digest | `invalid JaCoCo input` | PASS |
| 64 MiB + 1 sparse input | `5` | JSON `codegauge-error/v1`, `INVALID_INPUT`, path only | `input exceeds 64 MiB` | PASS WITH WARNINGS |

Two fresh complete CLI runs over identical `CC=7` bytes both exited `0`, had empty stderr, emitted
`codegauge-result/v1`/`COMPLETE`/CRAP `7`, shared the same SHA-256, and matched byte-for-byte after
masking only `provenance.analysis_timestamp`.

Focused remediation tests also passed freshly:

```text
cargo test -p codegauge-provider-jacoco --locked --test jacoco raw_invalid_utf8_in_comment_and_processing_instruction_is_rejected
  1 passed, 0 failed, 6 filtered out
cargo test -p codegauge-provider-jacoco --locked --test jacoco unknown_wrappers_and_incompatible_jacoco_parents_are_rejected
  1 passed, 0 failed, 6 filtered out
```

## Normative requirement and scenario matrix

| Requirement/scenario | Fresh evidence | Verdict |
|---|---|---|
| Six crates, pinned Rust/Cargo, committed lockfile | Metadata, build, bootstrap check, `Cargo.toml`, `rust-toolchain.toml`, `Cargo.lock` | PASS |
| Isolation: core has no provider/I/O; overload IDs distinct | Core manifest/source boundary, bootstrap check, provider/conformance tests, fresh workspace build | PASS |
| Exactly one core-owned `crap-original-v1` formula | `codegauge-core/src/lib.rs:27-40`; production `powi` implementation appears only there; formula tests pass | PASS |
| Model semantics/profile/version axes/descriptor identity | Model contracts and schema tests; IDs preserve class, method, descriptor and overloads | PASS |
| JaCoCo complexity/instruction semantics | Provider/conformance tests and fresh complete `CC=7` CLI result with CRAP `7` | PASS |
| Bounded counters and aggregate ignoring | Fresh workspace/provider suite covers zero/full/partial, invalid counts, zero denominator, 16 counters, count cap, and aggregate counters | PASS |
| Constructors, `<clinit>`, synthetic, bridge, anonymous/lambda methods | `valid-methods.xml` plus provider/conformance assertions | PASS |
| Duplicate IDs, missing identity, invalid descriptors | Duplicate, missing-field, invalid-descriptor fixtures/tests and malformed CLI case | PASS |
| Missing/unresolved is indeterminate, never zero | Partial fixture yields 10 compatible symbols and exit `6`; zero-compatible yields structured `INCOMPATIBLE_MEASUREMENTS` | PASS |
| Deterministic order, paths, summaries, finite numbers, rounding, newline | Application/conformance/golden tests and fresh CLI repeatability; bytewise ID sorting and canonical output observed | PASS |
| Result/error schemas and IDs | Checked-in schemas equal generated Rust DTO schemas; every JSON CLI case parsed and had expected v1 ID/required envelope | PASS WITH WARNINGS |
| Parseable-input path/digest | Fresh malformed and incompatible errors include display path and lowercase exact-byte digest; results include provenance digest | PASS |
| Oversized-input digest tradeoff | Reader rejects metadata length before `fs::read`/SHA-256 at `codegauge-application/src/lib.rs:20`; fresh result is exit `5` with path only. `spec.md:56` scopes MUST digest to parseable-input errors; cap preservation is intentional | PASS WITH WARNINGS |
| Provenance and timestamp | Fresh results include tool/profile/provider/semantics/path/SHA-256/RFC3339 `Z` timestamp; unknown metadata remains absent | PASS |
| CLI required args, JSON-only format, exits `0/2/3/4/5/6/10` | Fresh profiles/version/complete/partial/missing/malformed/unsupported/invalid-format matrix plus CLI tests | PASS |
| No policy or quality verdict | Fresh result/error outputs contain no quality verdict; production grep finds no threshold/policy calculation | PASS |
| Security: UTF-8, hostile XML, limits, no tools/network/mutation/LLM/plugins | Fresh invalid UTF-8 and hierarchy probes now reject; existing DTD/entity/encoding/depth/count/counter/size tests pass; production grep finds no external runner/network path | PASS WITH WARNINGS |
| Future source/range ambiguity | No such provider exists in this MVP; descriptor identity is used and no fallback is present | NOT TESTED |
| Conformance, golden, README/release contract | Six conformance tests, golden comparison, README check, fmt/clippy/bootstrap/diff checks | PASS |
| Independent harness consumption | Standalone public CLI/result/error boundary exists, but no adapter is in this change | NOT TESTED |
| Cross-platform release builds/checksums | Only local `aarch64-apple-darwin` build exists | NOT TESTED |

### RFC Gherkin scenarios

| Scenario | Fresh runtime evidence | Verdict |
|---|---|---|
| Valid JaCoCo report | Complete and partial CLI runs produce result v1 JSON, profile, semantics, symbols and CRAP | PASS |
| Full coverage (`CC=7`) | Fresh complete CLI run returns CRAP `7` | PASS |
| Identical evidence twice | Fresh two-run comparison matches all non-timestamp JSON and digest | PASS |
| Missing artifact | Exit `3`, `INPUT_NOT_FOUND`, no fabricated metric result | PASS |
| Malformed artifact | Exit `5`, `INVALID_INPUT`, no CRAP result | PASS |
| Unsupported profile | Exit `4`, `UNSUPPORTED_PROFILE`, no heuristic/provider attempt | PASS |
| No quality policy | Result reports CRAP only and contains no quality PASS/FAIL verdict | PASS |
| Auditable provenance | Tool version, profile, provider, semantics, path, digest, and `Z` timestamp present | PASS |
| Hostile XML | Invalid UTF-8 comments/PIs and invalid JaCoCo parent/wrapper inputs now exit `5` with `INVALID_INPUT` | PASS |
| Ambiguous future suffix/range mapping | No future mapping capability exists in this change | NOT TESTED |

## Correctness table

| Area | Evidence | Verdict |
|---|---|---|
| Formula/invariants | Core tests and bounded conformance properties pass full/zero/partial, finite, and monotonicity cases | PASS |
| Provider extraction/correlation | Seven provider tests plus six conformance tests pass counters, identities, overloads, generated methods, omission, hostile input, and limits | PASS |
| Application join/status/summary | Three application tests and fresh complete/partial/incompatible CLI runs pass | PASS |
| Canonical JSON | Golden/schema/timestamp/float/order/UTF-8/newline/repeatability checks pass | PASS |
| Error mapping/streams | Requested normal matrix and hostile probes produce one JSON stdout document, expected exits, and stderr diagnostics | PASS WITH WARNINGS |
| Oversized input | Hard cap is enforced; no digest is emitted because bytes are not read/hashed | PASS WITH WARNINGS |

## Design coherence

| Design decision | Fresh evidence | Verdict |
|---|---|---|
| Independent six-crate Rust repository | Six workspace members; no harness dependency or adapter in target | PASS |
| Model/core inward dependency direction | Cargo manifests and bootstrap checks; core depends only on model | PASS |
| Providers collect; core calculates | Provider emits `metrics.crap = None`; application is the only production caller of `calculate_crap` | PASS |
| Artifact-first, no runner/network/policy | CLI consumes supplied XML; production source/dependency grep finds no external command/network/LLM/install path | PASS WITH WARNINGS |
| Versioned CLI/result/error boundary | Profiles/version/JSON IDs/schemas/exit mapping exercised | PASS |
| Bounded UTF-8/hierarchy parser | Full-input UTF-8 validation and explicit JaCoCo hierarchy allowlist are present and runtime regressions pass | PASS |
| References are non-normative | `crap4java`, `crap4go`, `crap4clj` appear only in README/check context; no copied code/dependency/runner found | PASS |
| Harness integration deferred | No `agent-harness`, canonical specs/config, or QA report changes | PASS for scope; NOT TESTED for adapter acceptance |

## Findings

### CRITICAL

None. The two previous CRITICAL findings were remediated and independently re-executed:

- raw invalid UTF-8 in comments/PIs now returns `INVALID_INPUT` exit `5`;
- unknown wrappers and incompatible parent-child elements now return `INVALID_INPUT` exit `5`.

### WARNING

1. **Oversized input has no SHA-256 in its error details.** The reader checks file metadata before
   reading to enforce the 64 MiB cap, so it cannot hash the rejected bytes without violating the
   intended bounded-read tradeoff. `spec.md` requires path/digest for parseable-input errors; this
   file is rejected before parsing. Fresh evidence confirms exit `5`, `INVALID_INPUT`, path-only
   details. Keep this as an explicit contract/security limitation or narrow the specification if the
   project later requires digests for rejected oversized files.
2. **Dependency scans are unavailable/ineffective.** Socket depscore failed with `No valid session`.
   Semgrep supply-chain returned `targets: []`, `scanned: []`, and `total_bytes: 0`; it did not scan
   the `codegauge` submodule.
3. **Release/integration evidence is local only.** No cross-platform release builds/checksums or
   `agent-harness` adapter exist in this change.

### SUGGESTION

1. Add an independent JSON Schema instance validator to CI. Python `jsonschema` is unavailable in
   this environment; current evidence is runtime JSON parsing/ID/required-key checks plus Rust DTO
   schema equality and result/error deserialization tests.
2. Add a dedicated fixture for the intentional oversized-digest tradeoff if the warning is retained
   as part of the public release contract.

## Forbidden scopes and reference checks

- Fresh parent `git diff --name-status -- agent-harness openspec/config.yaml openspec/specs RFC` is
  empty. Parent status still shows the RFC as pre-existing untracked content (`RFC tracked: False`),
  so no HEAD diff exists for it; this verification did not write or modify the RFC.
- `agent-harness/`, `openspec/config.yaml`, `openspec/specs/`, and `qa-report.md` were not changed.
- The target contains no harness adapter, no additional provider, and no release output claim.
- `crap4java`, `crap4go`, and `crap4clj` references are confined to README/readme-check context;
  no copied source, dependency, runner, threshold, policy, or reference execution was found.
- No production file was modified during this verification; only this OpenSpec report was updated.

## Untested and unavailable scope

- Independent JSON Schema instance validator: **NOT TESTED** (`jsonschema=unavailable`).
- Coverage: **NOT TESTED**; no coverage tool/configuration exists.
- Cross-platform release builds/checksums: **NOT TESTED**.
- `agent-harness` adapter/end-to-end consumption: **NOT TESTED**; adapter is out of scope.
- Socket dependency scoring: **NOT TESTED**; tool returned `No valid session`.
- Semgrep supply-chain: **NOT TESTED**; result had `targets: []` and scanned zero bytes.
- Future path/range ambiguity capability: **NOT TESTED**; no such provider exists in the MVP.

## Verdict table

| Finding | Judge A | Judge B | Severity | Status |
|---------|---------|---------|----------|--------|
| Raw invalid UTF-8 comments/PIs now rejected by provider regressions and CLI | ✅ | ✅ | CRITICAL resolved | Resolved |
| Unknown wrappers/incompatible JaCoCo parents now rejected by provider regressions and CLI | ✅ | ✅ | CRITICAL resolved | Resolved |
| 64 MiB + 1 input exits `5` without digest because reader rejects before read/hash | ✅ | ✅ | WARNING | Accepted limitation |
| Independent JSON Schema instance validator unavailable | ✅ | ✅ | SUGGESTION | NOT TESTED |
| Cross-platform releases and harness adapter absent by scope | ✅ | ✅ | WARNING | NOT TESTED |
| Socket/Semgrep dependency scans unavailable or empty-target | ✅ | ✅ | WARNING | NOT TESTED |

## Final verdict and next recommendation

**Technical verdict: PASS WITH WARNINGS.** The remediation is effective and the fresh implementation,
CLI behavior, contract checks, and required test/lint/documentation commands pass. The only remaining
technical warning is the explicitly documented >64 MiB path/digest tradeoff; it is not CRITICAL under
the `spec.md` parseable-input wording. Next recommendation: hand off to **`sdd-qa`** for independent
acceptance QA; do not claim product acceptance from this technical report.
