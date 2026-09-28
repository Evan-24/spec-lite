---
name: spec-lite
description: Maintain lightweight project context and task contracts when the developer explicitly invokes $spec-lite to initialize a project, create or update a task spec, resume specified work, or finish a spec. Do not invoke for ordinary development tasks.
---

# Spec Lite

Use the smallest durable context that helps a developer and Codex carry complex work across sessions. The developer decides when to use this skill. Never infer its use from task size, keywords, or the number of files involved.

Treat code, tests, and real configuration as the source of truth. Git records changes; Spec Lite records intent and durable reasoning. Keep generated content concise and use the user's or project's prevailing language.

## Route the explicit request

- `init`: build a minimal project map.
- `task`: create or maintain a task contract.
- `finish`: complete the relevant task spec and review durable knowledge.
- When the developer explicitly invokes `$spec-lite` with natural language, choose the matching behavior from their intent.
- When continuing a named spec, read the root `AGENTS.md`, `spec-lite/PROJECT.md`, that spec, and only the relevant current code before proceeding.

## Initialize a project

Inspect progressively rather than scanning the repository exhaustively:

1. Read the root structure, README, root `AGENTS.md`, manifests, and build and test configuration.
2. Use that context to locate the main application, runtime, API, data, or other core entry points that actually exist.
3. Read a small representative set of core files to verify the map, then stop once the purpose, core modules, main entry points, important flows, and development and test approach are clear.

Create or minimally supplement the root `AGENTS.md`. Preserve existing content and add only evidenced, stable rules that should govern future development; never overwrite it or create nested `AGENTS.md` files as part of this workflow.

Keep all Spec Lite-owned project documents under the dedicated root `spec-lite/` directory so they do not collide with a project's existing documentation structure.

Create or update `spec-lite/PROJECT.md` from [assets/PROJECT.template.md](assets/PROJECT.template.md). Keep it a low-granularity navigation aid, not a second copy of the code. Do not reconstruct decisions from project history during initialization.

## Create or maintain a task spec

Create `spec-lite/specs/<meaningful-kebab-case-name>.md` from [assets/TASK_SPEC.template.md](assets/TASK_SPEC.template.md). Capture only the goal, scope, non-goals, constraints, and acceptance criteria. Base content on the developer's request and known project facts; ask only when a missing choice would materially change the contract.

If requirements change but retain the same goal, update the current spec. Create a new spec only for an independent goal. Do not use version suffixes or numbering systems.

Keep implementation plans, task lists, progress logs, ownership, priority, timelines, proposals, and change logs out of the spec. Plan dynamically from the current code; do not synchronize the plan into the task contract.

## Validate project knowledge lazily

Use `spec-lite/PROJECT.md` to navigate, then validate only the claims relevant to the current work against current code, tests, and configuration. Correct a relevant stale claim when discovered. Do not scan unrelated modules for drift or add automatic freshness, indexing, or synchronization machinery.

## Finish a spec

Confirm the relevant spec and actual implementation. If more than one spec could be meant and context does not resolve it, ask the developer which one to finish.

In the completed spec:

- Change `Status: Active` to `Status: Completed`.
- Append a brief `Outcome` describing what was delivered and any important divergence from the contract.
- Append a brief `Verification` describing how completion was established.
- Keep the spec in `spec-lite/specs/`; do not archive it.

Then perform a lightweight knowledge review:

- Update the root `AGENTS.md` only for a newly established, durable rule that future agents must follow.
- Update only the relevant parts of `spec-lite/PROJECT.md` when core modules, primary entry points, important flows, or project-level structure changed.
- Create `spec-lite/decisions/<meaningful-kebab-case-name>.md` from [assets/DECISION.template.md](assets/DECISION.template.md) only for an important, lasting design choice whose rationale will matter later.

Do not mechanically produce all three. Ordinary implementation details, local fixes, and facts easily rediscovered from code do not belong in durable knowledge.
