## Exploration: integrate-diagram-design-skills

### Current State

The target is an OpenCode configuration harness whose reusable instructions are discovered from
`skills/**/SKILL.md` when the checkout is installed as the global OpenCode configuration. The
harness already groups skills by ecosystem, keeps progressive-disclosure material beside each
skill, and documents the source tree in `README.md`, `AGENTS.md`, and `.agents/skill-registry.md`.
The existing `opencode.json` also contains a git-based plugin entry for `openpencil-skill`, but
that is a repository-specific OpenCode plugin contract, not evidence that arbitrary GitHub skill
repositories can be loaded as OpenCode plugins.

There is no active application package or root test suite. Validation for this configuration-only
change must therefore use deterministic filesystem/frontmatter checks and the upstream skill's
available Python smoke checks; it MUST NOT be described as product acceptance or strict TDD.

### Affected Areas

- `skills/design/diagram-design/` — vendored, version-pinned Diagram Design skill and its relative
  `references/`, `scripts/`, and `assets/` dependencies.
- `commands/diagram-*.md` — optional OpenCode-native command adapters for export and draw.io/Mermaid
  import; upstream commands use Claude-specific syntax and are not directly discovered by OpenCode.
- `README.md` — document the new skill, supported workflows, provenance, and local validation.
- `AGENTS.md` and `.agents/skill-registry.md` — update the generated/maintained skill inventory and
  routing guidance so the skill is visible to future agents.
- `openspec/changes/integrate-diagram-design-skills/` — durable SDD artifacts; the OpenSpec root is
  the parent workspace `/Users/acosta/Dev/agent-swarm/openspec`, not this repository directory.

### Evidence

#### Harness

- `README.md` states that skills are auto-discovered from `skills/**/SKILL.md` and that references
  live beside the skills.
- Current skills use nested ecosystem paths such as `skills/design/impeccable/` and preserve large
  progressive-disclosure reference trees and scripts.
- Current OpenCode documentation confirms that installed global skills are loaded from
  `~/.config/opencode/skills/<name>/SKILL.md`; this repository's `skills/` directory is the source
  material for that installed path. The skill frontmatter `name` must match its containing skill
  directory and use the lowercase-hyphen naming grammar.
- `opencode.json` already has
  `openpencil-skill@git+https://github.com/zseven-w/openpencil-skill.git`, but that upstream repo
  has a `package.json`/OpenCode installer contract and explicitly registers its skills. The target
  repository has no `package.json` or `.opencode/INSTALL.md`, so adding its URL to the plugin array
  would not be a verified integration path.

#### Upstream repository

The inspected `main` tree resolves to commit
`a5e3978088cf89c7caff5c20cabd99fbc2a301de` and advertises version `2.3.5` in
`.codex-plugin/plugin.json`; the skill frontmatter reports `2.3`. It is MIT licensed and declares
that its shared skill lives at `skills/diagram-design/`.

The skill is not just one prompt: `SKILL.md` references a substantial relative tree of type,
semantic-pattern, animation, onboarding, import, output, export, and primitive references. It also
ships `drawio_extract.py`, `mermaid_extract.py`, `self_check.py`, HTML templates/examples/gallery,
and icon assets. Copying only `SKILL.md` would leave broken relative references and would not deliver
the requested capability. The repository's Claude/Codex marketplace manifests and Claude/Pi
commands are client-specific metadata; OpenCode should consume the Agent Skills-compatible skill
tree and receive separately adapted native commands.

### Approaches

1. **Vendor the complete skill snapshot (recommended)** — copy the upstream
   `skills/diagram-design/` tree into `skills/design/diagram-design/`, retain the relative
   references/scripts/assets, add provenance and applicable license notices, and pin the source
   commit/version in the change documentation.
   - Pros: native discovery after harness deployment; works offline; preserves progressive
     disclosure and the packaged self-check; no undocumented OpenCode API; deterministic rollback.
   - Cons: large static snapshot; upstream updates require an intentional refresh and license
     review; HTML examples/icons increase repository size.
   - Effort: Medium/High

2. **Add the GitHub URL to `opencode.json` as a plugin** — treat the upstream repository like the
   existing OpenPencil entry.
   - Pros: small config diff and potentially automatic updates.
   - Cons: the upstream repository is not an OpenCode plugin package; it has no verified OpenCode
     loader contract, and the external `.codex-plugin`/`.claude-plugin` manifests are not OpenCode
     manifests. This risks a silent non-discovery or startup failure.
   - Effort: Low to write, High risk

3. **Submodule/subtree or runtime fetch** — reference the external repository instead of storing a
   snapshot.
   - Pros: smaller primary diff and easier upstream refresh in theory.
   - Cons: submodule initialization and dotfiles distribution can omit the skill; runtime fetch
     makes normal skill use network-dependent and weakens reproducibility. A subtree still creates
     the same large vendored content while adding update mechanics.
   - Effort: Medium/High

4. **Thin local wrapper with remote references** — add only a local `SKILL.md` that points agents to
   GitHub raw files.
   - Pros: smallest diff.
   - Cons: broken/offline progressive disclosure, no local scripts/assets, and URL content becomes
     an unpinned execution input. It is not a complete integration.
   - Effort: Low, unacceptable capability fidelity

### Recommendation

Choose Approach 1. Vendor the complete `skills/diagram-design/` capability as a source-controlled,
version-pinned snapshot under the harness's existing `skills/design/` ecosystem. Do not add the
upstream repository to `opencode.json` unless it later publishes a verified OpenCode plugin package.
Adapt only the three useful upstream command surfaces into OpenCode's local `commands/` format;
do not vendor Claude marketplace manifests, Pi prompts, upstream CI, screenshots, or repository
contributor documentation because they do not participate in OpenCode skill discovery.

The snapshot must preserve all files needed by the skill's relative links: `SKILL.md`, all
`references/`, `scripts/`, and `assets/`. Include the MIT notice and third-party icon provenance
near the vendored skill. Add an explicit refresh note with the upstream commit and version so a
future update is deliberate rather than an unreviewed network mutation.

### Scope

#### In scope

- Native OpenCode discovery of the Diagram Design skill after harness installation.
- Complete relative reference/script/asset tree required for static diagrams, imports, exports,
  semantic patterns, accessibility, optional motion, and self-checking.
- OpenCode command adapters for diagram export and draw.io/Mermaid redraw workflows.
- Inventory/README/provenance/license updates and focused smoke validation.

#### Out of scope

- Adding unsupported OpenCode plugin hooks or changing existing MCP/agent permissions.
- Automatic upstream updates, runtime downloads, or a new package manager dependency.
- Vendoring upstream marketplace manifests, CI workflows, screenshots, ADR/contributor docs, or
  non-skill root files that OpenCode will not load.
- Claiming generated diagrams are product-accepted; this change integrates instructions and local
  tooling only.

### Risks

- **Snapshot drift:** upstream changes may improve or break relative references; pin and record the
  source commit, and rerun the focused checks when refreshing.
- **Large review surface:** the asset/reference tree may exceed the 400-line guard; split the apply
  work into reviewable slices or use the approved size strategy before implementation.
- **License/provenance:** bundled Tabler, Simple Icons, log-z, Devicon, and one-off logo sources
  need to remain attributed beside the vendored content.
- **Command portability:** export PNG requires Playwright/Chromium and import scripts require
  Python; commands must surface unavailable tooling rather than installing it implicitly.
- **Deployment skew:** source checkout and `~/.config/opencode`/dotfiles can diverge; verify native
  discovery from the effective installed path, not only this checkout.
- **Brand gate:** the skill intentionally pauses on its default style guide; agents must not remove
  that gate or silently customize brand tokens.

### Ready for Proposal

Yes. The proposal should authorize a pinned, complete skill snapshot plus three local command
adapters, explicitly reject the unverified Git-plugin option, define the excluded upstream files,
and include rollback as removal of the vendored snapshot/commands plus inventory reversion. It
should also set the validation boundary: frontmatter/reference/path checks and upstream smoke
scripts are technical evidence, not application acceptance.

---

**Status**: success
**Executive Summary**: The upstream repository is Agent Skills-compatible but not a verified OpenCode plugin package. A complete, pinned local snapshot is the safest way to preserve its progressive-disclosure references, scripts, and assets; OpenCode-native commands should be adapted separately.
**Artifacts**: `openspec/changes/integrate-diagram-design-skills/exploration.md`, `openspec/changes/integrate-diagram-design-skills/state.yaml`
**Next Recommended**: `sdd-propose`
**Risks**: Snapshot size/drift, third-party license provenance, optional Python/Playwright tooling, deployment skew, and the upstream default-style gate.
