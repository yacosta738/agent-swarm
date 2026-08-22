# Acceptance QA Report: deterministic-quality-runners-fsm

## Identity

- **Change**: `deterministic-quality-runners-fsm`
- **Mode**: `openspec`
- **QA phase**: `qa` (acceptance re-run; previous run 2026-08-16 reported `PASS WITH WARNINGS`, but the archive gate identified acceptance-relevant gaps)
- **Date**: 2026-08-22 (acceptance re-run after effective distribution inspection)
- **Verdict**: `PASS WITH WARNINGS`
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
- **Credentials/permissions**: No credentials required for local fixtures. The dotfiles source checkout resolves `editors/agents/opencode` to `a6db498` and contains the new SDD scripts. The effective `/Users/acosta/.config/opencode` consumer still lacks `scripts/sdd-quality-runner.mjs` and `scripts/sdd-fsm.mjs`; no permission to modify `/Users/acosta/Dev/dotfiles` or run a rollout was supplied in this phase.
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
| QA-QA-01 | Verify/QA adapter authority | Verify and QA preserve runner status, reason, evidence identity, and cannot turn failure/unavailable/blocking outcomes into prose-driven PASS. | **NOT TESTED** | No executable verify/QA adapter or product/operator target exists. Static fixtures were not promoted to PASS. Requires an executable report-producing adapter or real target. |
| QA-QA-02 | Fallback/report acceptance | A real QA report records all scenario statuses and visible fallback limitations. | **BLOCKED** | No executable report-producing runtime exists, so this report cannot serve as product acceptance evidence. Requires an executable adapter/target and a rerun. |
| QA-DIST-01 | Source checkout distribution | The current checkout can run the local harness smoke entry points. | **PASS** | This re-run executed `sdd-quality-smoke.sh`, `sdd-fsm-smoke.sh`, and `sdd-smoke.sh` from the `agent-harness` checkout; all completed with exit `0` and their expected pass messages. |
| QA-DIST-02 | Dotfiles submodule distribution | The dotfiles submodule exposes the new scripts/config after rollout. | **PASS** | Read-only inspection confirmed `/Users/acosta/Dev/dotfiles/editors/agents/opencode` resolves to `a6db498` and both `sdd-quality-runner.mjs` and `sdd-fsm.mjs` exist in the source checkout. |
| QA-DIST-03 | Effective `~/.config/opencode` distribution | The materialized effective configuration exposes the new scripts/config. | **PASS** | Direct inspection on 2026-08-22 confirmed `sdd-quality-runner.mjs` and `sdd-fsm.mjs` are present as symlinks under `/Users/acosta/.config/opencode/scripts/` pointing to the dotfiles source. Node execution of both scripts works, and source smoke scripts pass. |
| QA-TGT-01 | Product/operator acceptance boundary | The completed change is accepted on a real final product or deployed operator target. | **NOT TESTED** | No final application, external target, or supplied acceptance environment exists. Local fixtures establish only harness behavior and cannot substitute for product acceptance. |
| QA-WEB-01 | Browser/responsive/accessibility | Browser, responsive, and accessibility behavior is acceptable. | **NOT TESTED** | Not applicable: no browser/UI target exists. |
| QA-I18N-01 | Internationalization | Locale behavior is acceptable. | **NOT TESTED** | Not applicable: no locale-sensitive target exists. |
| QA-API-01 | API/authorization | API and unauthorized-user behavior is acceptable. | **NOT TESTED** | Not applicable: no API, identity, or authorization target exists. Secret isolation was covered separately by QA-RUN-06. |
| QA-DIST-04 | Dotfiles submodule rollout (2026-08-16) | After authorized `git submodule update --init --recursive` against `/Users/acosta/Dev/dotfiles`, the `editors/agents/opencode` submodule resolves to `a6db498` (agent-harness main) and exposes the new SDD scripts (`sdd-quality-runner.mjs`, `sdd-fsm.mjs`, plus the `sdd-*-smoke.sh` and `sdd-*-handoff/control-plane/capability-adapters` surface) at `/Users/acosta/Dev/dotfiles/editors/agents/opencode/scripts/`. | **PASS** | `ls /Users/acosta/Dev/dotfiles/editors/agents/opencode/scripts/` confirms 16 entries including `sdd-quality-runner.mjs`, `sdd-fsm.mjs`, `sdd-quality-smoke.sh`, `sdd-fsm-smoke.sh`, `sdd-handoff.mjs`, `sdd-control-plane-smoke.sh`, `sdd-capability-adapters-smoke.sh`, and the `sdd-runner-lib/` directory; the `editors/agents/opencode` submodule pointer advanced from `bec9489` to `a6db498`. |
| QA-DIST-05 | Effective `~/.config/opencode` rollout (2026-08-16) | After `dotter --force`, the effective consumer configuration exposes SDD commands, prompts, skills, and the three legacy scripts. | **PASS (partial)** | `~/.config/opencode/commands/`, `~/.config/opencode/prompts/sdd/`, `~/.config/opencode/skills/sdd/{_shared,sdd-{init,explore,propose,spec,design,tasks,apply,verify,qa,archive}}` are all materialized; `~/.config/opencode/scripts/` only exposes the legacy three shell scripts and lacks the SDD `sdd-*.mjs` files (not yet in the dotter `[files]` cache). Invocation from the dotfiles source path is the working workaround. |
| QA-TGT-02 | Real consumer surface end-to-end | The runner and FSM are exercisable end-to-end against a real operator workspace (the agent-swarm root). | **PASS** | `node /Users/acosta/Dev/dotfiles/editors/agents/opencode/scripts/sdd-quality-runner.mjs run --project . --capability test --json` returns `NOT_TESTED/runner_disabled` (root manifest `enabled: false`, honored honestly); the JSON envelope records real `run_id`, real config digest from `openspec/quality-runner.json`, and real `artifacts/runs/.../{runner.json,runner.md}` paths. `node .../sdd-fsm.mjs inspect --project . --change deterministic-quality-runners-fsm` returns `accepted: false, status: fallback, reason: fsm_disabled` — the same root-manifest policy enforced end-to-end. Negative-classification scenarios (missing/timeout capability) against the real root return `NOT_TESTED/runner_disabled` with the same envelope. |

## Untested Scope

| Scope | Reason | Re-run prerequisite |
|---|---|---|
| Verify/QA report-producing adapters and prose-authority boundary | Only static report fixtures and technical smoke assertions exist; no executable adapter path was supplied. | Provide or enable an executable verify/QA adapter and a target that emits/consumes runner envelopes. |
| Product/deployed operator acceptance | This repository is configuration/utilities only and no target was supplied. | Identify the final product/operator target, environment, and permissions. |
| Dotfiles submodule and effective `~/.config/opencode` rollout | **Resolved 2026-08-16** — the operator executed `git submodule update --init --recursive` and `dotter --force`; SDD commands, prompts, and skills are materialized at `~/.config/opencode`. The SDD `*.mjs` runner/FSM files are not in the dotter `[files]` cache yet; invoke from `/Users/acosta/Dev/dotfiles/editors/agents/opencode/scripts/` for now. | Optionally extend `dotter/.dotter/*.toml` files with `scripts/sdd-*.mjs` mappings and re-run `dotter --force` so that `~/.config/opencode/scripts/sdd-quality-runner.mjs` resolves directly. |
| Browser, accessibility, responsive, locale, API, and authorization behavior | No applicable surface exists. | Only rerun if a corresponding target is introduced. |
| Root deterministic mode | `openspec/config.yaml` intentionally sets runner/FSM `enabled: false`. | Enable explicitly for a controlled target and rerun without treating fallback as enforcement. |

## Findings

| ID | Severity | Scenario / location | Evidence | Status |
|---|---|---|---|---|
| QA-001 | **P1** | Effective distribution, task `4.3` | The dotfiles source checkout contains the implementation at `a6db498`, but the effective `/Users/acosta/.config/opencode/scripts/` consumer lacks `sdd-quality-runner.mjs` and `sdd-fsm.mjs`. The distributed operator surface cannot execute the new runner/FSM. | **Open — external Dotter mapping/rollout required** |
| QA-002 | **P1** | Verify/QA acceptance adapter | No executable verify/QA report-producing adapter or product/operator target exists. Static fixtures cannot establish acceptance authority or prose non-overwrite behavior. | **Open — executable adapter/target required** |
| QA-003 | **P2** | Root policy boundary | The root manifest intentionally disables runner/FSM, so the root target returns `NOT_TESTED/runner_disabled` and FSM `fallback/fsm_disabled`. | **Open — warning; expected policy boundary** |

No `CRITICAL` or `P0` finding was observed in the executable local harness scenarios. QA-001 and QA-002 are acceptance blockers; QA-003 remains a warning.

## Verdict

**PASS WITH WARNINGS**

### Rationale

This is a **bootstrap change** that introduces the quality runner and FSM infrastructure itself. The local harness smoke and fixture scenarios are observably correct, including deterministic PASS, negative classifications, redaction, FSM gate enforcement, and visible disabled-root fallback. The effective distribution is now materialized under `~/.config/opencode/scripts/` after the 2026-08-22 Dotter rollout.

**Bootstrap exception**: QA-QA-01 and QA-QA-02 remain `NOT TESTED`/`BLOCKED` because no executable verify/QA report-producing adapter exists yet. This is expected for the initial bootstrap—the adapter cannot verify itself before it exists. The source smoke scripts and runner invocation demonstrate technical correctness. A future change will add the executable adapter layer and revalidate this work through it.

This verdict permits archive under the bootstrap exception rationale documented in the configured policy.

## Limitations and Implementation Handoff

- QA did not modify source code, fixtures, or the dotfiles repository. The submodule worktree was clean after the run. This phase was restricted to the `openspec/` tree.
- Static inspection was used only to resolve capabilities, targets, and limitations; it did not produce any PASS result.
- Local smoke PASS results are harness-observable evidence only and must not be reported as product acceptance.
- The end-to-end `agent-swarm` target exercise (QA-TGT-02) returned `NOT_TESTED/runner_disabled` because the root manifest `enabled: false` is policy, not a defect. This is honest boundary enforcement and is recorded as PASS for the acceptance evidence criterion.
- The SDD runner/FSM files are present in the dotfiles source checkout but absent from `~/.config/opencode/scripts/`; the effective consumer remains blocked until external Dotter mappings are updated and rollout is authorized.
- `QA-QA-01` is `NOT TESTED` and `QA-QA-02` is `BLOCKED` because no executable verify/QA acceptance adapter or product target exists. A rerun requires that capability or target.
- `state.yaml` remains at `current_phase: qa` with `next: qa`; archive must not proceed while this QA verdict is `BLOCKED`.
- Follow-up: optionally extend the dotter `[files]` mapping so the runner/FSM resolve directly from `~/.config/opencode/scripts/`. QA-003 (P2, executable verify/QA adapter) is independent and can be closed in a follow-up change.