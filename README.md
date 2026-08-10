# Agent Swarm

Agent orchestration workspace that consumes reusable agent harnesses as Git submodules and keeps
the Spec-Driven Development (SDD) artifacts of this workspace in `openspec/`.

The primary harness, [`agent-harness`](https://github.com/yacosta738/agent-harness), is an
OpenCode agent configuration: the Kerrigan architect persona, SDD phase executors, skills,
prompts, commands, plugins, and MCP integrations.

## Layout

```
agent-swarm/
├── agent-harness/        # Submodule — OpenCode agent configuration + SDD pipeline
├── codegauge/            # Submodule — CRAP metric tool (code health analysis)
├── openspec/             # SDD artifacts of this workspace (specs, changes, quality config)
├── .gitmodules           # Submodule registry
└── README.md
```

## Submodules

| Submodule       | Remote                                          | Notes                                                          |
| --------------- | ----------------------------------------------- | -------------------------------------------------------------- |
| `agent-harness` | `git@github.com:yacosta738/agent-harness.git`   | Pinned to a local commit; see [Working state](#working-state). |
| `codegauge`     | `git@github.com:yacosta738/codegauge.git`       | CRAP metric tool; bootstrapped with initial commit `d53bdb7`.  |

## Quick start

Clone with submodules:

```sh
git clone --recurse-submodules git@github.com:yacosta738/agent-swarm.git
```

If the repository is already cloned without submodules:

```sh
git submodule update --init --recursive
```

Update submodules to the commit recorded in the index:

```sh
git submodule update --remote --recursive
```

> Note: `--remote` moves submodules to the latest upstream branch. To keep the deterministic
> pinned behavior, prefer plain `git submodule update --init --recursive`.

## Working state

- The `agent-harness` submodule points to local commit `a4d9b4b`, which diverges from
  `origin/main` (`da453f8`). The only difference is that `openspec/` artifacts were moved out of
  the submodule into this workspace's root. The submodule working tree is clean.
- `openspec/` at the workspace root is currently untracked; it holds the SDD artifacts
  (`config.yaml`, `quality-runner.json`, specs, changes) and should be committed once the
  workspace baseline is agreed.

## Spec-Driven Development

The workspace uses OpenSpec filesystem artifacts:

- `openspec/specs/` — canonical specifications.
- `openspec/changes/` — active and archived change records (proposal, spec, design, tasks,
  verification, and QA reports).
- `openspec/config.yaml` — SDD policy: scenario format (Given/When/Then), RFC 2119 keywords,
  task grouping rules, and QA verdict policy.
- `openspec/quality-runner.json` — capability-driven quality runner configuration (currently
  disabled).
