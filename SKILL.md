---
name: spec-lite
description: Distill durable project context, task contracts, and verified facts when the developer explicitly invokes $spec-lite to initialize a project, create or update a spec, resume specified work, or finish a spec. Do not invoke for ordinary development tasks.
---

# Spec Lite

Spec Lite is a lightweight context distillation protocol for coding agents. Use the smallest durable context that helps a developer and an agent carry work across sessions and phases. The developer decides when to use this skill. Never infer its use from task size, keywords, or the number of files involved.

Code, tests, and real configuration establish current reality; Git records changes. PROJECT provides navigation; AGENTS holds durable work rules; specs preserve intent and contracts; facts preserve expensive empirical lessons; decisions preserve lasting rationale. Preserve commitments, not conversations or reasoning history. Keep generated content concise and use the user's or project's prevailing language.

## Route the explicit request

- `init`: build a minimal project map.
- `task`: create or maintain a task contract; `init` is not a prerequisite.
- `finish`: complete the relevant task spec and review durable knowledge.
- When the developer explicitly invokes `$spec-lite` with natural language, choose the matching behavior from their intent.
- When continuing a named spec, read the root `AGENTS.md` if present, that spec, relevant facts and decisions, and current code. Use `spec-lite/PROJECT.md` for navigation when present. For a child, also load its parent and inherit shared intent and constraints; do not load every sibling or previous stage by default.
- Without PROJECT, use the README, root AGENTS, manifests, and relevant code for minimal context. Do not automatically run init.

Keep Spec Lite-owned project documents under the dedicated root `spec-lite/` directory. Discover active specs with search (for example, `rg -l 'Status: Active' spec-lite/specs`), not a registry or index.

## Initialize a project

Inspect progressively rather than scanning the repository exhaustively:

1. Read the root structure, README, root `AGENTS.md`, manifests, and build and test configuration.
2. Use that context to locate the main application, runtime, API, data, or other core entry points that actually exist.
3. Read a small representative set of core files to verify the map, then stop once the purpose, core modules, main entry points, important flows, and development and test approach are clear.

Create or minimally supplement the root `AGENTS.md`. Preserve existing content and add only evidenced, stable rules that should govern future development; never overwrite it or create nested `AGENTS.md` files as part of this workflow.

Create or update the optional `spec-lite/PROJECT.md` from [assets/PROJECT.template.md](assets/PROJECT.template.md). Keep it a low-granularity navigation aid: purpose, stack, modules, entry points, and important flows. Task details, test baselines, debugging findings, and plans belong elsewhere. Do not reconstruct decisions from project history or create empty facts/decisions during initialization.

## Create or maintain a task spec

By default, create `spec-lite/specs/<meaningful-kebab-case-name>.md` from [assets/TASK_SPEC.template.md](assets/TASK_SPEC.template.md). Capture the goal, scope, non-goals, constraints, and observable acceptance criteria. Base content on the developer's request and established facts; ask only when a missing choice would materially change the contract.

Use a topic directory only for a real parent/child decomposition or an existing repository convention, never merely because the spec count grew. For naturally separable, independently verifiable stages:

- A parent such as `spec-lite/specs/<topic>/spec.md` holds the shared goal, scope, non-goals, constraints, developer-confirmed direction, and a `Stages` section linking children with necessary dependencies.
- Each child uses the ordinary task contract with `Parent: ./spec.md` (relative to the child). It inherits parent intent and constraints unless explicitly narrowed; add only local boundaries. Narrowing cannot silently relax a shared constraint.
- Keep the default task template unchanged. Add Parent, Stages, or task-local `Verified Facts` only when needed; standalone tasks remain sufficient.

If requirements change but retain the same goal, update the current spec. Create a new spec only for an independent goal. Do not use version suffixes or numbering systems.

Keep file-level steps, dynamic TODO lists, progress logs, percentages, ownership, priority, estimates, timelines, proposals, and change logs out of specs, including parents. Stage decomposition is not an execution script. Plan dynamically from current code; do not persist or synchronize the execution plan.

At exploration-to-implementation or stage boundaries, recommend a fresh context when useful; do not require a reset or use a token threshold. Distill approved direction into parent/child contracts and necessary facts/decisions first. Load the parent and active child, relevant durable knowledge, and current code; previous stages are needed only when their outcomes cannot be recovered from current reality. A fork carrying the old discussion is not a context reset.

## Record and reuse verified facts

Record an empirical finding only when all three hold: it is not easily recovered from code, tests, or configuration; rediscovery has meaningful cost; and future work is likely to encounter it again. Task-local findings may use an optional `Verified Facts` section. Clearly reusable cross-task findings belong in optional `spec-lite/FACTS.md`, using [assets/FACTS.template.md](assets/FACTS.template.md). Record eligible findings during work without asking for routine confirmation.

State what was observed, how to recognize it using stable signatures, when and in what relevant environment it was verified, and which events warrant revalidation. Separate observed facts from constraints and hypotheses; do not promote an unexplained failure to a known baseline.

Before substantial investigation of test, runtime, toolchain, platform, environment, or external-service anomalies, consult relevant recorded facts. Compare signatures and conditions, not just failure counts; investigate new or changed observations rather than dismissing all failures as baseline.

Revalidate relevant facts on observation mismatch, causal changes to code/environment/dependencies, or explicit challenge. Age alone is not a trigger: no TTL or scheduled refresh. Update or remove invalidated facts in place; keep only currently useful knowledge. Git preserves history, so do not add deprecated entries, archives, fact IDs, databases, or history logs.

## Validate project knowledge lazily

Use PROJECT when available, then validate only claims relevant to the current work against current code, tests, and configuration. Correct relevant stale claims when discovered. Prefer durable semantic anchors (identifiers, module names, config keys, test names, regex, commands, stable paths) over file-and-line snapshots in all durable documents.

Do not scan unrelated modules for drift or add freshness services, knowledge indexes, synchronization, conflict detection, dependency graphs, stage schedulers, plan executors, progress dashboards, or lifecycle services.

## Finish a spec

Confirm the relevant spec and actual implementation. If more than one spec could be meant and context does not resolve it, ask the developer which one to finish.

In the completed spec:

- Change `Status: Active` to `Status: Completed`.
- Append a brief `Outcome` describing what was delivered and any important divergence from the contract.
- Append a brief `Verification` describing how completion was established.
- Keep the spec in `spec-lite/specs/`; do not archive it.

Complete a child the same way. Child completion does not automatically complete the parent: verify the parent's shared contract and relevant stage outcomes before marking it completed. Do not require reading all sibling discussions.

Then perform a lightweight knowledge review:

- Update the root `AGENTS.md` only for a newly established, durable rule that future agents must follow.
- Update only the relevant parts of an existing `spec-lite/PROJECT.md` when core modules, primary entry points, important flows, or project-level structure changed.
- Record or update an eligible cross-task fact in `spec-lite/FACTS.md`; retain task-local findings in the completed spec when useful.
- Create `spec-lite/decisions/<meaningful-kebab-case-name>.md` from [assets/DECISION.template.md](assets/DECISION.template.md) only for an important, lasting design choice whose rationale will matter later.

Do not mechanically update or create any of these documents. Ordinary implementation details, local fixes, temporary workarounds, and facts easily rediscovered from code do not belong in long-term knowledge.
