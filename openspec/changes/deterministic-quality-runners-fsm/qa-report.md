# Acceptance QA Report: deterministic-quality-runners-fsm

## Identity

- **Change**: `deterministic-quality-runners-fsm`
- **Mode**: `openspec`
- **QA phase**: `qa` (acceptance re-run; previous run 2026-08-09 returned `BLOCKED`)
- **Date**: 2026-08-16
- **Verdict**: `BLOCKED`
- **Boundary**: This report evaluates observable behavior of the repository-local harness CLI and its configured fixtures. It does not claim acceptance of a product or deployed operator surface.

## Sources of Truth and Technical Verification Handoff

- **Proposal**: `openspec/changes/deterministic-quality-runners-fsm/proposal.md`
- **Delta specifications**:
  - `openspec/changes/deterministic-quality-runners-fsm/specs/quality-runners/spec.md`
  - `openspec/changes/deterministic-quality-runners-fsm/specs/sdd-workflow-fsm/spec.md`
  - `openspec/changes/deterministic-quality-runners-fsm/specs/acceptance-qa/spec.md`
- **Design**: `openspec/changes/deterministic-quality-runners-fsm/design.md`
- **Tasks**: `openspec/changes/deterministic-quality-runners-fsm/tasks.md`
- **Apply handoff**: `openspec/changes/deterministic-quality-runners-fsm/apply-progress.md`
- **Technical verification**: `openspec/changes/deterministic-quality-runners-fsm/verify-report.md` (`PASS WITH WARNINGS`, 2026-08-09)
- **Repository policy**: `openspec/config.yaml` (`static_inspection_verdict: prohibited`, blocked/not-tested policy scope)
- **Operator documentation**: `README.md`, `commands/sdd-continue.md`, SDD verify/QA prompts and skills
- **Relevant executable surfaces**: `agent-harness/scripts/sdd-quality-runner.mjs`, `agent-harness/scripts/sdd-fsm.mjs`, `agent-harness/scripts/sdd-{quality,fsm,smoke,handoff,control-plane,capability-adapters}-smoke.sh`, `agent-harness/scripts/sdd-runner-lib/`, `agent-harness/scripts/fixtures/`

`verify-report.md` hands off technical conformance as `PASS WITH WARNINGS`; it explicitly does not claim product/operator acceptance. QA preserves that boundary. The repository has no general test runner and no final application under test.

## Target and Environment

- **Target**: Repository-local standalone quality-runner/FSM commands (`sdd-quality-runner.mjs`, `sdd-fsm.mjs`) operating on the supplied Node, Python, runner, and temporary FSM fixtures under the `agent-harness` submodule checkout. No separate product target or deployed operator target was supplied.
- **Environment**: `/Users/acosta/Dev/agent-swarm` (change artifacts) with implementation at `/Users/acosta/Dev/agent-swarm/agent-harness` (submodule, HEAD `a6db498`, implementation commit `7589bcf "feat(sdd): add deterministic quality runner and FSM"`). macOS Darwin arm64, Node `v24.19.0`, GNU Bash 3.2.57, `python3` 3.14.7. All scenarios executed live from the checkout.
- **Credentials/permissions**: No credentials required for local fixtures. External distribution inspection was read-only; Dotter was not executed and no global configuration was changed, per authorization constraint.
- **Limitations**:
  - The root manifest and FSM policy are intentionally disabled (`enabled: false`), so direct root execution is compatibility `NOT_TESTED`/`fallback`, not deterministic enforcement.
  - There is no product UI, API, deployed service, or independent verify/QA report generator to exercise end-to-end acceptance.
  - The dotfiles submodule pointer and effective `~/.config/opencode` do not contain this change; rollout is outside this checkout and was not modified.
  - My direct runner invocations write fixture evidence under `scripts/fixtures/*/artifacts/`; the submodule worktree was verified clean after the run (0 untracked files after cleanup).

## Capability Inventory

| Capability | Availability | Selected? | Rationale / rejection reason |
|---|---:|---|---|
| Repository-local process/CLI execution | available | yes | Narrow executable path for observable harness operator behavior. |
| Configured Node fixture capability | available | yes | Exercises explicit `argv` configuration and result identity. |
| Configured Python fixture capability | available | yes | Exercises a different stack without inference. |
| Runner negative/boundary fixtures | available | yes | Exercises unavailable executable, timeout, parser, threshold, redaction, environment isolation, path safety, and skip. |
| FSM temporary-project persistence | available | yes | Exercises transitions, archive gates, legacy state, idempotency, locking, and atomic writes in scratch projects. |
| JSON/Markdown evidence inspection after execution | available | yes | Evidence was inspected only after commands ran; static source inspection was not promoted to PASS. |
| Browser / Playwright / Chrome | available | no | Rejected: the change exposes CLI/configuration, not a browser surface. |
| API/client or deployed service | unavailable | no | No service or external target exists in this repository. |
| Data-store/persistence integration | unavailable | no | No product data store; temporary filesystem state was selected instead. |
| Accessibility and responsive checks | unavailable | no | No UI target. |
| Locale/internationalization checks | unavailable | no | No locale-sensitive product surface or acceptance requirement. |
| Manual/exploratory CLI checks | available | yes | Direct invocations supplemented the smoke scripts for negative classification, archive gates, and fallback behavior. |
| External checkout/submodule/effective-config inspection | available | yes, read-only | Used filesystem and Git metadata only; no Dotter run or modification. |
| Dotter materialization | available but not authorized | no | Deliberately rejected because the request forbids Dotter/global changes without authorization. |
| Static source/report inspection as acceptance evidence | available | no | Used for context and limitations only; it cannot produce PASS under QA policy. |
| General repository test runner | unavailable | no | `openspec/config.yaml` documents that none exists. |

## Scenario Matrix

`PASS` in this table means the local executable harness scenario produced the expected observable outcome in this re-run. It is not a product-acceptance claim. Runner statuses embedded in evidence remain authoritative and are shown explicitly.

| ID | Capability | Acceptance scenario | Result | Evidence or reason |
|---|---|---|---|---|
| QA-RUN-01 | Node/Python fixture execution | An operator configures two different stacks and obtains the declared capability result without stack inference. | **PASS** | Re-run `node scripts/sdd-quality-runner.mjs run --project scripts/fixtures/node-project --capability test --json` → exit `0`, runner `PASS/policy_satisfied`, capability `test`, argv `node -e ...`, cwd `.`, config digest recorded. Python fixture → same `PASS/policy_satisfied`, exit `0`, declared `python3` argv; no stack-specific branch. |
| QA-RUN-02 | Repeatability | Identical normalized inputs produce the same status and reason on repeated runs. | **PASS** | Re-run of `threshold` capability twice → both returned runner `FAIL/threshold_rejected` and parsed value `3` with `min: 4`. |
| QA-RUN-03 | Negative classification | A missing executable is not converted into a pass. | **PASS** | Re-run `missing` capability → `UNAVAILABLE/executable_not_found`, stderr ENOENT, no `PASS`. |
| QA-RUN-04 | Timeout classification | A capability exceeding its timeout is not converted into a pass. | **PASS** | Re-run `timeout` capability → `BLOCKED/timeout`, `timed_out: true`, no `PASS`. |
| QA-RUN-05 | Parser/exit classification | Parser rejection and non-accepted exit are not converted into a pass. | **PASS** | Re-run `parser` → `FAIL/parser_rejected`, exit `0`, `parsed: false`; `secret` fixture → `FAIL/exit_code_rejected`, exit `3`. |
| QA-RUN-06 | Security-sensitive runner behavior | Non-allowlisted environment values are not exposed and configured secrets do not persist. | **PASS** | Re-run `env` fixture (non-allowlisted `FIXTURE_SECRET`) → stdout `not-visible`. Re-run `secret` fixture → stderr redacted to `[REDACTED] diagnostic`. Post-run `grep -rl fixture-secret` over persisted evidence found no match (`NO SECRET PERSISTED`). |
| QA-RUN-07 | Evidence auditability | An attempted result has inspectable JSON and Markdown evidence with identity, reason, status, and references. | **PASS** | Re-run produced `artifacts/runs/<run-id>/{capability}.json|.md` under `scripts/fixtures/runner-project/artifacts/`; files were read after execution; `--json` envelope retained version, run_id, capability, command, exit, duration, parser, status, reason, and evidence references. |
| QA-RUN-08 | Intentional skip classification | A disabled capability returns `NOT_TESTED` with its skip reason. | **PASS** | Re-run `skip` capability → `NOT_TESTED/fixture intentional skip`. |
| QA-FSM-01 | FSM state graph / gate enforcement | `qa → archive` requires both reports; archive is rejected otherwise and state is preserved. | **PASS** | Scratch project with `workflow_fsm.enabled: true`, state at `qa` with `next: archive`: archive transition without reports → exit `1`, `accepted: false`, `reason: missing_verify_report`; with verify only → `reason: missing_qa_report`; `state.yaml` unchanged after both attempts. |
| QA-FSM-02 | FSM smoke suite | Parallel spec/design completion, idempotency, stale revision, concurrency, stale lock, atomic transitions. | **PASS** | Re-run `bash scripts/sdd-fsm-smoke.sh` → `fsm: parallel completion, idempotency, stale revision, concurrency, stale lock, and atomic transition checks passed`, exit `0`. |
| QA-FSM-03 | Legacy/resume persistence | Legacy four-field state remains resumable and its fields preserved. | **PASS** | `sdd-fsm-smoke.sh` and `sdd-smoke.sh` legacy fixture paths passed in this re-run; direct scratch projects used the four legacy fields without revision metadata and transitions validated them. |
| QA-FSM-04 | Fallback visibility | Disabled root runner/FSM behavior is visible and cannot be mistaken for deterministic enforcement. | **PASS** | Re-run against the change's project root: runner → `NOT_TESTED/runner_disabled`; FSM `inspect` → `status: fallback`, `reason: fsm_disabled`, exit `3`. |
| QA-QA-01 | Verify/QA adapter authority | Verify and QA preserve runner status, reason, evidence identity, and cannot turn failure/unavailable/blocking outcomes into prose-driven PASS. | **NOT TESTED** | No executable verify/QA adapter or product/operator target exists. `verify-runner-fail.md` and `qa-fallback.md` remain static fixtures; their text was not promoted to PASS. Requires an executable report-producing adapter or real target. |
| QA-QA-02 | Fallback/report acceptance | A real QA report records all scenario statuses and visible fallback limitations. | **BLOCKED** | The repository has no general report-generation runtime and the root manifest/FSM are intentionally disabled (external policy constraint), so report behavior could only be inspected as fixtures, which is technical evidence already owned by verify. |
| QA-DIST-01 | Source checkout distribution | The current checkout can run the local harness smoke entry points. | **PASS** | This re-run executed `sdd-quality-smoke.sh`, `sdd-fsm-smoke.sh`, and `sdd-smoke.sh` from the `agent-harness` checkout; all completed with exit `0` and their expected pass messages. |
| QA-DIST-02 | Dotfiles submodule distribution | The dotfiles submodule exposes the new scripts/config after rollout. | **BLOCKED** | Read-only inspection: `agent-harness` submodule contains the implementation (a6db498) but `/Users/acosta/Dev/dotfiles/editors/agents/opencode/scripts/sdd-quality-runner.mjs` and `sdd-fsm.mjs` are absent. Task `4.3` remains outside this checkout; no dotfiles changed. |
| QA-DIST-03 | Effective `~/.config/opencode` distribution | The materialized effective configuration exposes the new scripts/config. | **BLOCKED** | Read-only inspection: `sdd-quality-runner.mjs` and `openspec/quality-runner.json` absent from `/Users/acosta/.config/opencode`; Dotter not run by authorization constraint. |
| QA-TGT-01 | Product/operator acceptance boundary | The completed change is accepted on a real final product or deployed operator target. | **NOT TESTED** | No final application, external target, or supplied acceptance environment exists. Local fixtures establish only harness behavior and cannot substitute for product acceptance. |
| QA-WEB-01 | Browser/responsive/accessibility | Browser, responsive, and accessibility behavior is acceptable. | **NOT TESTED** | Not applicable: no browser/UI target exists. |
| QA-I18N-01 | Internationalization | Locale behavior is acceptable. | **NOT TESTED** | Not applicable: no locale-sensitive target exists. |
| QA-API-01 | API/authorization | API and unauthorized-user behavior is acceptable. | **NOT TESTED** | Not applicable: no API, identity, or authorization target exists. Secret isolation was covered separately by QA-RUN-06. |

## Untested Scope

| Scope | Reason | Re-run prerequisite |
|---|---|---|
| Verify/QA report-producing adapters and prose-authority boundary | Only static report fixtures and technical smoke assertions exist; no executable adapter path was supplied. | Provide or enable an executable verify/QA adapter and a target that emits/consumes runner envelopes. |
| Product/deployed operator acceptance | This repository is configuration/utilities only and no target was supplied. | Identify the final product/operator target, environment, and permissions. |
| Dotfiles submodule and effective `~/.config/opencode` rollout | Consumer pointer/materialization is stale; Dotter execution is not authorized in this phase. | Update the consumer pointer outside this change, materialize through the authorized rollout process, then rerun read-only distribution smoke. |
| Browser, accessibility, responsive, locale, API, and authorization behavior | No applicable surface exists. | Only rerun if a corresponding target is introduced. |
| Root deterministic mode | `openspec/config.yaml` intentionally sets runner/FSM `enabled: false`. | Enable explicitly for a controlled target and rerun without treating fallback as enforcement. |

## Findings

| ID | Severity | Scenario / location | Evidence | Status |
|---|---|---|---|---|
| QA-001 | **P1** | Distribution rollout, task `4.3` | New scripts/config are absent from the dotfiles submodule checkout and effective `/Users/acosta/.config/opencode`; only the source checkout was executable. Re-verified read-only on 2026-08-16: both files still absent. | **Open — external rollout prerequisite** |
| QA-002 | **P1** | Acceptance target boundary | No final product, deployed operator target, or supplied acceptance environment exists; several acceptance scenarios are therefore `NOT TESTED`. | **Open — target required before acceptance can close** |
| QA-003 | **P2** | Verify/QA adapters | Status-preservation and prose non-overwrite behavior has no independent executable report-generation target; static fixtures cannot be a QA PASS. | **Open — rerun with executable adapter/target** |

No `CRITICAL` or `P0` finding was observed in the executable local harness scenarios. The P1 findings are acceptance blockers, not claims that the locally executed runner/FSM behavior failed.

## Verdict

**BLOCKED**

### Rationale

The repository-local CLI behavior was observably consistent in this re-run for all selected harness scenarios: two configured stacks passed deterministically (`PASS/policy_satisfied`), non-pass outcomes remained `UNAVAILABLE`/`BLOCKED`/`FAIL`/`NOT_TESTED`, the configured secret was redacted in persisted evidence, FSM archive gates rejected missing reports while preserving state, and disabled-root mode reported visible `NOT_TESTED`/`fallback`. However, QA cannot close acceptance because the change has no final product/deployed operator target, verify/QA authority preservation was not independently executable, and the required dotfiles/effective-configuration rollout remains externally blocked (unchanged since 2026-08-09). The `allow_non_runtime_exception` policy is not applied: this change includes executable scripts and distribution behavior, so it is not documentation/config-only.

## Limitations and Implementation Handoff

- QA did not modify source code, fixtures, dotfiles, Dotter state, global configuration, or consumer pointers. The submodule worktree was clean after the run.
- Static inspection was used only to resolve capabilities, targets, and limitations; it did not produce any PASS result.
- Local smoke PASS results are harness-observable evidence only and must not be reported as product acceptance.
- `state.yaml` stays at `current_phase: qa` with `next: qa`; do not archive until the P1 acceptance blockers are resolved or a policy-allowed exception is explicitly established.
- Follow-up: provide the real acceptance target and authorized distribution rollout, then rerun this QA phase. Add an executable verify/QA adapter path if status-preservation behavior must be accepted end-to-end.