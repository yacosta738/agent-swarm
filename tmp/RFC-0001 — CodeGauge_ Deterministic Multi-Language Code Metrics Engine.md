# RFC-0001 — CodeGauge: Deterministic Multi-Language Code Metrics Engine

**Status:** Proposed  
**Date:** 2026-08-10  
**Authors:** Project Maintainers  
**Target release:** `0.1.0`  
**Repository:** Independent Git repository  
**Implementation language:** Rust  
**Distribution model:** Standalone CLI + machine-readable contracts  
**Primary initial metric:** CRAP  
**Initial provider:** Java / JaCoCo  

---

## 1. Summary

CodeGauge is a standalone, deterministic, multi-language code metrics engine.

Its initial purpose is to provide a reliable implementation of the CRAP metric across programming languages without requiring each consuming harness, CI pipeline, IDE integration, or AI agent to understand how cyclomatic complexity and test coverage are obtained for each ecosystem.

CodeGauge separates:

- metric definitions;
- provider-specific data extraction;
- metric calculation;
- normalization;
- provenance;
- serialization;
- policy.

The central architectural principle is:

> **CodeGauge measures code. It does not decide whether the measured code is acceptable.**

For CRAP specifically:

```text
Language/provider-specific collector
            │
            │ extracts
            ▼
Cyclomatic Complexity + Coverage
            │
            ▼
      Canonical Model
            │
            ▼
       CRAP Engine
            │
            ▼
    Structured Evidence
```

The CRAP formula is implemented exactly once in the CodeGauge core.

Language- and ecosystem-specific collectors are responsible only for translating their source artifacts into the canonical metric model.

Consumers such as an agent harness remain responsible for policy evaluation and workflow transitions.

---

# 2. Motivation

Different programming ecosystems expose code-quality information differently.

Examples include:

- JVM projects using JaCoCo;
- Kotlin projects using Kover or JaCoCo-compatible reports;
- Go using native coverage and source/AST analysis;
- Python using coverage reports plus complexity analyzers;
- JavaScript/TypeScript using Istanbul-compatible coverage plus source analysis;
- Rust using ecosystem-specific coverage and complexity providers.

The CRAP metric itself is simple:

```text
CRAP = CC² × (1 - coverage)³ + CC
```

where:

```text
CC       = cyclomatic complexity
coverage = coverage ratio in the range [0, 1]
```

The difficult problem is not evaluating this formula.

The difficult problem is obtaining trustworthy and semantically compatible values for:

```text
CC(symbol)
coverage(symbol)
```

for the same symbol in different technology stacks.

This problem should not be reimplemented independently by every agent harness.

---

# 3. Problem Statement

A technology-agnostic harness may know that it requires a capability named:

```text
crap
```

but it should not need logic such as:

```text
if Java:
    run JaCoCo
    parse method complexity
    parse instruction coverage

if Python:
    run coverage.py
    run complexity tool
    correlate functions

if Go:
    parse coverprofile
    analyze Go AST

if TypeScript:
    parse Istanbul
    analyze TypeScript AST
```

That would leak technology-specific metric implementation into the orchestration layer.

Instead, the desired relationship is:

```text
Harness
   │
   │ requests CRAP evidence
   ▼
CodeGauge
   │
   │ returns canonical evidence
   ▼
Harness Policy
```

The harness should not need to know how the evidence was calculated beyond its declared provenance and semantics.

---

# 4. Goals

CodeGauge MUST:

1. Provide deterministic code metric calculations.
2. Implement the CRAP formula once.
3. Support multiple programming languages and metric providers.
4. Preserve the semantics of every measurement.
5. Produce structured, machine-readable output.
6. Provide enough provenance for results to be audited.
7. Be usable independently from any AI agent or harness.
8. Be consumable from CI, scripts, IDE integrations and other tooling.
9. Support reproducible execution.
10. Allow additional metrics to be introduced without redesigning the architecture.
11. Avoid coupling the core domain to any specific language.
12. Avoid coupling the project to OpenCode or any specific agent runtime.

---

# 5. Non-Goals

CodeGauge `0.x` is NOT intended to:

- act as an AI agent;
- contain LLM functionality;
- decide whether a CRAP score is acceptable;
- enforce project quality policies;
- implement workflow transitions;
- replace test runners;
- execute an entire CI pipeline;
- install JaCoCo, Kover, pytest, Vitest or other ecosystem tools;
- automatically download dependencies during analysis;
- run mutation testing;
- replace coverage generators;
- provide automatic refactoring;
- interpret whether duplicated code should be refactored;
- estimate metrics using heuristics or an LLM;
- become a generic build system.

CodeGauge measures and normalizes evidence.

---

# 6. Fundamental Responsibility Boundaries

The system follows these ownership rules:

```text
Provider Collector owns:
"How do I obtain CC and coverage for this symbol?"

CodeGauge Core owns:
"What CRAP value corresponds to these inputs?"

CodeGauge Model owns:
"What does a canonical measurement look like?"

CodeGauge CLI owns:
"How does an external process invoke CodeGauge?"

Consumer Policy owns:
"Is this measurement acceptable?"

Consumer FSM owns:
"What happens next?"

LLM owns:
"How should the code be changed?"
```

A critical invariant is:

> **An LLM is never part of the metric calculation chain.**

---

# 7. Repository Strategy

CodeGauge SHALL live in an independent Git repository.

It SHALL NOT be:

- embedded in the agent harness repository;
- maintained as copied source inside the harness;
- implemented as a Git submodule of the harness;
- dependent on harness-specific contracts.

Conceptually:

```text
repositories/
│
├── agent-harness/
│   └── CodeGauge adapter
│
└── codegauge/
    └── standalone metrics engine
```

The relationship is dependency-based:

```text
agent-harness
     │
     │ invokes released executable
     ▼
 codegauge
     │
     │ JSON result
     ▼
metrics/evidence adapter
```

The repository SHOULD be designed so that it can be published independently even if its first consumer remains private.

---

# 8. Monorepo Strategy

CodeGauge SHALL use a Rust Cargo workspace.

Individual languages SHALL NOT initially be split across independent Git repositories.

The workspace provides:

- one source repository;
- one `Cargo.lock`;
- one release process;
- common schemas;
- common domain types;
- common conformance tests;
- coordinated versioning;
- isolated provider crates.

Initial structure:

```text
codegauge/
├── Cargo.toml
├── Cargo.lock
├── rust-toolchain.toml
├── README.md
├── LICENSE
│
├── crates/
│   ├── codegauge-core/
│   ├── codegauge-model/
│   ├── codegauge-application/
│   ├── codegauge-cli/
│   │
│   ├── codegauge-provider-jacoco/
│   └── codegauge-conformance/
│
├── schemas/
│   ├── codegauge-result-v1.schema.json
│   └── codegauge-error-v1.schema.json
│
├── fixtures/
│   └── jacoco/
│
└── tests/
    └── golden/
```

Future providers may add:

```text
codegauge-provider-kover
codegauge-provider-go
codegauge-provider-python
codegauge-provider-istanbul
codegauge-provider-rust
```

---

# 9. Architectural Model

CodeGauge follows a ports-and-adapters style architecture.

```text
                    ┌──────────────────┐
                    │       CLI        │
                    └────────┬─────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │    Application      │
                  └─────────┬───────────┘
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
      Provider Registry              Metric Engine
             │                             │
             ▼                             ▼
      Provider Adapter              codegauge-core
             │
             ▼
     Structured Artifact
```

The dependency direction MUST point toward the domain/core.

Provider implementations SHALL depend on canonical CodeGauge contracts.

The core SHALL NOT depend on providers.

---

# 10. Core Domain

`codegauge-core` SHALL remain deliberately small.

Its responsibilities include:

- metric formulas;
- input validation;
- value objects;
- deterministic calculation.

It SHALL NOT contain:

- filesystem access;
- XML parsing;
- JSON parsing;
- build-system knowledge;
- source-language knowledge;
- CLI code;
- policy;
- network access.

Conceptually:

```rust
pub struct CrapInput {
    pub cyclomatic_complexity: f64,
    pub coverage: f64,
}

pub struct CrapScore(f64);

pub fn calculate_crap(input: CrapInput) -> Result<CrapScore, MetricError> {
    // deterministic formula
}
```

Required invariants:

```text
CC >= 1

0 <= coverage <= 1

all numeric values MUST be finite
```

---

# 11. Canonical Measurement Model

Metric values MUST NOT lose their semantics.

The following representation is insufficient:

```json
{
  "complexity": 7,
  "coverage": 0.83
}
```

CodeGauge MUST preserve where those numbers came from and what they mean.

Conceptually:

```json
{
  "complexity": {
    "value": 7,
    "metric": "cyclomatic",
    "semantics": "jacoco-cyclomatic",
    "provider": "jacoco"
  },

  "coverage": {
    "ratio": 0.83,
    "covered": 83,
    "missed": 17,
    "semantics": "instruction",
    "provider": "jacoco"
  }
}
```

This distinction is essential because different ecosystems may use:

```text
instruction coverage
statement coverage
line coverage
form coverage
branch coverage
```

The CRAP formula is common.

Its input semantics may not be.

---

# 12. Symbol Model

CRAP SHALL be calculated at symbol granularity.

A symbol may represent:

```text
method
function
closure
callable
other executable unit
```

Each symbol SHALL have a stable canonical representation containing enough information to correlate measurements.

Example:

```json
{
  "language": "java",
  "kind": "method",
  "path": "src/main/java/com/acme/OrderService.java",
  "namespace": "com.acme",
  "container": "OrderService",
  "name": "calculateTotal",
  "signature": "(Lcom/acme/Order;)D",
  "start_line": 42
}
```

A canonical `symbol_id` MAY be derived from these attributes.

Provider implementations MUST NOT correlate complexity and coverage using only display names when stronger identities are available.

---

# 13. Metric Profiles

CodeGauge SHALL introduce the concept of a **metric profile**.

A profile defines the exact semantics under which a metric was calculated.

Example:

```yaml
id: java-jacoco-v1

language: java

complexity:
  provider: jacoco
  metric: cyclomatic
  semantics: jacoco-cyclomatic

coverage:
  provider: jacoco
  metric: instruction
  semantics: jacoco-instruction

formula:
  metric: crap
  version: original-v1
```

Therefore:

```text
CRAP = 7.2
```

is considered incomplete evidence.

The complete interpretation is:

```text
CRAP = 7.2
profile = java-jacoco-v1
```

Future profiles may include:

```text
kotlin-kover-v1
jvm-jacoco-v1
go-native-v1
python-coverage-v1
typescript-istanbul-v1
```

Profiles SHALL be versioned whenever their semantics change.

---

# 14. Provider Contract

Providers translate ecosystem-specific artifacts into canonical measurements.

Conceptually:

```rust
pub trait MetricProvider {
    fn id(&self) -> ProviderId;

    fn supported_profiles(&self) -> &[ProfileId];

    fn collect(
        &self,
        request: CollectionRequest,
    ) -> Result<Vec<SymbolObservation>, ProviderError>;
}
```

A provider is responsible for:

- parsing its supported artifact;
- identifying symbols;
- extracting raw measurements;
- correlating measurements to symbols;
- declaring measurement semantics;
- reporting provenance;
- rejecting malformed or incompatible data.

A provider is NOT responsible for:

- applying CRAP thresholds;
- declaring code quality acceptable;
- modifying source code;
- invoking an LLM;
- performing workflow transitions.

---

# 15. Derived Metrics

CodeGauge SHALL distinguish source measurements from derived metrics.

For CRAP:

```text
ComplexityMeasurement ─┐
                       ├──► CRAP Engine
CoverageMeasurement ───┘
```

Conceptually:

```yaml
metric: crap

kind: derived

requires:
  - cyclomatic-complexity
  - coverage

join:
  granularity: symbol

formula:
  id: crap-original-v1
```

The metric engine SHALL refuse to silently combine incompatible measurements.

For example:

```text
Project-wide coverage
+
Method-level complexity
```

MUST NOT automatically become a method-level CRAP score.

---

# 16. CRAP Formula

CodeGauge `crap-original-v1` SHALL implement:

```text
CRAP = CC² × (1 - coverage)³ + CC
```

Inputs:

```text
CC       ∈ [1, ∞)
coverage ∈ [0, 1]
```

The implementation SHALL exist only once inside `codegauge-core`.

Provider crates MUST NOT independently implement the CRAP formula.

Golden tests SHALL protect the formula from accidental semantic changes.

---

# 17. Initial Java / JaCoCo Provider

Release `0.1.0` SHALL provide the first complete vertical slice using Java and JaCoCo.

The provider SHALL consume a structured JaCoCo report.

Conceptual flow:

```text
Java project
    │
    │ external build/test process
    ▼
JaCoCo XML
    │
    ▼
codegauge-provider-jacoco
    │
    ├── symbol
    ├── cyclomatic complexity
    └── instruction coverage
            │
            ▼
      canonical model
            │
            ▼
         CRAP core
```

CodeGauge SHALL NOT be responsible for generating the JaCoCo report.

The consuming system is responsible for ensuring the report exists.

---

# 18. Artifact-First Design

CodeGauge SHALL prefer structured artifacts over human-readable process output.

Preferred order:

```text
1. Structured API / protocol
2. JSON
3. XML
4. Standard machine format
5. Stable documented text format
6. Human stdout parsing
```

Human stdout parsing SHOULD NOT be implemented when a structured alternative exists.

For Java/JaCoCo, the XML report SHALL be the canonical source.

---

# 19. CLI

The public interface SHALL initially be a standalone CLI named:

```bash
codegauge
```

Minimum commands:

```bash
codegauge analyze
codegauge profiles
codegauge version
```

Example:

```bash
codegauge analyze \
  --profile java-jacoco-v1 \
  --input build/reports/jacoco/test/jacocoTestReport.xml \
  --format json
```

Possible human-oriented output:

```text
CodeGauge

Profile: java-jacoco-v1
Symbols analyzed: 183

Highest CRAP:

12.84  OrderService.calculateTotal
10.21  PaymentService.authorize
 8.47  PricingEngine.resolve
```

Machine-oriented execution:

```bash
codegauge analyze \
  --profile java-jacoco-v1 \
  --input report.xml \
  --format json
```

SHALL produce the canonical JSON contract.

---

# 20. Configuration

CLI parameters SHALL be sufficient for `0.1.0`.

A project configuration file MAY subsequently be supported.

Potential format:

```toml
schema-version = 1
profile = "java-jacoco-v1"

[input]
path = "build/reports/jacoco/test/jacocoTestReport.xml"

[output]
format = "json"
```

Suggested filename:

```text
codegauge.toml
```

Configuration SHALL describe analysis.

It SHALL NOT contain project quality thresholds in the initial architecture.

---

# 21. No Policy Inside CodeGauge

The following configuration does NOT belong in CodeGauge core:

```yaml
crap:
  max: 6
```

CodeGauge reports:

```json
{
  "crap": 8.72
}
```

A consumer may decide:

```text
FAST:
  advisory

STANDARD:
  maximum = 10

FULL:
  maximum = 6
```

These decisions belong outside CodeGauge.

The architectural rule is:

> **Measurement and policy are separate concerns.**

CodeGauge MAY eventually support display filtering such as sorting or limiting output, but these options MUST NOT be represented as quality policy.

---

# 22. Result Contract

Machine-readable output SHALL use a versioned schema.

Example:

```json
{
  "schema": "codegauge-result/v1",

  "tool": {
    "name": "codegauge",
    "version": "0.1.0"
  },

  "profile": "java-jacoco-v1",

  "analysis": {
    "status": "complete",
    "symbols": 183
  },

  "summary": {
    "crap": {
      "max": 12.84,
      "mean": 2.73
    }
  },

  "symbols": [
    {
      "symbol": {
        "id": "java:com.acme.OrderService#calculateTotal(...)",
        "kind": "method",
        "path": "src/main/java/com/acme/OrderService.java",
        "name": "calculateTotal",
        "start_line": 42
      },

      "complexity": {
        "value": 7,
        "metric": "cyclomatic",
        "semantics": "jacoco-cyclomatic"
      },

      "coverage": {
        "ratio": 0.83,
        "covered": 83,
        "missed": 17,
        "semantics": "jacoco-instruction"
      },

      "metrics": {
        "crap": 7.24
      }
    }
  ],

  "provenance": {
    "provider": "jacoco",
    "input": {
      "path": "build/reports/jacoco/test/jacocoTestReport.xml",
      "sha256": "..."
    }
  }
}
```

---

# 23. Provenance

Every analysis SHOULD preserve enough provenance to explain how its numbers were produced.

Required provenance includes:

```text
CodeGauge version
profile
provider
provider semantics
input artifact identity
input artifact digest
analysis timestamp
```

When available:

```text
provider version
source revision
source tree state
configuration digest
```

Example:

```json
{
  "provenance": {
    "codegauge_version": "0.1.0",
    "profile": "java-jacoco-v1",
    "provider": "jacoco",
    "provider_version": "0.8.x",

    "artifact": {
      "path": "...",
      "sha256": "..."
    },

    "repository": {
      "commit": "abc123...",
      "dirty": false
    }
  }
}
```

CodeGauge SHALL NOT fabricate provenance that cannot be determined.

Unknown values SHALL remain absent or explicitly unknown.

---

# 24. Error Model

Operational failure and poor code quality MUST remain separate concepts.

CodeGauge itself does not produce quality `PASS` or `FAIL`.

Example execution outcomes:

```text
COMPLETE
PARTIAL
INPUT_NOT_FOUND
INVALID_INPUT
UNSUPPORTED_PROFILE
UNSUPPORTED_PROVIDER
INCOMPATIBLE_MEASUREMENTS
INTERNAL_ERROR
```

CLI exit codes SHOULD have stable documented semantics.

Example initial proposal:

```text
0   valid complete analysis
2   invalid CLI/configuration
3   input unavailable
4   unsupported profile/provider
5   invalid input artifact
6   incomplete/incompatible analysis
10  internal error
```

The exact values may be finalized during implementation but SHALL become stable public API once released.

Diagnostics SHOULD be written to stderr.

Machine-readable results SHOULD be written to stdout or an explicitly configured output path.

---

# 25. Determinism Requirements

Given identical:

```text
CodeGauge version
profile
configuration
input artifacts
```

CodeGauge MUST produce semantically identical metric results.

Variable metadata such as timestamps MUST NOT influence metric values.

CodeGauge SHALL NOT during normal analysis:

- access an LLM;
- fetch remote resources;
- install dependencies;
- resolve `"latest"` versions;
- download provider tools;
- modify project source code;
- mutate input artifacts.

All external tooling required to produce input artifacts SHALL execute before CodeGauge analysis.

---

# 26. Toolchain Reproducibility

The repository SHALL pin its Rust toolchain.

Example:

```text
rust-toolchain.toml
```

`Cargo.lock` SHALL be committed.

Release builds SHALL be produced from immutable Git revisions.

Published binaries SHOULD include cryptographic checksums.

The released binary version SHALL be included in all machine-readable output.

---

# 27. JSON Schema

Public machine contracts SHALL have versioned JSON schemas.

Rust types SHALL be the authoritative domain representation.

Schema generation SHOULD be automated from Rust types when practical.

Expected schemas:

```text
codegauge-result-v1.schema.json
codegauge-error-v1.schema.json
```

Breaking changes SHALL introduce a new contract version rather than silently changing `v1`.

---

# 28. Conformance Suite

`codegauge-conformance` SHALL verify that every provider obeys the same canonical contract.

Each provider SHOULD contain:

```text
input fixtures
expected canonical observations
expected CRAP results
invalid artifact fixtures
edge-case fixtures
```

Example:

```text
fixtures/
└── jacoco/
    ├── simple-method.xml
    ├── overloaded-methods.xml
    ├── zero-coverage.xml
    ├── full-coverage.xml
    ├── generated-code.xml
    └── malformed.xml
```

Golden tests SHALL validate stable JSON output.

---

# 29. Testing Strategy

The project SHALL use several deterministic testing layers.

### Unit tests

Cover:

- CRAP formula;
- value invariants;
- coverage normalization;
- complexity normalization;
- identifiers;
- profile validation.

### Property tests

Useful invariants include:

```text
For fixed CC:
increasing coverage cannot increase CRAP.

At coverage = 1:
CRAP = CC.

For fixed coverage:
increasing CC cannot decrease CRAP.
```

### Provider tests

Validate:

- artifact parsing;
- symbol mapping;
- overloaded methods;
- malformed input;
- missing counters;
- unsupported constructs.

### Golden tests

Verify canonical output remains stable.

### CLI tests

Verify:

- arguments;
- exit codes;
- JSON output;
- stderr behavior;
- deterministic serialization expectations.

---

# 30. Performance

CodeGauge is expected to operate primarily over analysis artifacts rather than recompiling projects.

Therefore initial performance targets are modest but explicit.

For typical reports, analysis SHOULD:

- use bounded memory;
- operate approximately linearly with report size;
- avoid loading unnecessary source trees;
- avoid network access.

Premature parallelization is NOT required.

Correctness and deterministic behavior take priority over raw throughput.

---

# 31. Security Model

CodeGauge processes potentially untrusted project artifacts.

Collectors MUST treat parsed files as untrusted input.

Requirements include:

- bounded parsing where practical;
- no execution of values embedded in reports;
- no shell interpolation from report contents;
- no implicit command execution;
- no automatic package installation;
- clear path handling;
- no network access during analysis by default.

---

# 32. Extensibility

Future metrics may include:

```text
cyclomatic complexity
coverage
CRAP
cognitive complexity
duplication measurements
maintainability metrics
other deterministic source metrics
```

However, CodeGauge SHALL NOT become a dumping ground for arbitrary quality checks.

A metric belongs in CodeGauge when:

1. it can be calculated deterministically;
2. its semantics can be described precisely;
3. its provenance can be preserved;
4. its result can be represented independently of policy.

---

# 33. Provider Extensibility

`0.x` SHALL use compile-time Rust provider crates.

Dynamic native plugin ABI support is explicitly deferred.

Rationale:

- Rust has no stable language-level ABI for arbitrary plugin crates;
- dynamic loading adds compatibility and security complexity;
- current provider count is expected to remain manageable;
- one released binary simplifies harness consumption.

If third-party providers become important, a future RFC SHOULD evaluate a process-based provider protocol such as:

```text
CodeGauge
    │
    │ JSON request
    ▼
external provider process
    │
    │ canonical observations
    ▼
CodeGauge
```

This is preferred over prematurely creating a dynamic Rust ABI.

---

# 34. Harness Integration

The consuming agent harness SHALL treat CodeGauge as an external deterministic capability provider.

Conceptually:

```text
Harness Capability Registry
          │
          ▼
CodeGauge Adapter
          │
          ▼
codegauge analyze --format json
          │
          ▼
codegauge-result/v1
          │
          ▼
Harness normalized evidence
          │
          ▼
Policy evaluator
          │
          ▼
FSM
```

The harness adapter SHOULD know only:

- executable location;
- CodeGauge version;
- command contract;
- result schema.

The harness SHOULD NOT know:

- how JaCoCo represents complexity;
- how Go computes source complexity;
- how Python functions are correlated;
- how CRAP is calculated internally.

---

# 35. Distribution

Primary distribution SHOULD use versioned binary releases.

Target platforms initially:

```text
macOS arm64
macOS x86_64
Linux x86_64
Linux arm64
```

Windows support MAY follow.

Potential additional distribution mechanisms:

```text
cargo install
Homebrew
package managers
container image
GitHub Action
```

These are secondary to the stable CLI contract.

Consumers SHOULD pin exact CodeGauge versions.

---

# 36. Licensing

The project SHOULD be designed as independently publishable open-source tooling.

If made public, a permissive Rust-compatible license such as:

```text
MIT OR Apache-2.0
```

is recommended.

Final licensing remains a repository-level decision and is not required for the architectural implementation.

---

# 37. Versioning

CodeGauge SHOULD follow semantic versioning.

Three independent concepts require versioning:

### Tool version

```text
codegauge 0.3.0
```

### Result contract

```text
codegauge-result/v1
```

### Metric profile

```text
java-jacoco-v1
```

These MUST NOT be conflated.

A new CodeGauge release does not necessarily imply a new result schema.

A changed provider semantic SHOULD result in a new profile even when the overall result schema remains unchanged.

---

# 38. Initial MVP — `0.1.0`

The first release SHALL deliberately remain small.

## Included

```text
Rust Cargo workspace

codegauge-core
codegauge-model
codegauge-application
codegauge-cli
codegauge-provider-jacoco
codegauge-conformance

CRAP formula

java-jacoco-v1 profile

JaCoCo XML parsing

canonical symbol model

canonical measurement model

JSON output

provenance

artifact hashing

JSON schema

golden tests

CLI integration tests

version command
profiles command
analyze command
```

## Explicitly excluded

```text
Kotlin-specific collector
Go
Python
JavaScript/TypeScript
Rust target analysis
mutation testing
DRY
quality thresholds
policy engine
automatic dependency installation
automatic test execution
automatic language detection
dynamic plugin system
LLM integration
```

---

# 39. Proposed Roadmap

## `0.1.x` — Foundation

```text
Core architecture
Java + JaCoCo
CRAP
stable CLI
stable result v1
```

## `0.2.x` — JVM Expansion

Evaluate:

```text
Kotlin
Kover
mixed Java/Kotlin projects
JVM profile semantics
```

The outcome MAY be:

```text
kotlin-kover-v1
```

rather than pretending Java and Kotlin bytecode semantics are identical.

## `0.3.x` — Go

Potential architecture:

```text
Go AST complexity
+
Go coverage artifact
+
symbol correlation
```

## `0.4.x` — Python

Introduce a collector capable of correlating:

```text
source function complexity
+
coverage artifact
```

## `0.5.x` — JavaScript / TypeScript

Introduce:

```text
AST/source complexity
+
Istanbul-compatible coverage
```

Exact roadmap ordering may change based on actual demand.

---

# 40. Acceptance Criteria

CodeGauge `0.1.0` is considered complete when:

1. The project builds as an independent Rust Cargo workspace.
2. `codegauge-core` has no dependency on language-specific providers.
3. The CRAP formula exists in exactly one production implementation.
4. A valid JaCoCo XML artifact can be analyzed.
5. Complexity and coverage are associated at method/symbol granularity.
6. CRAP is calculated for each compatible symbol.
7. Every result declares the `java-jacoco-v1` profile.
8. Output validates against `codegauge-result/v1`.
9. The input artifact SHA-256 is present in provenance.
10. CodeGauge does not run Maven, Gradle, tests, or JaCoCo itself.
11. CodeGauge does not install dependencies.
12. No LLM participates in metric extraction or calculation.
13. No CRAP quality threshold is applied.
14. Unsupported or malformed input results in a deterministic structured error.
15. Identical inputs produce identical metric values.
16. Golden fixtures verify output stability.
17. The CLI can execute without any agent harness present.
18. The harness can consume CodeGauge solely through the public CLI/result contract.

---

# 41. Gherkin Scenarios

## Scenario: Calculate CRAP from valid JaCoCo evidence

```gherkin
Feature: Calculate CRAP metrics

  Scenario: Analyze a valid Java JaCoCo report
    Given a valid JaCoCo XML report
    And the profile "java-jacoco-v1"
    When CodeGauge analyzes the report
    Then every supported method with complexity and coverage evidence is identified
    And CRAP is calculated using "crap-original-v1"
    And the result conforms to "codegauge-result/v1"
    And the analysis completes successfully
```

## Scenario: Full coverage

```gherkin
Scenario: Calculate CRAP for a fully covered method
  Given a method with cyclomatic complexity 7
  And instruction coverage equal to 1.0
  When CodeGauge calculates CRAP
  Then the CRAP value is 7
```

## Scenario: Deterministic result

```gherkin
Scenario: Analyze identical evidence twice
  Given the same CodeGauge version
  And the same profile
  And the same input artifact
  When CodeGauge performs the analysis twice
  Then both analyses produce identical metric values
```

## Scenario: Artifact is missing

```gherkin
Scenario: Input artifact does not exist
  Given an input path that does not exist
  When CodeGauge attempts analysis
  Then no metric result is fabricated
  And the execution reports INPUT_NOT_FOUND
  And the process exits with the documented input-unavailable exit code
```

## Scenario: Artifact is malformed

```gherkin
Scenario: JaCoCo artifact is malformed
  Given a malformed JaCoCo XML artifact
  When CodeGauge analyzes the artifact
  Then the artifact is rejected
  And no CRAP score is fabricated
  And the execution reports INVALID_INPUT
```

## Scenario: Unsupported profile

```gherkin
Scenario: Requested profile is unsupported
  Given the profile "unknown-provider-v1"
  When CodeGauge starts analysis
  Then the execution reports UNSUPPORTED_PROFILE
  And CodeGauge does not attempt heuristic analysis
```

## Scenario: No quality policy

```gherkin
Scenario: A method has a high CRAP score
  Given a valid method whose calculated CRAP is 35
  When CodeGauge analyzes the method
  Then CodeGauge reports CRAP as 35
  And CodeGauge does not report the code as passed or failed
```

## Scenario: Provenance

```gherkin
Scenario: Produce auditable evidence
  Given a valid JaCoCo input artifact
  When CodeGauge completes analysis
  Then the result contains the CodeGauge version
  And the result contains the metric profile
  And the result contains the provider
  And the result contains the SHA-256 digest of the analyzed artifact
```

---

# 42. Architectural Decisions

The following decisions are considered foundational.

### ADR-001 — Independent repository

CodeGauge is maintained independently from agent harnesses.

### ADR-002 — Rust workspace monorepo

Providers live as crates in a common Cargo workspace.

### ADR-003 — One canonical CRAP implementation

The formula belongs only to `codegauge-core`.

### ADR-004 — Providers collect; core calculates

Collectors do not independently implement derived metric formulas.

### ADR-005 — Artifact-first analysis

CodeGauge consumes structured analysis artifacts instead of controlling test execution.

### ADR-006 — No quality policy

Thresholds and workflow decisions belong to consumers.

### ADR-007 — Semantic metric profiles

Metric provenance includes a versioned profile describing provider semantics.

### ADR-008 — CLI + JSON as integration boundary

External consumers integrate through a released executable and stable schema.

### ADR-009 — No dynamic Rust plugin ABI in v1

Provider crates are compiled into the distributed executable.

### ADR-010 — No runtime dependency installation

CodeGauge never resolves or downloads external metric tools while analyzing.

---

# 43. Design Invariants

These invariants SHOULD be treated as architectural tests where practical.

```text
codegauge-core
MUST NOT depend on provider crates.

provider crates
MAY depend on model/core abstractions.

CLI
MUST NOT contain provider-specific business rules.

policy
MUST NOT exist inside metric calculation.

network access
MUST NOT be required for analysis.

LLM
MUST NOT be required for analysis.

metric results
MUST preserve semantics and provenance.

derived metrics
MUST NOT combine incompatible granularities silently.

provider changes that alter semantics
MUST introduce a new profile version.
```

---

# 44. Future Vision

Although CRAP is the initial use case, CodeGauge is intentionally named around measurement rather than around one metric.

The long-term conceptual model is:

```text
                         CodeGauge
                             │
        ┌────────────────────┼─────────────────────┐
        │                    │                     │
   Complexity            Coverage             Other source
                                             observations
        │                    │                     │
        └──────────────┬─────┘                     │
                       │                           │
                       ▼                           ▼
                 Derived Metrics              Metrics
                       │
              ┌────────┴────────┐
              │                 │
             CRAP           future metrics
```

Consumers remain free to decide which measurements matter.

This creates a reusable deterministic foundation for:

```text
AI agent harnesses
CI pipelines
local development tooling
pre-commit workflows
IDEs
quality dashboards
repository audits
code review automation
```

without placing any of those workflows inside CodeGauge itself.

---

# 45. Final Principle

The project is governed by one primary architectural rule:

> **CodeGauge owns measurement, not judgement.**

More broadly:

```text
Providers own extraction.
CodeGauge owns metrics.
Evidence owns truth.
Consumers own policy.
FSMs own transitions.
LLMs own reasoning.
Humans own exceptions.
```

That separation is the foundation on which CodeGauge should evolve.