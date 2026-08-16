# Acceptance QA Report: integrate-diagram-design-skills

## Identity

- **Change:** `integrate-diagram-design-skills`
- **Mode:** `openspec`
- **QA phase:** `sdd-qa`
- **Date:** `2026-08-15`
- **Branch:** `feature/diagram-design-docs`
- **HEAD:** `06f69d99ea9c27c0af8ad5f67e58f42f75fc0f4b`
- **Upstream pin:** `a5e3978088cf89c7caff5c20cabd99fbc2a301de` / `2.3.5`
- **Technical verification handoff:** `verify-report.md` verdict `PASS WITH WARNINGS`; this report is an independent acceptance gate and does not convert technical evidence into product acceptance.

## Sources of Truth

- `openspec/config.yaml`
- `openspec/changes/integrate-diagram-design-skills/proposal.md`
- `openspec/changes/integrate-diagram-design-skills/specs/diagram-design-skill/spec.md`
- `openspec/changes/integrate-diagram-design-skills/specs/diagram-command-adapters/spec.md`
- `openspec/changes/integrate-diagram-design-skills/design.md`
- `openspec/changes/integrate-diagram-design-skills/tasks.md`
- `openspec/changes/integrate-diagram-design-skills/apply-progress.md`
- `openspec/changes/integrate-diagram-design-skills/verify-report.md`
- `openspec/changes/integrate-diagram-design-skills/state.yaml`
- `openspec/quality-runner.json`
- `README.md`, `AGENTS.md`, `.agents/skill-registry.md`
- `commands/diagram-export.md`, `commands/diagram-import-drawio.md`, and `commands/diagram-import-mermaid.md`

## Technical Verification Handoff

The preceding phase reports the following as technical/conformance evidence only:

- The pinned snapshot contains 150 manifest entries, 147 upstream Agent Skill resource files, and matching hashes/sizes/bytes for the checked-in snapshot.
- Python extractor/self-check smoke covered temporary valid, malformed, hostile, and representative HTML cases; this is not product or operator acceptance.
- Documentation and registry consistency checks covered the three command names, pin, path, prerequisites, and offline boundary; static inspection is not a QA pass.
- The six implementation commits were reviewed as the approved feature-branch chain: `7ec47c1`, `379fd57`, `1557a27`, `3c0ec93`, `7ca50cf`, and `06f69d9`.
- `openspec/quality-runner.json` exists but is disabled. No runner envelope exists; the required visible `fallback` path is recorded below.

## Target and Environment

- **Target:** No application, product surface, operator workflow, or effective installed OpenCode harness was supplied for acceptance testing. The repository checkout is a conformance/evidence surface only. A shell existence probe found `~/.config/opencode`, but it was not treated as the target, launched, installed, synchronized, or otherwise exercised.
- **Environment:** macOS checkout at `/Users/acosta/Dev/agent-swarm/agent-harness`; OpenSpec root at `/Users/acosta/Dev/agent-swarm/openspec`; Python `3.14.6` available.
- **Credentials/permissions:** No credentials are applicable. No installation, network fetch, browser download, command mutation, or snapshot mutation was attempted.
- **Runner status:** `quality-runner/v1` is disabled (`enabled: false`); no general repository test runner, build, or coverage command is configured.
- **Renderer status:** Python Playwright is unavailable and no `chromium`, `chromium-browser`, `google-chrome`, or `google-chrome-stable` executable was found. The capability is `UNAVAILABLE` with reason `PLAYWRIGHT_OR_CHROMIUM_UNAVAILABLE`.
- **QA boundary:** The Markdown adapters are model-driven prompts, not deterministic CLIs. Their invocation, redraw, generated artifacts, and emitted fidelity ledgers were not executed. No browser/runtime/product acceptance is claimed.

## Capability Decision Matrix

| Capability | Availability | Selected? | Decision and rationale |
|---|---|---:|---|
| Repository filesystem/Git/doc inspection | Available | Yes, conformance handoff only | Used to identify the change, target boundary, branch, commits, and prior evidence. Static evidence cannot produce a QA `PASS`. |
| Python 3 standard-library extractors and `self_check.py` | Available | Yes, technical evidence only | Prior apply/verify smoke evidence is reusable for conformance; it is not user/operator acceptance. No source or snapshot was mutated. |
| OpenSpec quality runner/FSM | Unavailable | No | `quality-runner.json` is disabled; record `fallback`, not a fabricated runner result. |
| Repository test/build/coverage runner | Unavailable | No | No root runner or build manifest is configured and strict TDD is disabled by policy. |
| Markdown command invocation | Unavailable as executable capability | No | The three command files are model-driven adapters, not deterministic CLIs; no OpenCode session or command executor target was available. |
| Browser / Playwright / Chromium rendering | Unavailable | No | Playwright and Chromium are absent. PNG/HTML+PNG behavior was not invoked and no substitute format was claimed. |
| Effective installed OpenCode skill scanner | Unavailable as an acceptance target | No | No installed harness target was supplied or exercised; the presence of a home-directory path is not evidence of installation acceptance. |
| API/client, data, persistence, and state-transition target | Not applicable | No | This change adds local configuration, documentation, skills, and prompt adapters; it has no application API or persistent product state under test. |
| Manual/operator/exploratory session | Unavailable | No | There is no product operator or installed harness workflow to exercise. |
| Accessibility, responsive, locale/internationalization | Rejected as not applicable | No | There is no product UI or runtime surface in this change. The bundled HTML examples are resources, not an application under test. |

## Scenario Matrix

Every acceptance scenario is intentionally `NOT TESTED`. References to technical evidence below are handoff evidence only and are not scenario passes.

| ID | Capability | Acceptance scenario | Result | Evidence or reason |
|---|---|---|---|---|
| DS-1 | Effective installed skill scanner | Offline native discovery finds `diagram-design` at the canonical installed path with unchanged plugin configuration. | `NOT TESTED` | `E-03` records checkout path/frontmatter and unchanged `opencode.json`; no effective installed OpenCode target or scan was exercised. |
| DS-2 | Effective installed skill scanner | A moved skill or mismatched frontmatter identity is rejected by the integrity check. | `NOT TESTED` | `E-03` records a temporary negative identity fixture at technical-verification level; no acceptance target or OpenCode integrity run was available. |
| DS-3 | Repository snapshot conformance | A complete pinned snapshot resolves all resources and agrees on commit, version, MIT, and third-party notices. | `NOT TESTED` | `E-03`/`E-04` report the 150-file pinned snapshot and manifest evidence; static/filesystem conformance is not product acceptance. |
| DS-4 | Repository snapshot conformance | A missing or drifted resource, notice, commit, or version returns a non-passing integrity result identifying the item. | `NOT TESTED` | `E-03` records a temporary manifest-drift check; no acceptance runner or installed target was exercised. |
| DS-5 | Interactive skill/session | A diagram request without an approved style guide pauses at the default style-guide gate. | `NOT TESTED` | `E-03` records source-order/default-token conformance; no agent session or user-visible skill interaction was available. |
| DS-6 | Import trust-boundary session | Hostile Mermaid/draw.io input is surfaced as unsafe or unsupported data without execution or fetching. | `NOT TESTED` | `E-04` reports extractor/self-check hostile fixtures; adapter-level and user/operator behavior were not executed. |
| DS-7 | Operator documentation workflow | A clean offline checkout exposes the same pinned capability and prerequisites from local inventory and documented paths. | `NOT TESTED` | `E-03` records static README/AGENTS/registry consistency; no operator or installed-checkout workflow was observed. |
| DS-8 | Installed rollback workflow | Reverting the change preserves unrelated skills and the existing plugin entry with no migration. | `NOT TESTED` | `E-03` records Git-boundary rollback evidence; effective installed-path rollback was not performed. |
| DA-1 | Markdown command execution | A valid declared command invocation performs only the requested operation and reports format, output, and status. | `NOT TESTED` | `E-03` records contract text only; Markdown adapters are not executable CLIs and no OpenCode command session was available. |
| DA-2 | Markdown command execution | Missing/unknown/unsafe/unavailable invocation returns a deterministic nonzero reason without installation or input mutation. | `NOT TESTED` | `E-04` reports backend extractor negative probes, but adapter validation and command exit behavior were not invoked. |
| DA-3 | Export renderer | HTML, SVG, and PNG exports produce the requested exact representation and output path when prerequisites are available. | `NOT TESTED` | No executable export adapter or renderer target was available; no output artifact was observed. |
| DA-4 | PNG renderer capability | With Playwright or Chromium absent, PNG export reports `UNAVAILABLE`/`BLOCKED` with the prerequisite reason and does not substitute SVG. | `NOT TESTED` | `E-07` observed `PLAYWRIGHT_OR_CHROMIUM_UNAVAILABLE`; the Markdown command was not invoked, so no command result is claimed. |
| DA-5 | Import/redraw session | A valid local import with explicit dials produces a redraw and a complete fidelity ledger. | `NOT TESTED` | `E-04` reports extractor fixtures only; model-driven redraw and ledger emission were not run. |
| DA-6 | Import trust-boundary session | Malformed or executable source is ledgered, not executed/fetched, and cannot return an unqualified success. | `NOT TESTED` | `E-04` reports extractor rejection/inert-data behavior; adapter-level ledger and redraw status were not observed. |
| DA-7 | Documentation/operator workflow | Commands, README, AGENTS guidance, and registry describe the same command contracts and offline boundary. | `NOT TESTED` | `E-03` reports static consistency evidence; no operator workflow or installed documentation scan was exercised. |
| DA-8 | Documentation drift detection | Omitting a command or claiming an unavailable tool is installed is reported as a documentation failure. | `NOT TESTED` | `E-03` reports a temporary simulated omission check; static inspection is not an acceptance result. |

### Cross-cutting coverage

| Category | Result | Evidence or non-applicability reason |
|---|---|---|
| Happy-path user/operator outcome | `NOT TESTED` | No product or operator target exists; documented contracts and extractor smoke are technical handoff evidence only. |
| Negative and boundary input | `NOT TESTED` | Technical extractor negatives exist in `E-04`, but no acceptance command/session executed the adapter boundary. |
| Repeated, interrupted, or resumed operation | `NOT TESTED` | No executable command session, durable process, or stateful target exists in this change. |
| Unauthorized or security-sensitive behavior | `NOT TESTED` | Untrusted-source cases are technically covered in `E-04`; no runtime/operator acceptance session was available. |
| Browser and renderer behavior | `NOT TESTED` | No browser target; Playwright/Chromium is `UNAVAILABLE`. |
| Accessibility behavior | `NOT TESTED` | No product UI or browser-rendered acceptance surface is in scope. |
| Responsive behavior | `NOT TESTED` | No product UI target is in scope. |
| Locale/internationalization behavior | `NOT TESTED` | No localized product surface or user-facing runtime target is in scope. |
| API/client, data, persistence, and state transitions | `NOT TESTED` | This is a local configuration/skills integration with no application API or persistent product state. |
| Manual and exploratory behavior | `NOT TESTED` | No operator, installed harness, or interactive OpenCode target was supplied. |

## Evidence References

- **E-01:** `openspec/config.yaml` — QA verdict rules, prohibition on static-inspection passes, visible fallback guidance, and archive exception for documentation/configuration.
- **E-02:** `openspec/quality-runner.json` — `quality-runner/v1` present but `enabled: false`; no deterministic runner envelope.
- **E-03:** `verify-report.md` — technical handoff, spec matrix, 150-entry snapshot, documentation/configuration checks, six-commit chain, and explicit product/operator boundary.
- **E-04:** `apply-progress.md` § Phase 3 focused validation — Python extractor/self-check evidence, hostile-input boundary, model-driven adapter limitation, and no product acceptance claim.
- **E-05:** Read-only Git inspection — branch `feature/diagram-design-docs`, HEAD `06f69d99ea9c27c0af8ad5f67e58f42f75fc0f4b`, parent `7ca50cf7a38d8e4116e1bc2f9906b0616e2f1bfe`, and the six implementation commits.
- **E-06:** `README.md`, `AGENTS.md`, `.agents/skill-registry.md`, and the three command files — local inventory and documented offline/model-driven boundaries.
- **E-07:** Read-only environment probe — Python `3.14.6`; Playwright and local Chromium executables unavailable; no installation or download attempted.

## Untested and Blocked Scope

| Scope | Reason | Re-run prerequisite |
|---|---|---|
| Product, operator, and application acceptance | No application or operator target exists for this configuration/skills change. | Provide a real acceptance target and an operator scenario; rerun this phase without treating repository inspection as acceptance. |
| Effective `~/.config/opencode` discovery and rollback | The home-directory path was not supplied as an exercised installation target; no install/sync/launch/rollback was performed. | Provide an isolated installed harness path or explicitly authorize a controlled installed-target session. |
| Markdown adapter invocation, redraw, output, and fidelity ledger | Adapters are model-driven Markdown prompts, not deterministic CLIs; no OpenCode session was available. | Run each command in a real, controlled OpenCode session or add a separately testable executable adapter. |
| HTML/SVG/PNG renderer acceptance | No executable export target was available; PNG renderer capability is `UNAVAILABLE`. | Provide local renderer execution; for PNG, provide Python Playwright and launchable Chromium without substituting SVG. |
| Browser, accessibility, responsive, locale, API, data, persistence, and interrupted/state-transition behavior | No runtime product surface or persistent application state is in scope. | Re-run only if a runtime target is introduced. |
| Test-runner/TDD acceptance | Repository policy declares no runner and strict TDD false. | Add and configure an appropriate runner in a separate change if required. |

No scenario was marked `BLOCKED`: there was no external execution attempt that was prevented. The disabled runner and missing renderer are recorded as `fallback`/`UNAVAILABLE`, and the affected QA scenarios remain `NOT TESTED` under the required mapping.

## Findings

No unresolved `CRITICAL`, `P0`, or `P1` findings were identified.

| ID | Severity | Scenario / location | Evidence | Status |
|---|---|---|---|---|
| QA-001 | `P2` | Product/operator acceptance and installed target | No application, operator, or exercised installed OpenCode target; DS-1–DS-8 and DA-1–DA-8 remain `NOT TESTED`. | Accepted limitation under the explicit non-runtime documentation/configuration exception; rerun prerequisite recorded. |
| QA-002 | `P2` | `commands/diagram-*.md` invocation/redraw | The adapters document contracts but are model-driven prompts, not deterministic CLIs; no command exit, redraw artifact, or ledger was observed. | Known capability boundary; no implementation fix made; follow-up only if executable command acceptance becomes a requirement. |
| QA-003 | `P3` | `diagram-export` PNG path | `PLAYWRIGHT_OR_CHROMIUM_UNAVAILABLE`; Playwright and Chromium were not available. | Environment limitation; no install, browser download, format substitution, or false success. |
| QA-004 | `P3` | Runner/TDD | `quality-runner.json` disabled; no root test/build/coverage runner. | Policy-confirmed limitation; no TDD or product-acceptance claim. |
| QA-005 | `P3` | Representative asset audit | Verify handoff reports 102/104 generic `self_check.py` asset checks passing; `icons.html` and `index.html` are gallery/index surfaces with expected generic-check limitations. | Acknowledged upstream/gallery warning; no release-blocking severity assigned. |

## Verdict

`NOT TESTED`

### Rationale

The QA verdict is deliberately `NOT TESTED`, because this repository contains configuration, documentation, vendored skills, scripts, and model-driven command prompts rather than an application under test. No static inspection, pinned snapshot check, Python smoke check, documentation check, or Git evidence is promoted to product/operator acceptance. PNG is `UNAVAILABLE`, Markdown command invocation/redraw is `NOT TESTED`, the quality runner is a visible `fallback`, and no browser/runtime acceptance was claimed.

`openspec/config.yaml` explicitly permits `allow_non_runtime_exception: true` for `documentation` and `configuration`, requires a warning on that exception, and separately states that acceptance-relevant `NOT TESTED` normally blocks archive. This change is the requested non-runtime configuration/documentation/skills integration, so the exception is applicable without changing the QA verdict. The warning in this report must remain visible to archive/reviewers.

## Limitations and Handoff

- QA did not modify source code, commands, the pinned snapshot, dependencies, or renderer/tooling state.
- This report is the acceptance audit record; it is not a claim that the harness has product acceptance.
- Preserve the `NOT TESTED` verdict, `UNAVAILABLE` renderer capability, and visible non-runtime exception warning during archive.
- Archive may proceed to `sdd-archive` under the explicit documentation/configuration exception because `verify-report.md` exists with `PASS WITH WARNINGS`, `qa-report.md` exists, and no unresolved `CRITICAL`/P0/P1 findings exist.
- If runtime acceptance is later required, rerun QA with an exercised installed OpenCode target, a real command/session execution path, and local Playwright/Chromium for PNG; do not infer acceptance from the existing technical evidence.
