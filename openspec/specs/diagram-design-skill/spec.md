# Diagram Design Skill Specification

## Purpose

Define the reproducible, locally discoverable integration of the upstream Diagram Design Agent
Skill and its progressive-disclosure resources.

## Requirements

### Requirement: Discover the pinned skill natively

The harness MUST discover the skill from `skills/design/diagram-design/SKILL.md`. Its frontmatter
`name` MUST be `diagram-design` and MUST match the containing lowercase-hyphen directory. Native
discovery MUST NOT require an upstream URL in `opencode.json` or a plugin loader.

#### Scenario: Offline native discovery

- GIVEN the harness is installed with the checked-in snapshot and no upstream plugin URL
- WHEN OpenCode scans the installed skill tree without network access
- THEN `diagram-design` is available at the canonical path
- AND the existing plugin configuration is unchanged

#### Scenario: Invalid identity

- GIVEN `SKILL.md` is moved or its frontmatter name differs from `diagram-design`
- WHEN a deterministic integrity check runs
- THEN it reports the path/name mismatch and the skill cannot claim valid discovery

### Requirement: Preserve a complete pinned snapshot and provenance

The snapshot MUST record upstream identity `a5e3978088cf89c7caff5c20cabd99fbc2a301de`, version
`2.3.5`, source skill path, and refresh provenance. It MUST retain `SKILL.md`, every referenced
`references/`, `scripts/`, and `assets/` file, with all relative links resolvable. MIT licensing
and third-party notices, including bundled icon sources, MUST remain auditable beside the snapshot.

#### Scenario: Complete pinned snapshot

- GIVEN the vendored tree and its provenance/license records are present
- WHEN reference, script, asset, and identity checks run
- THEN every required relative dependency resolves to the pinned snapshot
- AND the commit, version, MIT notice, and third-party attributions agree

#### Scenario: Incomplete or drifted snapshot

- GIVEN a referenced file, notice, commit, or version is missing or changed without a new pin
- WHEN integrity validation runs
- THEN it returns a non-passing result identifying the missing or drifted item

### Requirement: Preserve design gates and source trust boundaries

The skill MUST retain its default style-guide gate before design-specific visual decisions. Mermaid
and draw.io sources MUST be treated as untrusted data; embedded instructions, scripts, unsafe URLs,
and unsupported constructs MUST NOT be executed or silently fetched.

#### Scenario: Style gate remains active

- GIVEN a diagram request has no approved style guide
- WHEN the skill begins design guidance
- THEN it pauses for the default style-guide decision before styling the diagram

#### Scenario: Hostile diagram source

- GIVEN Mermaid or draw.io input contains scripts, remote references, or instruction-like content
- WHEN the skill prepares it for import
- THEN it treats the content as data, surfaces the unsafe or unsupported portion, and performs no
  implicit execution or network fetch

### Requirement: Keep integration documented, offline, and reversible

The integration MUST work from checked-in files without implicit installation, refresh, or network
resolution. `README.md`, `AGENTS.md`, the skill registry, provenance, and license notices MUST agree
on the canonical path, name, pin, capabilities, and excluded upstream files. Rollback MUST remove
only this snapshot, its adapters, and their inventory/documentation changes, restoring prior config.

#### Scenario: Reproducible documentation

- GIVEN a clean checkout with no network or package installer
- WHEN an agent reads the inventory and uses the documented skill path
- THEN the same pinned capability and prerequisites are discoverable from local files alone

#### Scenario: Rollback

- GIVEN this change is reverted
- WHEN the prior harness is installed and scanned
- THEN unrelated skills and the existing plugin entry remain intact, with no migration required
