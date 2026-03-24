# Architecture Index

This directory contains the approved software architecture for CeeVee.

Primary audience:

- implementation LLMs
- planning agents
- review agents

Interpretation rule:

- each file in this directory is authoritative only for its own topic
- do not merge rules from multiple files when one file already owns the topic
- use cross-references instead of inventing duplicate rules
- if two files appear to overlap, prefer the file whose topic name most directly matches the question

## Documents

- [overview.md](./overview.md)
  Defines global architecture decisions, invariants, and non-goals.

- [system-context.md](./system-context.md)
  Defines external actors, external dependencies, and high-level system flows.

- [module-design.md](./module-design.md)
  Defines codebase placement, layering, and module ownership.

- [interfaces.md](./interfaces.md)
  Defines entry-point behavior and shared capability-layer rules for HTTP and MCP.

- [boundaries.md](./boundaries.md)
  Defines cross-boundary constraints and “must not cross” rules.

- [port-contracts.md](./port-contracts.md)
  Defines architectural port contracts for external dependencies.

- [storage.md](./storage.md)
  Defines storage responsibilities and relational-plus-vector storage semantics.

- [user-perspective.md](./user-perspective.md)
  Defines user-visible system behavior and the learning loop semantics.

- [data-model.md](./data-model.md)
  Defines entities, relationships, lifecycle meaning, and truth-model rules.

- [runtime-observability.md](./runtime-observability.md)
  Defines runtime semantics, sync-vs-async rules, triggers, and observability requirements.

## Relationship Between Documents

- `overview.md` owns global architecture rules.
- `system-context.md` owns outside-in system context.
- `module-design.md` owns internal placement and layering.
- `interfaces.md` owns entry-point semantics.
- `boundaries.md` owns cross-boundary prohibitions.
- `port-contracts.md` owns external dependency contracts.
- `storage.md` owns storage-form semantics.
- `user-perspective.md` owns product behavior from the user viewpoint.
- `data-model.md` owns entity meaning and lifecycle semantics.
- `runtime-observability.md` owns execution and trigger semantics.

Execution rule for downstream LLMs:

- read only the files needed for the current task
- do not infer implementation details when the owning file defines only architecture-level constraints
- if a needed detail is not defined, preserve the existing architecture and avoid inventing new architecture silently
