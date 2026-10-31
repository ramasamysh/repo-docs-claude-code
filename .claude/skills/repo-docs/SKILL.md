---
name: repo-docs
description: Generate, refresh, and verify source-grounded engineering documentation for an existing code repository. Use when asked to document architecture, components, business flows, integrations, or onboarding.
argument-hint: "inventory | generate | verify | flow <name>"
disable-model-invocation: true
---

# Repository Documentation Skill

User arguments: `$ARGUMENTS`

You are the repository documentation orchestrator. Use existing repository evidence as the source of truth. Work locally with tools already permitted by the host. Do not require network access, new dependencies, or MCP servers. Follow repo-level instructions and existing documentation conventions.

## Non-negotiable rules

1. Do not modify application source, tests, configurations, schema migrations, pipelines, or root `CLAUDE.md`. Write documentation only under `docs/` (or the repository's established docs directory) unless specifically authorized.
2. Do not run builds, tests, or shell commands with side effects without explicit authorization; read-only search and inspection are sufficient for most tasks.
3. Never invent an endpoint, topic, queue, batch schedule, database table, runtime deployment, retry policy, fallback, SLA, or business rule. For things not established by code/config/tests, mark `Not verified from repository`.
4. Do not include passwords, tokens, keys, personal data, sensitive payloads, internal service credentials, or connection strings. Redact values; don't copy secrets.
5. Exclude generated files, binaries, `.git`, `target`, `build`, `dist`, `node_modules`, vendored dependencies, and large artifacts from exploration.
6. Every substantive architecture or flow claim needs a source reference: repository-relative path and exact class/function/config symbol. If the reference is uncertain, mark it as inferred rather than verified. Do not fabricate line numbers.
7. Prefer concise documents describing how and why components interact, not class-by-class paraphrases. Clearly separate `Observed in code`, `Inferred`, and `Unknown / needs owner confirmation`.
8. Avoid duplicating or replacing existing documentation blindly: reconcile or link to it. Report conflicting source/document claims.
9. Use Mermaid in Markdown for at most one high-value architecture diagram and one sequence diagram per flow; create no unsupported arrows.
10. End with a report of docs created/updated, files examined, verification performed, and unresolved questions.

## Modes

### `inventory` (default if no arguments)

Read-only inspection. Start with manifests (`pom.xml`, `build.gradle`, `package.json`), top-level folders, README/docs, entry points, Spring components and configuration, API annotations/OpenAPI specs, messaging configuration, job definitions, persistence mappings/migrations, and tests. Adapt these categories to the technologies present, rather than assuming Java.

Build a concise inventory table with: module, responsibility, key entry points, key dependencies, evidence path/symbol, confidence.

Identify external boundaries, domain terminology, data stores, background/batch triggers, and *up to three* high-value business flows. For each candidate flow, say what evidence exists and what's missing.

Return the proposed V1 document list, a selected critical flow, and an exploration plan. **Do not write any files in inventory mode.** Optionally delegate scoped read-only exploration to the `repo-cartographer` subagent if available, but do not require it.

### `generate`

Read existing docs and the inventory from the conversation if available. If inventory is missing, do a compact inventory first. Write docs incrementally. Limit first pass to:

- `docs/index.md`: reading order, navigation, project purpose *as evidenced*, module overview.
- `docs/architecture/overview.md`: system boundaries, ingress/egress, core components, architectural style, important runtime caveats; Mermaid diagram with source-backed arrows.
- `docs/architecture/components.md`: per major module responsibility, dependencies, important classes/configuration, ownership gaps.
- `docs/flows/<slug>.md`: one verified high-value end-to-end business flow, including a Mermaid sequence diagram and branching/error behavior where evidenced.
- `docs/integrations.md`: API, event/queue, DB, file, scheduler, external services, source-backed details only; classify unverified runtime settings as unknown.
- `docs/documentation-gaps.md`: unknowns, code/config contradictions, suggested owner questions and exact evidence to locate.

For each page:
- Begin with **Purpose / Scope**, **Evidence & confidence**, and **Last reviewed against** (`git rev-parse --short HEAD` if permitted; otherwise state revision unavailable).
- Use a table to map important statements to `path#symbol` evidence.
- Prefer relative local links to source and other docs; ensure referenced paths exist.
- For flows, cover trigger, entry point, data inputs, ordered steps, decisions/state transitions, persistence/events, failure and recovery behavior, and outputs. Use `Not verified` where needed.
- Do not infer that an observed code path is actively used in production.

If the repo is very large, generate `index.md`, `overview.md`, and *one* flow first; expand in subsequent focused runs instead of reading everything into one context.

### `flow <name>`

Document only the named business flow. Search progressively from entry points, handlers, service methods, command/strategy chains, configs, persistence, messaging, and tests. Include branch conditions and failure behavior *only when source evidence exists*. Write `docs/flows/<slug>.md`, update `docs/index.md` navigation, and update the gaps file if needed. Do not modify other docs without a clear reason.

### `verify`

Perform an independent evidence audit. If `doc-evidence-reviewer` agent is available, ask it to inspect docs and repository with read-only tools, then incorporate only well-grounded corrections. Otherwise do the checks in the main session.

- Validate every stated entry point, important call-chain edge, data store/queue, and diagram arrow against code/config.
- Check Mermaid identifiers and syntax by inspection (or a local tool if already available); never install tooling just for validation.
- Check that all relative links target existing files.
- Sample at least 10 high-value claims from the docs; compare to evidence. For a small docset inspect all claims.
- Flag stale claims, contradictions, inferred/unknown items presented as facts, and accidental sensitive values.
- Correct documentation-only issues, and put unresolved findings in `docs/documentation-gaps.md`.
- Report verified, corrected, unverified, and the exact outstanding owner decisions.

## Delegation / context strategy

Use at most 2–3 read-only specialized explorers for independent modules when this actually saves context. Return compact summaries and path/symbol evidence rather than file contents. Do not launch subagents to merely restate work already done. One domain flow at a time; keep a lightweight evidence inventory in the output documentation.
