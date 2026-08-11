## Exploration: codegauge-rfc-0001

### Current State

#### Repos and baseline

- `tmp/RFC-0001 — CodeGauge_ Deterministic Multi-Language Code Metrics Engine.md` is a 1,773-line
  proposed RFC targeting `0.1.0`, implemented in Rust, distributed as a standalone CLI with
  machine-readable contracts (`:1-11`).
- `codegauge` is an independent Git checkout on `main`, clean at `d53bdb7` (`chore: bootstrap
  repository`). It currently contains only `README.md` and `.gitignore`; there is no
  `Cargo.toml`, `Cargo.lock`, `rust-toolchain.toml`, source tree, test suite, fixture set, or
  schema. This phase did not modify that repository.
- The parent workspace lists `agent-harness` and `codegauge` as sibling submodules in
  `.gitmodules:1-6`. This is not a CodeGauge submodule inside `agent-harness`; it is compatible
  with the RFC's independent-repository boundary (`RFC:231-267`).
- The inspected environment has stable Rust `1.97.1` on `aarch64-apple-darwin`, but the target
  repository has not pinned a toolchain yet.

#### Actual MVP `0.1.0` scope

The RFC's MVP is one complete vertical slice, not a genuinely multi-language implementation yet:

- **Workspace and crates:** one Cargo workspace with `codegauge-core`, `codegauge-model`,
  `codegauge-application`, `codegauge-cli`, `codegauge-provider-jacoco`, and
  `codegauge-conformance` (`RFC:270-315`, `:1381-1422`). The crates should remain thin; this
  structure is mandatory, but it does not justify speculative providers or plugin infrastructure.
- **Metric path:** JaCoCo XML is the canonical structured input; the Java/JaCoCo provider extracts
  method-level cyclomatic complexity and instruction coverage, maps compatible observations into
  the canonical model, and delegates CRAP calculation to core (`RFC:664-716`). CodeGauge does not
  generate JaCoCo reports or run Maven, Gradle, tests, or JaCoCo.
- **Contracts:** profile `java-jacoco-v1`, formula `crap-original-v1`, versioned result
  `codegauge-result/v1`, versioned error `codegauge-error/v1`, JSON output, provenance, SHA-256
  artifact hashing, schemas, golden fixtures, and CLI integration tests (`RFC:501-555`,
  `:1058-1073`, `:1381-1422`).
- **CLI:** public commands `codegauge analyze`, `codegauge profiles`, and `codegauge version`;
  `analyze` must accept at least `--profile`, `--input`, and `--format json` (`RFC:720-771`).
  CLI arguments are sufficient for `0.1.0`; a project configuration file is deferred
  (`RFC:773-800`).
- **Explicit exclusions:** Kotlin/Kover, Go, Python, JavaScript/TypeScript, Rust source analysis,
  DRY, mutation testing, thresholds, policy, automatic language detection, dependency/tool
  installation, test execution, dynamic plugins, LLMs, and workflow/FSM behavior
  (`RFC:1424-1441`).

#### Mandatory architectural invariants and decisions

- CodeGauge measures evidence; consumers own quality policy and workflow transitions. A high CRAP
  value is reported, never converted into CodeGauge `PASS`/`FAIL` (`RFC:31-33`, `:804-840`).
- `codegauge-core` owns the one production implementation of
  `CRAP = CC² × (1 - coverage)³ + CC`; it has no filesystem, XML/JSON parsing, provider, build,
  network, CLI, policy, or language-specific dependency (`RFC:363-410`, `:641-660`).
- Core inputs must satisfy `CC >= 1`, `0 <= coverage <= 1`, and finite numeric values. Derived
  metrics may join only compatible measurements at the same symbol granularity; project-wide
  coverage must never be silently combined with method complexity (`RFC:627-637`).
- Providers own artifact parsing, symbol identity, raw extraction, correlation, semantic labels,
  provenance, and rejection of malformed/incompatible data. Providers do not calculate CRAP or
  apply thresholds (`RFC:558-593`).
- Symbols require a stable canonical identity. For Java/JaCoCo, method name alone is forbidden
  when a stronger identity exists; the JVM class name plus method name plus descriptor is the
  minimum identity candidate (`RFC:464-499`).
- Profiles, result schemas, and tool version are independent version axes. A provider semantic
  change requires a new profile version; a schema-breaking change requires a new result schema
  version (`RFC:1349-1377`).
- Provider extensibility is compile-time Rust crates in `0.x`; a dynamic Rust ABI is explicitly
  deferred (`RFC:1227-1254`). No CodeGauge runtime dependency installation, network resolution,
  LLM access, source mutation, or artifact mutation is allowed (`RFC:1009-1034`).
- Rust types are authoritative for public contracts; schemas are versioned and should be generated
  from those types where practical (`RFC:1058-1073`).

#### Acceptance and observable contract

The acceptance bar is the RFC's 18-item list (`RFC:1512-1533`): independent Cargo build; core
provider isolation; exactly one CRAP implementation; valid JaCoCo analysis; method/symbol-level
correlation; CRAP for every compatible symbol; profile declaration; result-schema validation;
input SHA-256; no build/test/report generation or dependency installation; no LLM or policy;
deterministic structured errors; identical metric values for identical inputs; golden fixtures;
standalone CLI execution; and consumption by the harness using only the public CLI/result contract.
The Gherkin scenarios additionally require full-coverage identity (`CC=7` yields CRAP `7`), stable
repeatability, missing-input `INPUT_NOT_FOUND`, malformed-input `INVALID_INPUT`, unsupported-profile
rejection, no quality verdict, and auditable provenance (`RFC:1537-1627`).

#### JaCoCo facts that constrain the design

The official JaCoCo DTD (`https://www.jacoco.org/jacoco/trunk/coverage/report.dtd`) defines:

- `report` with `sessioninfo*`, groups/packages, and aggregate counters; package names use VM
  notation; classes have a fully qualified VM name and optional `sourcefilename`;
- methods with required `name` and JVM descriptor `desc`, plus optional first-source `line`;
- counters whose types include `INSTRUCTION`, `BRANCH`, `LINE`, `COMPLEXITY`, `METHOD`, and `CLASS`,
  each with required non-negative-looking `missed`/`covered` text attributes.

JaCoCo's coverage-counter documentation states that instruction coverage is bytecode-instruction
coverage, complexity is calculated per non-abstract method using `v(G) = B - D + 1`, exception
handling is not a branch, and constructors/static initializers (including generated ones) count as
methods. Therefore the profile must explicitly define that MVP CRAP uses the method's total
`COMPLEXITY` counter (`missed + covered`) and method `INSTRUCTION` ratio, not method coverage,
line coverage, branch coverage, or an aggregate class/project counter. That exact interpretation
is implied by the RFC but is not yet specified tightly enough for implementation.

#### Relevant `agent-harness` contract

No CodeGauge-specific adapter or reference exists in `agent-harness`. The relevant existing
boundary is generic and external-process oriented:

- `agent-harness/README.md:152-170` documents the opt-in `quality-runner/v1`, configured commands,
  explicit argv by default, shell opt-in, timeouts, environment allowlists, artifact limits,
  redaction, and visible unavailable/fallback states. It explicitly says no OpenCode plugin hook
  is required for the first runner implementation.
- `agent-harness/scripts/sdd-quality-runner.mjs:14-26` executes declared capabilities and emits a
  versioned runner envelope; `scripts/sdd-runner-lib/config.mjs:37-163` validates project-root
  paths, argv/shell, timeouts, exit codes, parsers, and artifacts; `scripts/sdd-runner-lib/result.mjs`
  records execution, parser status, artifact hashes, and bounded evidence.
- The current `openspec/quality-runner.schema.json:62-81` and
  `scripts/sdd-runner-lib/metrics.mjs:5-10,39-60` formalize `metrics/v1` only for `tests`, `lint`,
  and `coverage`. A legacy `normalizeCrap` helper remains at `metrics.mjs:284`, and smoke scripts
  exercise it, but codegraph found no runtime callers. The current main specs explicitly state
  that CRAP adapters are out of scope (`openspec/specs/capability-adapters/spec.md:1-5` and
  `openspec/specs/capability-policy/spec.md:45-61`).

This means the current phase does **not** require a harness change. A stable standalone CLI/result
contract is the correct integration boundary for the MVP. A later harness adapter may invoke
`codegauge analyze` and translate `codegauge-result/v1` into a consumer evidence/policy contract,
but that should be a separate change: it must not import CodeGauge crates, duplicate the formula,
or silently reinterpret a metric profile. The existing legacy JavaScript CRAP helper is a boundary
risk to resolve in that later change, not a reason to couple the MVP repositories now.

#### OpenSpec context and policy

The root `openspec/config.yaml` is scoped to the harness, not CodeGauge: it describes an OpenCode
configuration stack, Markdown/JSON/Node/Bash conventions, no repository test runner, and
`strict TDD: false` (`openspec/config.yaml:3-10`). Its reusable rules (RFC 2119 scenarios,
rollback plans, task grouping, verification/QA reporting, and the 400-line review guard) remain
valid, but its context and testing claims are not valid Rust-project context.

Do not rewrite that global context during exploration or silently turn the harness configuration
into a Rust configuration. In proposal/design, choose an explicit scoped strategy: document the
CodeGauge target context alongside the change, or introduce an accepted target-local OpenSpec
context without changing unrelated harness policy. The target context must mention pinned stable
Rust/Cargo, workspace crates, `cargo test`/fixture/golden/CLI verification, and rustfmt/clippy
policy if those checks are adopted. The global `quality-runner.json` remains disabled with no
capabilities and `quality-toolchain.lock` contains only Node; neither should be changed merely to
bootstrap CodeGauge. Existing harness main specs and the active unrelated change
`openspec/changes/deterministic-quality-runners-fsm/` must remain untouched.

`state.yaml` is orchestrator-owned. At the original exploration point this phase did not create it;
the change directory now also contains downstream `proposal.md`, `spec.md`, `design.md`, and
`tasks.md` artifacts with `state.yaml` reporting `next: apply`. None of those artifacts were
modified for this reference-implementation update.

#### Decisions that are still underspecified

The following must be closed in proposal/spec/design before implementation:

1. **JaCoCo method semantics:** require `COMPLEXITY` and `INSTRUCTION` counters; define whether
   missing `BRANCH`, `LINE`, or `METHOD` counters are acceptable; define inclusion of constructors,
   static initializers, synthetic/bridge/anonymous/lambda methods; reject or report duplicates.
2. **Symbol identity and paths:** use `(class VM name, method name, JVM descriptor)` for identity and
   a deterministic `symbol_id`; decide how optional `line` and `sourcefilename` are represented,
   because JaCoCo cannot derive the RFC example's `src/main/java` prefix from XML alone. Never
   fabricate repository-relative paths.
3. **Coverage normalization:** use instruction counts `covered/(covered+missed)`; reject negative,
   non-integer, overflow, `covered > total`, and zero-denominator counters; specify whether
   incompatible symbols are omitted, make the analysis partial, or fail the artifact.
4. **Profiles and provider versions:** define the supported JaCoCo XML/DTD versions for
   `java-jacoco-v1`, and distinguish profile semantics from an unknown provider release. Do not
   claim a provider version that the input artifact does not contain.
5. **Provenance and hashing:** hash the exact input bytes before parsing; normalize only displayed
   paths; define optional repository/config provenance and absent-value behavior. The RFC calls
   analysis timestamp required (`RFC:914-965`) but also requires deterministic results; decide
   whether timestamps are excluded from canonical/golden output or treated as non-metric metadata.
6. **Deterministic serialization:** sort symbols and tie-breaks explicitly; define path separators,
   duplicate handling, summary behavior for zero symbols, float precision/rounding, JSON formatting,
   and whether byte-identical JSON or only identical metric values is promised.
7. **Errors and exit codes:** finalize the suggested mapping (`0`, `2`, `3`, `4`, `5`, `6`, `10` at
   `RFC:968-1005`), decide whether `--format json` emits structured errors on stdout for non-zero
   exits, and keep diagnostics on stderr without contaminating machine output. Define `PARTIAL`
   versus `INVALID_INPUT`/`INCOMPATIBLE_MEASUREMENTS`.
8. **Parsing/security limits:** choose bounded XML parsing, maximum input/output bytes, XML depth,
   symbol/counter counts, integer bounds, entity/DTD behavior, encoding policy, path/symlink policy,
   and guaranteed absence of external entity resolution/network access.
9. **Rust stable/toolchain:** pin an exact stable toolchain in `rust-toolchain.toml`, commit
   `Cargo.lock`, select XML/CLI/serialization dependencies compatible with that pin, and define the
   minimum supported Rust version versus the release toolchain. The current environment only proves
   local stable `1.97.1`; it does not establish CI or cross-platform compatibility.

### Reference Implementations

The following implementations were inspected as concrete reference points, not as dependencies or
acceptance authorities for CodeGauge:

- [crap4java README](https://github.com/unclebob/crap4java/blob/main/README.md)
- [`crap4java/src/crap4java/CrapScore.java`](https://github.com/unclebob/crap4java/blob/main/src/crap4java/CrapScore.java),
  [`JacocoCoverageParser.java`](https://github.com/unclebob/crap4java/blob/main/src/crap4java/JacocoCoverageParser.java),
  [`CrapAnalyzer.java`](https://github.com/unclebob/crap4java/blob/main/src/crap4java/CrapAnalyzer.java),
  and [`test/crap4java/CrapScoreTest.java`](https://github.com/unclebob/crap4java/blob/main/test/crap4java/CrapScoreTest.java)
- [crap4go README](https://github.com/unclebob/crap4go/blob/master/README.md),
  [`crap4go/internal/crap/crap.go`](https://github.com/unclebob/crap4go/blob/master/internal/crap/crap.go),
  [`crap4go/internal/coverage/coverage.go`](https://github.com/unclebob/crap4go/blob/master/internal/coverage/coverage.go),
  and [`crap4go/internal/complexity/complexity.go`](https://github.com/unclebob/crap4go/blob/master/internal/complexity/complexity.go)
- [crap4clj README](https://github.com/unclebob/crap4clj/blob/master/README.md),
  [`crap4clj/src/crap4clj/crap.cljc`](https://github.com/unclebob/crap4clj/blob/master/src/crap4clj/crap.cljc),
  [`crap4clj/src/crap4clj/coverage.cljc`](https://github.com/unclebob/crap4clj/blob/master/src/crap4clj/coverage.cljc),
  and [`crap4clj/src/crap4clj/core.cljc`](https://github.com/unclebob/crap4clj/blob/master/src/crap4clj/core.cljc)

#### Patterns CodeGauge can adopt

- **Formula and edge cases:** `crap4java/src/crap4java/CrapScore.java` and
  `crap4java/test/crap4java/CrapScoreTest.java` implement the same formula and test full coverage,
  zero coverage, partial coverage, and unknown coverage. `crap4go/internal/crap/crap.go` and
  `crap4clj/src/crap4clj/crap.cljc` also return an indeterminate result when coverage is absent
  (`nil`) instead of treating missing evidence as `0%`. CodeGauge should preserve that distinction
  in the canonical model: no coverage evidence means no derived CRAP score, not an uncovered symbol.
- **Stable report ordering:** `crap4go/internal/crap/crap.go` copies the input and uses
  `sort.SliceStable`, placing numeric CRAP scores first, unknown scores last, and retaining a name
  tie-break for two unknown entries. CodeGauge should make this stronger and explicit by sorting
  canonical symbols with a deterministic primary metric order and a final `symbol_id` tie-break;
  it must not depend on hash-map iteration order.
- **Correlation diagnostics:** `crap4clj/src/crap4clj/coverage.cljc` exposes
  `lcov-diagnostics`, including normalized source candidates, the number of coverage-file entries,
  and closest suffix matches. `crap4clj/src/crap4clj/core.cljc` reports unresolved namespace
  fallback mappings to stderr as `N/A`, not as zero coverage. CodeGauge can adopt structured
  diagnostics for unmatched, ambiguous, or incomplete provider observations, while keeping them
  separate from metric values and machine-readable result status.
- **Path suffix matching and range mapping as future-provider patterns:**
  `crap4go/internal/coverage/coverage.go` maps coverage by source-file plus line range and falls
  back to segment-aware path suffix matching. `crap4clj/src/crap4clj/coverage.cljc` has analogous
  path candidates, suffix matching, line-range aggregation, and namespace/file fallbacks. These are
  useful future-provider techniques when an ecosystem exposes only source paths and ranges; they
  should be versioned provider semantics, not generic CodeGauge correlation heuristics.
- **Bounded semantics at the provider boundary:** The Go and Clojure implementations separate
  coverage parsing, complexity extraction, and CRAP calculation into distinct packages/namespaces.
  That separation supports the RFC's provider-collects/core-calculates ownership, provided CodeGauge
  keeps provider-specific parsing out of `codegauge-core`.

#### Patterns explicitly rejected for RFC-0001

- **Running build/test/coverage tools:** the three READMEs and their core entry points are
  orchestration tools: `crap4java/README.md` runs Maven/JaCoCo, `crap4go/README.md` runs
  `go test ./... -coverprofile=...`, and `crap4clj/README.md`/`crap4clj/src/crap4clj/core.cljc`
  runs Cloverage through `clj` or Babashka. CodeGauge `0.1.0` must consume an existing JaCoCo XML
  artifact and never execute Maven, Go, Clojure, test runners, or JaCoCo itself.
- **Deleting artifacts or installing dependencies:** the README workflows explicitly delete stale
  coverage output; the Clojure `delete-coverage-dir` and `run-coverage` functions implement the
  same lifecycle. CodeGauge must not delete, regenerate, mutate, download, or install anything
  during analysis. Missing input is a structured unavailable/input error, not a trigger to repair
  the project.
- **Source parsing for the JaCoCo vertical slice:** `crap4java/src/crap4java/CrapAnalyzer.java`
  parses Java source to discover methods and complexity, then correlates coverage by source class,
  name, and line. That is necessary for its own source-driven workflow, but RFC-0001's JaCoCo XML
  already supplies method counters and a JVM descriptor. CodeGauge should not add a Java source
  parser or recompute complexity for this provider; it should use the artifact's
  `COMPLEXITY`/`INSTRUCTION` counters directly.
- **Name-plus-line fallback when a JVM descriptor exists:** `JacocoCoverageParser.java` keys data
  as `class#method:line`, and `CrapAnalyzer.java` first matches that key and then selects the
  nearest same-name method line. This can conflate overloaded methods and can be nondeterministic
  when equally near candidates are present. CodeGauge must use class VM name + method name + JVM
  descriptor as the JaCoCo identity; line/path suffix or range matching is only a future-provider
  fallback when no stronger identity exists, and must never silently override an ambiguity.
- **Thresholds and policy inside CodeGauge:** `crap4java/README.md` defines exit code `2` for a
  CRAP threshold above `8.0`; the Go and Clojure READMEs publish risk bands and workflow guidance.
  CodeGauge must report scores and analysis/error status only. Thresholds, risk labels, pass/fail
  policy, and workflow transitions remain consumer responsibilities under RFC-0001.

#### Implications for future provider contracts and tests

- A provider contract must declare the measurement semantics and units: complexity source and
  definition, coverage source and denominator, percentage-versus-ratio representation, symbol
  granularity, join identity, and the exact behavior for missing coverage. `nil`/`N/A` must remain
  distinguishable from a valid zero-coverage measurement.
- Provider contracts should expose correlation diagnostics and an explicit ambiguity outcome. An
  unmatched symbol, multiple suffix matches, missing range, or invalid counter must be
  `UNAVAILABLE`, `PARTIAL`, `INCOMPATIBLE_MEASUREMENTS`, or another documented non-pass state—not a
  guessed score. Diagnostics should be deterministic and machine-readable, with human detail on
  stderr only where the CLI contract permits it.
- Path suffix matching, source-range aggregation, and name fallback belong in provider-specific,
  versioned semantics. They require fixtures for exact matches, normalized separators, suffix
  collisions, overlapping ranges, no ranges, and ambiguous candidates. They must not become a
  generic cross-language heuristic in core.
- The JaCoCo conformance suite should include full/zero/partial instruction coverage, absent input,
  malformed XML, missing/invalid counters, overloaded methods with distinct descriptors, duplicate
  identities, constructors/static initializers, generated/synthetic methods, and missing optional
  source metadata. Golden output should verify stable `symbol_id` ordering and `N/A`/indeterminate
  behavior without relying on source-parser reconstruction.
- Core tests should cover the formula boundaries and finite-input invariants independently of all
  providers. CLI tests should assert that the analyzer never invokes an external runner, never
  deletes or installs artifacts/dependencies, preserves structured diagnostics, and keeps thresholds
  outside the result contract.

### Affected Areas

- `tmp/RFC-0001 — CodeGauge_ Deterministic Multi-Language Code Metrics Engine.md` — authoritative
  MVP scope, invariants, acceptance criteria, and architectural decisions to translate into the
  proposal/spec/design.
- `codegauge/README.md` and `codegauge/.gitignore` — only current target files; future implementation
  will add the Cargo workspace, pinned toolchain, crates, schemas, fixtures, and tests without
  changing the sibling `agent-harness` repository.
- `codegauge/Cargo.toml`, `codegauge/rust-toolchain.toml`, `codegauge/Cargo.lock` — required future
  reproducibility/workspace boundary; currently absent.
- `codegauge/crates/` and `codegauge/schemas/` — future core/model/application/CLI/provider/
  conformance and result/error schema surfaces mandated by the RFC.
- `codegauge/fixtures/jacoco/` and `codegauge/tests/golden/` — future semantic, malformed-input,
  overload, generated-code, and deterministic-serialization evidence.
- `agent-harness/scripts/sdd-quality-runner.mjs` and `agent-harness/scripts/sdd-runner-lib/{config,result,metrics}.mjs`
  — existing generic external command/evidence boundary; inspected for compatibility, not changed
  in this independent MVP.
- `openspec/quality-runner.schema.json`, `openspec/quality-runner.json`, and
  `openspec/quality-toolchain.lock` — existing harness runner contracts/configuration; preserve
  current disabled state and do not register a CRAP adapter in this change.
- `openspec/config.yaml` and `openspec/specs/{capability-adapters,capability-policy}/spec.md` —
  global policy/context and explicit existing exclusion of CRAP adapters; proposal/design must
  scope Rust context without rewriting harness policy.
- `openspec/changes/codegauge-rfc-0001/exploration.md` — this phase artifact; this request modifies
  only this file and leaves the existing downstream artifacts/state untouched.

### Approaches

1. **Independent six-crate CodeGauge vertical slice with CLI/result boundary** — implement the
   mandated Rust workspace and Java/JaCoCo provider in `codegauge`, with `codegauge-result/v1` and
   `codegauge-error/v1` as the only consumer-facing integration surface; leave the harness unchanged.
   - Pros: obeys the RFC's repository and dependency boundaries; keeps the formula/provenance
     authority in CodeGauge; standalone tests and releases are possible; avoids changing current
     harness specs and global policy.
   - Cons: requires resolving several JaCoCo/float/error semantics before coding; immediate harness
     policy integration is deferred; six crates plus schemas can exceed a comfortable single review.
   - Effort: High

2. **Implement CodeGauge and a harness adapter in one coordinated change** — extend the existing
   runner/metrics contract to invoke CodeGauge and normalize CRAP evidence immediately.
   - Pros: demonstrates end-to-end consumer integration sooner; can reuse existing execution,
     artifact-hash, redaction, and status plumbing.
   - Cons: expands scope across independent repositories; current `metrics/v1` and main harness
     specs explicitly exclude CRAP; risks a second CRAP implementation in JavaScript, coupling
     release cadence, and changing global OpenSpec policy; contradicts the RFC's clean boundary.
   - Effort: High

3. **Single-crate Rust prototype, split into the RFC workspace later** — use one crate to settle
   JaCoCo parsing and formula behavior before introducing the six-crate layout.
   - Pros: fastest way to experiment with XML semantics, symbol identity, and golden fixtures;
     smaller initial dependency graph.
   - Cons: not compliant with the `0.1.0` workspace structure; encourages CLI/provider/core
     coupling; migration can change public types and schema semantics after tests are written.
   - Effort: Medium initially, High migration risk

### Recommendation

Proceed with Approach 1. Treat `codegauge` as the product repository and `agent-harness` as a future
consumer, not as a shared implementation surface. Keep the six crates minimal and enforce the
dependency direction: model/core types at the center, provider parsing at the edge, application
orchestration above them, and CLI serialization/invocation at the boundary. Prefer a bounded,
non-networking JaCoCo parser and explicit canonical sorting/serialization, but make the exact crate
and numeric policies a design decision rather than an assumption.

The proposal should explicitly state that no harness code is needed for this MVP, while the CLI
contract must be stable enough for a later adapter: one machine-readable result/error document,
profile and schema identity, raw artifact SHA-256, semantic labels, deterministic symbol ordering,
documented exit codes, and no quality verdict. It should also split work into reviewable slices
(workspace/contracts/core; JaCoCo semantics; CLI/provenance/fixtures) because the likely change is
larger than the default 400-line review budget. A separate future change can decide how to retire or
adapt the current unused/legacy harness `normalizeCrap` helper.

### Risks

- **Semantic false precision:** JaCoCo's XML contains bytecode-level counters, optional source
  metadata, generated methods, and aggregate counters. A name-only or path-based join can silently
  assign a score to the wrong overload or claim source provenance that the artifact does not provide.
- **Coverage/complexity mismatch:** using line, branch, method, class, or aggregate counters instead
  of method instruction coverage plus total method complexity would produce a numerically plausible
  but semantically invalid CRAP result.
- **Incomplete-symbol policy:** the RFC names `PARTIAL` and compatible symbols but does not define
  whether missing counters are skipped, surfaced, or fatal; different choices change symbol counts
  and acceptance behavior.
- **Determinism contradiction:** required analysis timestamps, floating-point serialization, map
  ordering, and input path formatting can make repeated JSON bytes differ even when metric values
  match. This must be resolved before golden tests.
- **Input trust and resource exhaustion:** XML entities/DTDs, deep nesting, huge counters, duplicate
  classes, and oversized reports require explicit parser limits and rejection behavior; no external
  DTD or network lookup may occur.
- **Public error API drift:** suggested exit codes and `PARTIAL`/`INVALID_INPUT` semantics are not
  finalized. Once `0.1.0` ships, consumers will depend on them.
- **Rust reproducibility:** the target has no pinned toolchain or lockfile yet; parser, CLI, serde,
  and hashing dependencies must be pinned and compatible with stable Rust and the planned release
  targets.
- **Harness contract mismatch:** the existing generic runner can execute an external CLI, but its
  formal metric registry excludes CRAP and its legacy helper duplicates the formula. Integrating it
  prematurely could violate the “one formula/consumer owns policy” boundary.
- **OpenSpec scope drift:** the global config describes the harness and an unrelated active change
  already exists. Rewriting global context or state, or editing existing capability specs, would
  create unrelated blast radius and confuse change resolution.
- **Review size:** the mandated workspace, schemas, fixtures, parser, CLI, and tests likely exceed
  400 changed lines; task planning must forecast and slice delivery before apply.

### Ready for Proposal

Yes. The orchestrator should tell the user that exploration found a clean independent target and a
usable future CLI boundary, so no `agent-harness` production change is required for the MVP phase.
Before implementation, the proposal/spec/design must lock JaCoCo counter semantics, overload and
generated-symbol policy, path/identity rules, coverage normalization, timestamp/deterministic JSON
policy, structured errors/exit codes, parser limits/security, Rust toolchain/MSRV, scoped OpenSpec
context, and review slices. No implementation or QA claim is made by this exploration.
