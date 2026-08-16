# Proposal: Integrate Diagram Design Skills

## Intent

Integrate upstream Diagram Design locally. No verified OpenCode plugin contract exists; a complete
pinned snapshot is reproducible offline.

## Objectives / Non-objectives

- **Objectives:** native discovery; commit `a5e3978088cf89c7caff5c20cabd99fbc2a301de` / version
  `2.3.5`; three commands; auditable inventory, provenance, and licensing.
- **Non-objectives:** upstream URL in `opencode.json`, runtime fetch/refresh, new dependencies,
  marketplace manifests, production-code changes, TDD claims, or product acceptance.

## Scope

### In Scope

- Copy `skills/diagram-design/` to `skills/design/diagram-design/`, preserving `SKILL.md`,
  `references/`, `scripts/`, and `assets/` with relative links intact.
- Add `commands/diagram-export.md`, `commands/diagram-import-drawio.md`, and
  `commands/diagram-import-mermaid.md`; report missing Python/Playwright/Chromium honestly.
- Update `README.md`, `AGENTS.md`, `.agents/skill-registry.md`, and provenance/license notices. Run
  filesystem/frontmatter/reference and upstream smoke checks; technical evidence only, not TDD or
  product acceptance.

### Out of Scope

- Treating the GitHub URL as an OpenCode plugin because no verified contract exists.
- Upstream CI/screenshots/docs, network updates, or MCP/permission changes.

## Capabilities

### New Capabilities

- `diagram-design-skill`: pinned snapshot and provenance.
- `diagram-command-adapters`: native export, draw.io import, and Mermaid import.

### Modified Capabilities

- None.

## Approach

Copy the pinned skill tree, preserve relative resources, and record MIT/third-party attribution.
Keep `opencode.json` unchanged. Use thin adapters with no implicit installation and explicit
unavailable-tool paths.

## Affected Areas

| Area | Impact | Description |
|---|---|---|
| `skills/design/diagram-design/` | New | Pinned skill, references, scripts, assets. |
| `commands/diagram-*.md` | New | Three native adapters. |
| `README.md`, `AGENTS.md`, `.agents/skill-registry.md` | Modified | Inventory and routing docs. |

## Delivery Slices

1. Snapshot, integrity/license metadata.
2. Three adapters and unavailable-tool paths.
3. Inventory/docs and smoke evidence.

If the tree exceeds 400 lines, use stacked file-family slices with per-slice verification/rollback.

Decision needed before apply: Yes
Chained PRs recommended: Yes
400-line budget risk: High

## Rollout / Backward Compatibility

Additive checkout-to-installed-path rollout. Skills, commands, plugins, and config remain unchanged;
refreshes require a new pin. Verify checkout and installed path.

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Snapshot drift or license omission | Med | Pin identity, manifest, MIT, and third-party notices. |
| Optional tooling absent | Med | Report `UNAVAILABLE`/blocked; never install implicitly. |

## Rollback Plan

Revert snapshot, adapters, inventory, and notices; restore docs/registries. Preserve the existing
OpenPencil plugin entry and unrelated skills; no migration is required.

## Dependencies

- Upstream commit/version/notices; optional Python and Playwright/Chromium.

## Success Criteria

- [ ] Four snapshot areas and relative references exist at the pinned identity.
- [ ] OpenCode discovers the skill and all three commands without an upstream plugin URL.
- [ ] Inventory, provenance, MIT, and third-party licensing are auditable.
- [ ] Smoke evidence is deterministic and honest about unavailable tooling, with no TDD/product-
  acceptance claim; each slice follows the approved review strategy.
