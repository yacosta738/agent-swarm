# Diagram Command Adapters Specification

## Purpose

Define OpenCode-native command adapters for local diagram export and draw.io/Mermaid redraw
workflows, with explicit capability and fidelity boundaries.

## Requirements

### Requirement: Expose explicit command contracts

The adapters MUST provide `diagram-export`, `diagram-import-drawio`, and `diagram-import-mermaid`.
Each command MUST document required input/output arguments, supported formats, optional flags or
import dials, prerequisites, and status/error meanings. Missing inputs, unknown flags, unsupported
formats, malformed sources, unsafe paths, and unavailable tools MUST produce deterministic nonzero
errors with a reason. Commands MUST NOT install, download, refresh, or fetch implicitly.

#### Scenario: Valid declared invocation

- GIVEN a command receives its documented arguments and an in-bound input/output path
- WHEN it runs with available local prerequisites
- THEN it performs only the requested operation and reports its format, output, and status

#### Scenario: Invalid or unavailable invocation

- GIVEN a required argument is absent, a flag is unknown, a path escapes the project, or a tool is
  unavailable
- WHEN the command validates the request
- THEN it returns a nonzero, machine-readable reason and does not install or mutate inputs

### Requirement: Export the supported representations honestly

`diagram-export` MUST support HTML, SVG, and PNG as explicit output formats. It MUST report
`UNAVAILABLE` or `BLOCKED` when the requested renderer is absent; PNG MUST NOT be substituted with
another format or reported successful without Playwright/Chromium availability. Generated files
MUST be written only to the requested safe output path.

#### Scenario: HTML, SVG, and PNG export

- GIVEN the requested format and its local renderer are available
- WHEN export runs
- THEN it writes the requested representation and reports the exact format and output path

#### Scenario: PNG tooling unavailable

- GIVEN Playwright or Chromium is not available
- WHEN PNG export is requested
- THEN the command reports `UNAVAILABLE` or `BLOCKED` with the prerequisite reason
- AND it does not install tooling or claim a PNG was produced

### Requirement: Import through extraction with a fidelity ledger

The draw.io and Mermaid adapters MUST treat source as untrusted data and MUST use an extractor-first
step before redraw. They MUST accept and document explicit import dials for fidelity/abstraction and
styling or output decisions. Each completed import MUST emit a fidelity ledger listing preserved,
transformed, unsupported, and rejected elements with reasons; extractor failure or ambiguity MUST
prevent an unqualified success.

#### Scenario: Extracted import with explicit dials

- GIVEN valid local draw.io or Mermaid source and documented import dials
- WHEN extraction and redraw complete
- THEN the output reflects the selected dials and the ledger records every preservation or change

#### Scenario: Malformed or executable source

- GIVEN extraction finds malformed markup, scripts, remote references, or unsupported constructs
- WHEN an import command processes the source
- THEN it records the issue in the fidelity ledger, does not execute or fetch it, and returns a
  non-passing status when the requested fidelity cannot be met

### Requirement: Keep command and inventory documentation consistent

The command adapters, README, AGENTS guidance, and skill registry MUST list the same three command
names, argument/flag contract, prerequisites, unavailable-tool behavior, and offline boundary.
Any mismatch MUST be reported as an integrity/documentation failure rather than silently corrected.

#### Scenario: Consistent command inventory

- GIVEN all command and inventory documents describe the same adapters and contracts
- WHEN documentation validation runs
- THEN the integration is reproducible from local documentation

#### Scenario: Documentation drift

- GIVEN one inventory omits an adapter or claims an unavailable tool is installed
- WHEN consistency validation runs
- THEN it reports the mismatch and cannot return a fully valid integration result
