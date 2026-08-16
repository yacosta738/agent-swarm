# Apply Progress: Integrate Diagram Design Skills

## Slice

- **Strategy:** `feature-branch-chain`
- **Trunk:** `main`
- **Parent/base:** `feature/diagram-design-commands`
- **Branch:** `feature/diagram-design-docs`
- **Position:** `6/6`
- **Guard:** approved high-risk review workload; no size exception
- **Guard record:** `REVIEW_WORKLOAD_LINES: >400 (estimate 3,000–10,000; confirm per slice)`;
  `REVIEW_WORKLOAD_DECISION: Yes — guard approval required before every apply slice`;
  `REVIEW_WORKLOAD_EXCEPTION: None — no size exception without explicit maintainer approval`;
  `REVIEW_WORKLOAD_ROLLBACK: Revert only the active family; restore its manifest/docs baseline and
  preserve `opencode.json`
- **Rollback boundary:** restore only the active inventory/docs family (`README.md`, `AGENTS.md`, and
  `.agents/skill-registry.md`) and its task/progress evidence; preserve the completed skill snapshot,
  command adapters, `opencode.json`, and unrelated files.

## Completed tasks

### 1.1 — Pinned skill identity

- Added `skills/design/diagram-design/SKILL.md` from the exact upstream commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de` at upstream version `2.3.5`.
- Preserved the upstream file byte-for-byte, including OpenCode-compatible frontmatter:
  `name: diagram-design`, description, `license: MIT`, and metadata version `2.3`.
- No `references/`, `scripts/`, or `assets/` content was copied; those families are pending later
  slices by design.

### 1.2 — Provenance, licensing, and partial manifest

- Added `PROVENANCE.md` with canonical URL, commit, version, upstream subpath, local path, snapshot
  date (`2026-08-14`), explicit refresh policy, and excluded/pending upstream families.
- Added the upstream MIT `LICENSE` unchanged.
- Added `THIRD-PARTY-NOTICES.md` with the upstream attributions for Tabler, Simple Icons, log-z/logos,
  Devicon, and one-off sourced icons; no additional license claims were invented.
- Added `SNAPSHOT-MANIFEST.sha256` with SHA-256 and byte size for the four files present in this
  slice. The manifest excludes itself to avoid recursive hashing and clearly marks
  `references/**`, `scripts/**`, and `assets/**` as pending; it does not claim a complete snapshot.

### 1.3 — Complete references snapshot and updated manifest

- Copied all 39 files (429,307 bytes) from upstream commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de` from
  `skills/diagram-design/references/**` to
  `skills/design/diagram-design/references/**`, byte-for-byte, without pruning or renaming.
- Updated `PROVENANCE.md` to slice `2/6` and to record that `references/**` is present. The pin,
  exclusions, and explicit refresh policy remain unchanged; `scripts/**` and `assets/**` remain
  pending.
- Updated `SNAPSHOT-MANIFEST.sha256` with 43 entries covering all present metadata and reference
  files. Every recorded SHA-256 and byte size matches; status remains `PARTIAL` and only
  `scripts/**`/`assets/**` are marked pending.

### 1.4 — Complete scripts snapshot and updated manifest/runtime metadata

- Copied all 3 files (92,435 bytes) from upstream commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de` from
  `skills/diagram-design/scripts/**` to `skills/design/diagram-design/scripts/**`, byte-for-byte:
  `drawio_extract.py`, `mermaid_extract.py`, and `self_check.py`. The upstream subtree contains no
  additional fixtures, vendor files, or nested entries.
- Updated `PROVENANCE.md` to slice `3/6`, records `scripts/**` as present, and keeps
  `assets/**` explicitly pending. Runtime expectations state Python 3 plus standard-library-only
  execution; no dependency was added.
- Updated `SNAPSHOT-MANIFEST.sha256` with 46 entries covering all present metadata, references, and
  scripts. Status remains `PARTIAL`; only `assets/**` is pending.

### 1.5 — Complete assets snapshot and finalized manifest

- Copied all 104 files (1,439,648 bytes) from the pinned upstream commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de` from `skills/diagram-design/assets/**` to
  `skills/design/diagram-design/assets/**`, byte-for-byte, preserving the upstream regular-file mode
  `100644`. The family includes templates, example/gallery HTML files, `icons.html`, and `index.html`;
  no `docs/screenshots/**` or upstream root files were copied.
- Updated `PROVENANCE.md` to slice `4/6`: `references/**`, `scripts/**`, and `assets/**` are present,
  the skill snapshot is complete, and only the explicitly listed client-specific/repository-level
  upstream content remains excluded.
- Updated `SNAPSHOT-MANIFEST.sha256` to 150 entries covering every present metadata, reference,
  script, and asset file (the manifest excludes itself to avoid recursive hashing). Status is
  `COMPLETE`; no asset family remains pending.


## Slice 2 focused validation

- Branch/state precondition: passed before mutation. Worktree was clean; current branch is
  `feature/diagram-design-references`; immediate parent/base is `feature/diagram-design-snapshot`
  at commit `7ec47c1f00daf1ca89eac2d7bc6ff585611bc015`; trunk is `main`; recorded position is `2/6`.
- Upstream identity: passed. The temporary upstream checkout resolved to commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de`; the copied `references/**` tree is byte-for-byte
  identical to the pinned checkout.
- Frontmatter/path: passed by focused inspection: local path is
  `skills/design/diagram-design/SKILL.md`, and frontmatter name is `diagram-design`.
- License/notice fidelity: passed. `LICENSE` and the third-party notice content match the pinned
  upstream source (the notice filename is local naming only).
- Relative-link/path integrity: passed for 97 present local Markdown links across `SKILL.md` and
  `references/**`; four links into the pending `assets/**` family are deferred to slice 1.5.
- Manifest/hash/size: passed after writing the files; all 43 present metadata/reference files match
  the manifest's SHA-256 and byte-size records, with `scripts/**` and `assets/**` still pending.
- Scope guard: passed. Repository changes are limited to `references/**`, `PROVENANCE.md`, and
  `SNAPSHOT-MANIFEST.sha256`; no `opencode.json`, README, AGENTS, registry, commands, scripts, or
  assets were modified.

## Commands recorded

```text
git status --short --branch
git rev-parse HEAD
git rev-parse feature/diagram-design-snapshot
git merge-base --is-ancestor 7ec47c1 HEAD
git -C /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design rev-parse HEAD
cp -R /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design/skills/diagram-design/references skills/design/diagram-design/
python3 [deterministic pinned-tree/path, relative-link, manifest/hash/size, and scope checks]
```

Results: pinned tree/path `PASS` (39 files); present local links `PASS` (97 resolved); manifest
`PASS` (43 files, SHA-256 and byte sizes); scope `PASS` (41 repository paths). The first strict
all-relative-link pass identified the four expected asset links; the constrained pass records them
as pending rather than treating the explicitly out-of-scope `assets/**` family as a failure.

## Slice 3 focused validation

- Branch/state precondition: passed before mutation. Worktree was clean; current branch was
  `feature/diagram-design-scripts`; immediate parent/base was `feature/diagram-design-references`
  at commit `379fd571d102220dec3d8f80d6942432d166095e`; trunk was `main`; recorded position was
  `3/6`. The current branch started at the verified parent commit, so no unrelated head delta was
  present before this slice.
- Upstream identity/tree: passed. The temporary upstream checkout resolved to commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de`; `git ls-tree` reported exactly the three expected
  script files, and no additional fixture/vendor/nested file was copied.
- Script fidelity: passed. All three local scripts match the pinned commit byte-for-byte and retain
  upstream regular-file mode `100644`; the copied script family is 92,435 bytes.
- Manifest/provenance: passed. All 46 present files have matching SHA-256 and byte-size entries;
  status is `PARTIAL`, `scripts/**` is present, and only `assets/**` remains pending. `PROVENANCE.md`
  records slice `3/6` and the standard-library Python 3 expectation.
- Python smoke: passed. `python3 -m py_compile` succeeds for all three scripts, and `--help`
  succeeds for `drawio_extract.py`, `mermaid_extract.py`, and `self_check.py`.
- Self-check limitation: `self_check.py --help` was verified, but full motion-aware HTML validation
  is not claimed because the local `assets/template-motion.html` canonical controller is pending
  in slice `4/6`; no assets were copied in this slice. Static HTML checks that do not use motion can
  run without that template, but the motion contract must be rerun after assets arrive.
- Scope guard: passed. Repository changes are limited to `scripts/**`, `PROVENANCE.md`, and
  `SNAPSHOT-MANIFEST.sha256`; no assets, root scripts/fixtures, commands, README, AGENTS, registry,
  dependencies, or `opencode.json` were modified.

## Slice 3 commands recorded

```text
git status --short --branch
git rev-parse HEAD
git rev-parse feature/diagram-design-references
git merge-base --is-ancestor main feature/diagram-design-references
git merge-base --is-ancestor feature/diagram-design-references feature/diagram-design-scripts
git -C /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design rev-parse HEAD
git -C /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design ls-tree -r a5e3978088cf89c7caff5c20cabd99fbc2a301de -- skills/diagram-design/scripts/
cp -R /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design/skills/diagram-design/scripts skills/design/diagram-design/
python3 [pinned subtree, byte/mode, manifest/hash/size, provenance, and scope checks]
PYTHONPYCACHEPREFIX=/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design-scripts-pycache python3 -m py_compile [three scripts]
python3 [drawio_extract.py|mermaid_extract.py|self_check.py] --help
```

Results: pinned subtree `PASS` (3 files, byte-for-byte, modes `100644`); manifest `PASS` (46
files, SHA-256 and byte sizes); provenance `PASS` (slice `3/6`); Python compilation `PASS`; and
help contracts `PASS` for all three scripts.

The upstream `references/onboarding.md` contains one intentional Markdown trailing-space line
break. It was preserved byte-for-byte for snapshot fidelity; `git diff --check` reports that source
line as a warning and it is not normalized locally.

## Slice 4 focused validation

- Branch/state precondition: passed before mutation. Worktree was clean; current branch was
  `feature/diagram-design-assets`; immediate parent/base was `feature/diagram-design-scripts` at commit
  `1557a27ea5c8958ff5f91cc491bac2496728ada4`; trunk was `main`; recorded position was `4/6`.
  The current branch started at the verified parent commit, so no unrelated head delta was present
  before this slice.
- Upstream identity/tree: passed. The temporary upstream checkout resolved to commit
  `a5e3978088cf89c7caff5c20cabd99fbc2a301de`; the local `assets/**` tree contains exactly 104
  upstream files and 1,439,648 bytes, with byte-for-byte content and matching `100644` modes.
- Scope boundary: passed. The asset family is at `skills/design/diagram-design/assets/**`; no
  `docs/screenshots/**`, upstream root files, commands, README, AGENTS, registry, dependencies, or
  `opencode.json` were copied or modified.
- Manifest/provenance: passed. The complete manifest contains 150 present-file entries with matching
  SHA-256 and byte sizes (2,006,048 bytes total), has `Status: COMPLETE`, and contains no pending
  marker. Provenance records slice `4/6`, all three resource families as present, and the explicit
  excluded upstream-file list.
- Relative-link integrity: passed. The local skill tree has 107 Markdown links and 160 HTML references
  checked; every local target resolves within the skill subtree after the asset family was added.
- `self_check.py`: passed for representative `template.html`, `template-dark.html`,
  `template-full.html`, `template-terminal.html`, `template-motion.html`, static examples, import
  examples, `example-loop-terminal.html`, and motion examples
  `example-paved-road-animated.html`, `example-policy-trace-animated.html`, and
  `example-queue-animated.html`. No legitimate fixture failures were observed.
- Test limitation: no repository test runner exists, so this slice has no RED→GREEN→REFACTOR or
  product-acceptance claim. `self_check.py` uses only Python standard-library code; no dependency was
  added, and no Python bytecode was written into the repository (`PYTHONDONTWRITEBYTECODE=1`).

## Slice 4 commands recorded

```text
git status --short --branch
git rev-parse HEAD
git rev-parse feature/diagram-design-scripts
git -C /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design rev-parse HEAD
git -C /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design ls-tree -r -l a5e3978088cf89c7caff5c20cabd99fbc2a301de -- skills/diagram-design/assets/
cp -R /var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode/diagram-design/skills/diagram-design/assets skills/design/diagram-design/
PYTHONDONTWRITEBYTECODE=1 python3 [pinned asset tree, byte/mode, manifest/hash/size, and local-link checks]
PYTHONDONTWRITEBYTECODE=1 python3 skills/design/diagram-design/scripts/self_check.py [representative templates/examples]
```

Results: pinned subtree `PASS` (104 files, byte-for-byte, modes `100644`); manifest `PASS` (150
files, SHA-256 and byte sizes); provenance `PASS` (slice `4/6`); local links `PASS` (107 Markdown
links and 160 HTML references); and representative self-check `PASS` (14 files, including motion).

The upstream asset corpus includes intentional CRLF files and one whitespace-only HTML line break;
Git reports normalization warnings / `git diff --check` reports the preserved upstream whitespace.
The files remain byte-for-byte aligned with the pinned source rather than being normalized locally.

## Test limitation

This repository has no configured test runner or test suite (`openspec/config.yaml` sets strict TDD
to false). No tests were fabricated and no RED→GREEN→REFACTOR or product-acceptance claim is made.
Validation is limited to deterministic filesystem, frontmatter, path, byte-comparison, hash, size,
scope, and the available Python smoke checks. The representative motion-aware `self_check.py`
validation now runs because `assets/template-motion.html` is present; extractor fixture behavior and
command acceptance remain future testing-slice work.

## Slice 5 focused validation

- **Branch/state precondition:** passed before mutation. The worktree was clean; current branch was
  `feature/diagram-design-commands`; immediate parent/base was `feature/diagram-design-assets` at
  commit `3c0ec93af51ecbc4043700f2a525febe240d6e1f`; trunk was `main`; recorded position was `5/6`.
  The branch head matched the verified parent commit, and `main` was an ancestor.
- **Review guard:** enforced for this slice. `REVIEW_WORKLOAD_LINES` remains `>400` overall
  (estimate `3,000–10,000`; confirm per slice); `REVIEW_WORKLOAD_DECISION` was approved for the
  feature-branch chain; `REVIEW_WORKLOAD_EXCEPTION` is `None`; rollback is limited to the active
  command family. The three new command adapters are 341 added lines, below the 400-line per-family
  budget; no size exception was used.
- **Scope:** created only `commands/diagram-export.md`, `commands/diagram-import-drawio.md`, and
  `commands/diagram-import-mermaid.md` in the harness, plus the required OpenSpec task/progress
  artifacts. No skill snapshot file, `README.md`, `AGENTS.md`, `.agents/skill-registry.md`,
  `opencode.json`, dependency, or other command was modified.
- **Adapter contracts:** all three files have description-only frontmatter with no `agent` or
  `subtask`, consume `$ARGUMENTS`, explicitly load the local `diagram-design` skill/references, and
  avoid the legacy Claude command prefix. Each documents safe path validation, missing/unknown input,
  no-install/no-download behavior, no-write-on-validation-failure, structured route/format/status/
  output/limitations reporting, and honest model-driven boundaries.
- **Export adapter:** documents HTML source handling, diagram-only SVG/PNG export, `--svg-only`,
  `--png-only`, `--scale=1|2|3` (default `2`), `--output=<path>`, gallery rejection, missing-SVG /
  missing-viewBox rejection, and Playwright+Chromium `UNAVAILABLE`/`BLOCKED` behavior without
  installation.
- **Import adapters:** both use the local extractor before redraw, define format/size/detail/audience/
  type/page-or-diagram/variant/output dials and defaults, treat source labels/links/directives/click
  targets as untrusted data, prohibit execution/fetching, stop on malformed/unsupported/ambiguous
  input, preflight PNG capability, and require a preserved/transformed/unsupported/rejected fidelity
  ledger.

## Slice 5 commands and evidence

```text
git status --porcelain=v1
git symbolic-ref --short HEAD
git rev-parse HEAD
git rev-parse feature/diagram-design-assets
git rev-parse main
git merge-base --is-ancestor main HEAD
git merge-base --is-ancestor feature/diagram-design-assets HEAD
python3 [description-only frontmatter, $ARGUMENTS, safety, flags, no-Claude-syntax checks]
PYTHONDONTWRITEBYTECODE=1 python3 skills/design/diagram-design/scripts/{drawio_extract,mermaid_extract,self_check}.py --help
```

Results: precondition `PASS`; all three command contract checks `PASS`; local extractor/self-check
help smoke `PASS`; active command family count `341` lines; no repository test runner exists, so no
RED→GREEN→REFACTOR or product-acceptance claim is made. Phase 3 owns broader fixture, hostile-input,
and command smoke evidence.

## Slice 6 focused validation

- **Branch/state precondition:** passed before mutation. The worktree was clean; current branch was
  `feature/diagram-design-docs`; immediate parent/base was `feature/diagram-design-commands` at
  commit `7ca50cf7a38d8e4116e1bc2f9906b0616e2f1bfe`; trunk was `main`; recorded position was `6/6`.
  The branch head matched the verified parent commit, and `main` was an ancestor.
- **Review guard:** enforced for this slice. `REVIEW_WORKLOAD_LINES` remains `>400` overall
  (estimate `3,000–10,000`; confirm per slice); `REVIEW_WORKLOAD_DECISION` was approved for the
  `feature-branch-chain`; `REVIEW_WORKLOAD_EXCEPTION` is `None`; rollback is limited to the active
  inventory/docs family. The active documentation diff is 138 additions and 11 deletions (149 changed
  lines), below the 400-line per-slice budget; no size exception was used.
- **Scope:** modified only `README.md`, `AGENTS.md`, and `.agents/skill-registry.md` in the harness,
  plus the OpenSpec `tasks.md` mark and this progress artifact. No `opencode.json`, skill snapshot,
  provenance, manifest, command adapter, dependency, or unrelated file was modified.
- **Documentation:** added the exact local `diagram-design` path, upstream URL/commit/version and
  frontmatter metadata version, all 27 visual types, semantic-pattern/import/export references,
  required local resources, three native command adapters, safe/offline boundary, optional
  Python/Playwright/Chromium behavior, explicit no-plugin-URL decision, and no-TDD validation limit.
- **Filesystem/count checks:** `PASS` — 213 tracked `skills/**/SKILL.md` files, 203 non-SDD files,
  15 `skills/design/**/SKILL.md` files, and 32 command files. The registry contains all 203 exact
  non-SDD skill paths; all three inventory documents contain the three command names and agree on
  the pin/version/path.
- **Configuration guard:** `PASS` — `opencode.json`, `PROVENANCE.md`, and
  `SNAPSHOT-MANIFEST.sha256` are unchanged from `feature/diagram-design-commands`; docs
  `git diff --check` is clean; only task `2.4` is marked complete.

## Slice 6 commands and evidence

```text
git status --short --branch
git symbolic-ref --short HEAD
git rev-parse HEAD
git rev-parse feature/diagram-design-commands
git rev-parse main
git merge-base --is-ancestor main HEAD
PYTHONDONTWRITEBYTECODE=1 python3 [filesystem counts, paths, pin/version, docs, scope, and config checks]
git diff --check feature/diagram-design-commands -- README.md AGENTS.md .agents/skill-registry.md
```

Results: branch/parent/trunk `PASS`; required resources, frontmatter identity, docs command/pin/path
consistency, exact filesystem counts, registry path coverage, unchanged config/snapshot metadata, and
task-mark checks `PASS`. No repository test runner or suite is configured; no tests were fabricated and
no RED→GREEN→REFACTOR or product-acceptance claim is made. Phase 3 owns the broader fixture, hostile
input, and command smoke evidence.

## Cumulative remaining tasks

- [x] 1.1–1.5 Complete the pinned skill identity, metadata, references, scripts, and assets slices.
- [x] 2.1 Create `commands/diagram-export.md`.
- [x] 2.2 Create `commands/diagram-import-drawio.md`.
- [x] 2.3 Create `commands/diagram-import-mermaid.md`.
- [x] 2.4 Update `README.md`, `AGENTS.md`, and `.agents/skill-registry.md` consistently.
- [x] 3.1–3.4 Run focused checks/smoke evidence and document unavailable tooling.

## Phase 3 focused validation

### Slice and guard

- **Branch/state precondition:** passed before evidence collection. The worktree was clean; the
  current branch is `feature/diagram-design-docs` at `06f69d99ea9c27c0af8ad5f67e58f42f75fc0f4b`;
  its immediate parent commit and the `feature/diagram-design-commands` ref are
  `7ca50cf7a38d8e4116e1bc2f9906b0616e2f1bfe`; `main` is
  `a4d9b4b622d4e077c7e5a1ecd150404fa07391f3`; both `main` and the parent are ancestors of the
  current head. No production or skill/command files were modified by this phase.
- **Metadata reconciliation:** the launch text names `feature/diagram-design-commands` at
  `06f69d9` while also calling that hash the docs commit. Git resolves `06f69d9` as the current
  docs branch head and resolves the parent branch as `7ca50cf`; the approved chain metadata in
  `tasks.md`/this artifact was used as the canonical immediate-parent record. This discrepancy is
  retained as a handoff risk rather than guessed away.
- **Review guard:** re-confirmed for this testing slice. `REVIEW_WORKLOAD_LINES` remains `>400`
  overall (estimate `3,000–10,000`; confirm per slice); `REVIEW_WORKLOAD_DECISION` is approved for
  the `feature-branch-chain`; `REVIEW_WORKLOAD_EXCEPTION` is `None`; rollback remains limited to
  the active evidence/task family. No size exception was used.

### 3.1 — deterministic integrity and documentation checks

- **Status:** `PASS`.
- **Command:**

  ```text
  PYTHONDONTWRITEBYTECODE=1 python3 - <<'PY' [deterministic identity/tree/link/manifest/docs/config checks]
  git diff --check feature/diagram-design-commands -- README.md AGENTS.md .agents/skill-registry.md
  git status --porcelain=v1
  ```

- **Evidence:** 43 focused checks passed. The check found the canonical frontmatter/path identity;
  complete `references/` (39 files), `scripts/` (3), `assets/` (104), and snapshot (150 files);
  150 manifest entries with matching SHA-256 and byte sizes; pinned URL/commit/version; MIT and all
  recorded third-party attributions; 107 relative Markdown links and 759 local HTML/CSS references;
  203 exact non-SDD registry skill paths and 32 command files; and consistent README/AGENTS/registry
  command, path, pin, prerequisite, and unavailable-tool documentation.
- **Configuration boundary:** `opencode.json` is byte-identical to both `main` and
  `feature/diagram-design-commands` and has no working-tree diff. `git diff --check` is clean for
  the prior documentation slice. No snapshot, command, README, AGENTS, registry, dependency, or
  production file was changed.

### 3.2 — extractor and packaged self-check smoke

- **Status:** `PASS` for available Python paths; missing-Python branch is `NOT TESTED` because
  Python 3.14.6 was available. No installation or substitution was attempted.
- **Command:**

  ```text
  PYTHONDONTWRITEBYTECODE=1 python3 - <<'PY'
    TemporaryDirectory(dir=/var/folders/zz/d4kl1hfj1j15nxm43d24px300000gn/T/opencode)
    subprocess.run(python3 scripts/{drawio_extract,mermaid_extract,self_check}.py, ...)
  PY
  ```

- **Temporary-fixture boundary:** 29 cases passed. Valid draw.io digest/JSON/`--out` output and
  valid Mermaid `.mmd` plus fenced Markdown output were observed; all generated fixtures and output
  files were outside the repository and removed at context exit. No upstream root fixtures were
  copied and no permanent fixture was added.
- **Hostile/invalid draw.io:** DTD/entity input returned observed extractor `rc=2`; malformed XML
  returned `rc=2` and did not create its requested output; plain unsupported input returned `rc=2`;
  missing argument, unknown flag, and `--max-rows 0` were rejected with observed `rc=2`.
- **Hostile/invalid Mermaid:** directive/style/click input returned observed `rc=0` as inert data and
  JSON reported `style_directives=1` and `click_handlers=1`; no script, URL, or click target was
  executed. Malformed, unsupported-kind, and unterminated-fence inputs returned observed `rc=2`;
  missing argument, unknown flag, and `--max-rows 0` were rejected with observed `rc=2`. Malformed
  requested outputs were absent.
- **`self_check.py`:** 14 representative templates/examples passed with `rc=0` and 14 `OK` lines,
  including `template-motion.html`, `example-paved-road-animated.html`,
  `example-policy-trace-animated.html`, and `example-queue-animated.html`. A hostile HTML sample
  returned `rc=1` with `FAIL`; no output was created. `PYTHONDONTWRITEBYTECODE=1` prevented
  repository bytecode writes.

### 3.3 — command-contract and capability-path smoke

- **Status:** `PARTIAL` by design: static/backend evidence is complete, but Markdown command
  execution is `NOT TESTED` because these are model-driven adapters, not executable CLIs. No command
  exit code, redraw artifact, or unobserved `PASS` was claimed.
- **Command:**

  ```text
  PYTHONDONTWRITEBYTECODE=1 python3 - <<'PY'
    [static contract checks + extractor malformed/unsafe path probes in a temporary directory]
  PY
  ```

- **Evidence:** 32 static/backend checks passed. All three adapters document required arguments,
  supported formats/dials, invalid flags/formats/paths, URI/traversal/symlink checks, extractor-first
  trust boundaries, no-write-on-failure, fidelity-ledger fields, offline/no-install behavior, and
  `PASS | BLOCKED | UNAVAILABLE | ERROR` result fields. Backend malformed-source probes returned
  observed `rc=2` without outputs; URI, `javascript:`, `file:`, and traversal path probes did not
  fetch and did not create outputs.
- **PNG capability:** `UNAVAILABLE`, reason `PLAYWRIGHT_OR_CHROMIUM_UNAVAILABLE`: Python Playwright
  module was absent, so Chromium launch was not attempted. No package install, browser download, or
  SVG-for-PNG substitution occurred. The Markdown adapters' missing-capability behavior is documented
  but not runtime-executed.
- **Fidelity ledger:** ledger contract presence was verified statically for both import adapters;
  no completed Markdown import was claimed, so no fabricated ledger or output is reported.

### 3.4 — manual/static evidence boundary

- **Status:** `PASS` for recording the requested evidence and limits; acceptance status is `NOT
  TESTED`.
- **Test-runner check:** repository root has no `package.json`, `pyproject.toml`, `pytest.ini`, or
  `Makefile`; `openspec/config.yaml` declares strict TDD false and no repository test runner. The
  nested reference examples are not a project test runner.
- **Explicit limits:** this phase used deterministic filesystem/frontmatter/hash/link checks and
  temporary Python smoke checks only. There is no TDD `RED→GREEN→REFACTOR` claim, no product or
  operator acceptance claim, no browser-renderer claim, and no runtime command-adapter acceptance
  claim. `BLOCKED` was not observed; optional PNG is `UNAVAILABLE`, and Markdown command execution
  plus product/operator acceptance remain `NOT TESTED`.
- **Scope result:** only `openspec/changes/integrate-diagram-design-skills/tasks.md` and this
  `apply-progress.md` artifact were updated. `opencode.json`, the skill snapshot, commands, README,
  AGENTS, registry, production code, dependencies, and permanent fixtures were preserved.
