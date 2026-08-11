# CodeGauge 0.1.0 Specification

### Requirement: Boundary

Workspace MUST contain six crates: `codegauge-core`, `codegauge-model`,
`codegauge-application`, `codegauge-cli`, `codegauge-provider-jacoco`, and `codegauge-conformance`.
Rust/Cargo MUST pin `1.97.1` with committed `Cargo.lock`. Core MUST NOT use providers/I/O and MUST
contain exactly one `crap-original-v1`: `CC² × (1-coverage)³ + CC`,
rejecting non-finite values, `CC < 1`, or coverage outside `[0,1]`. Providers MUST delegate;
threshold/PASS/FAIL MUST NOT be calculated. Observations MUST preserve profile, symbol
`{id,language,kind,class_vm,name,descriptor}`, and semantics. `java-jacoco-v1` fixes method kind,
`id=java:<class_vm>#<name><descriptor>`, JaCoCo cyclomatic/instruction semantics, and
`crap-original-v1`; joins MUST use identical ids/granularity, never aggregates. Versions are independent;
semantic changes require a new profile.

#### Scenario: Isolation
- GIVEN a pinned checkout
- WHEN built
- THEN no harness/network and distinct overload IDs

### Requirement: JaCoCo

Input MUST be a JaCoCo `<report>` with method `COMPLEXITY` and `INSTRUCTION`: complexity=`missed+covered`;
coverage=`covered/(missed+covered)`. Required counts MUST be bounded nonnegative base-10 integers,
denominator>0, and core-compatible. Missing/invalid/unresolved required coverage MUST omit the
method and remain indeterminate, never zero. A valid remainder is `PARTIAL`; zero compatible symbols
is `INCOMPATIBLE_MEASUREMENTS`; diagnostics MAY identify unresolved symbols. `BRANCH`, `LINE`,
`METHOD`, `CLASS`, and aggregates are ignored. Reported methods, including constructors, `<clinit>`,
synthetic, bridge, anonymous, and lambda methods, MUST be included; descriptors preserve overloads.
Duplicate ids, missing class/name/descriptor, or invalid descriptors MUST reject the artifact.
`sourcefilename`/`line` MAY remain; repository paths MUST NOT be fabricated.

#### Scenario: Missing
- GIVEN unresolved coverage beside a compatible method
- WHEN analyzed
- THEN status is `PARTIAL` and no zero is fabricated

### Requirement: Determinism

Symbols MUST sort bytewise by `symbol.id`; human worst-first ordering from reference tools MUST NOT become
policy/result contract. Paths MUST use `/` without `realpath` or inferred prefixes. Summary MUST use
unrounded compatible scores. Numbers MUST be finite binary64, serialized round-half-even to 12
decimals, stripping trailing zeroes and `-0`. JSON MUST be fixed-order UTF-8/newline-terminated.
Future source/range providers MAY use normalized path-suffix/range mapping only under a versioned
profile; ambiguous matches MUST reject or report indeterminate, never silently join nearest/name-only.

#### Scenario: Ambiguity
- GIVEN ambiguous suffix/range candidates
- WHEN matched
- THEN reject/indeterminate, never nearest/name-only

### Requirement: Contracts

Results MUST validate `codegauge-result/v1` with tool/profile/status (`COMPLETE`/`PARTIAL`), symbols,
summary, provenance, path, and digest. Errors MUST validate `codegauge-error/v1` with tool/code/message;
parseable-input errors MUST also include path/digest. SHA-256 MUST hash exact bytes before parsing as
lowercase 64-hex; unknown metadata MUST be absent. Results MUST include UTC RFC3339
`analysis_timestamp` ending `Z`; golden comparisons exclude it. `COMPLETE` means all methods
compatible; `PARTIAL` means compatible plus omissions; zero compatible means
`INCOMPATIBLE_MEASUREMENTS`. Malformed/structurally invalid XML, duplicates, or limits are
`INVALID_INPUT`; missing/unreadable input is `INPUT_NOT_FOUND`; unknown profile is
`UNSUPPORTED_PROFILE`. Exits: `0` complete, `2` CLI, `3` missing, `4` unsupported, `5` invalid,
`6` partial/incompatible, `10` internal. Each nonzero outcome MUST emit one error JSON on stdout
except `PARTIAL`, which emits one result JSON with exit `6`; diagnostics MAY use stderr only.
`analyze` MUST require `--profile java-jacoco-v1 --input PATH --format json`; it MUST NOT auto-detect,
read project config, or generate reports. `profiles` MUST print `java-jacoco-v1`; `version` MUST print
`codegauge 0.1.0`; invalid arguments/formats use CLI_ERROR.

#### Scenario: CLI
- GIVEN any input state
- WHEN `analyze` runs
- THEN matching JSON and exit MUST be emitted

### Requirement: Security

Input MUST be read-only UTF-8 XML (optional BOM), capped at 64 MiB, depth 128, 100,000 classes,
100,000 methods, 16 counters/method, and 1,000,000,000 per required count. DTD/DOCTYPE, entities,
external resolution, unsupported encodings, network, commands, installation, mutation, LLM, and
plugins MUST be absent. Fixtures MUST cover valid, zero/full/partial,
overload, generated, missing/unresolved, duplicate, zero-compatible, malformed, and hostile cases;
unit/property/provider/schema/golden/CLI/conformance tests MUST verify formula, identity,
ordering, hashes, floats/timestamps, contracts, exits, stdout, and stderr. MVP MUST NOT add other
providers, DRY/mutation, thresholds/policy, build/test/report execution, downloads,
auto-detection, plugins, LLMs, or source analysis. Later harness integration MUST use only released
CLI/v1 JSON: never copied source/deps, runners, thresholds, crate imports, or duplicate CRAP.

#### Scenario: Hostile
- GIVEN hostile XML or a reference
- WHEN handled
- THEN `INVALID_INPUT`; references remain inspiration only

> **References (non-normative):** JaCoCo DTD <https://www.jacoco.org/jacoco/trunk/coverage/report.dtd>; Java <https://github.com/unclebob/crap4java>; Go <https://github.com/unclebob/crap4go>; Clojure <https://github.com/unclebob/crap4clj>.
