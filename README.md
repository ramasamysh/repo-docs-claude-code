# Repository Documentation for Claude Code

A source-evidence-first workflow for creating useful engineering documentation from an existing code repository. Designed for governed environments with limited network access. No external APIs or MCP servers are required.

## Quick start (one-time)

1. Open the target repository in Claude Code.
2. Paste the text in `prompts/one-time-prompt.md` into the Claude Code conversation.
3. Ask for the **inventory** first, then the **generate** and **verify** steps.
4. Review generated documents with the module owners before merging.

## Reusable skill (recommended after the first trial)

Copy the provided `.claude/skills/repo-docs/SKILL.md` into the target repository at the same relative path. Copy the agent files from `.claude/agents/` too if you want specialized explorers and verification.

In Claude Code, run these prompts (each as a separate request):

- `/repo-docs inventory` — map the repository and propose a documentation plan without modifying files.
- `/repo-docs generate` — create or update the V1 documents from the plan, touching only documentation files.
- `/repo-docs verify` — audit documentation claims against source and correct doc-only errors.
- `/repo-docs flow <business-process>` — trace and document a specified business path.

If a newly created `.claude/agents` directory isn't detected, restart the Claude Code session. You do **not** need the `/agents` management command.

If organization policy blocks project skills or agents, use the one-time prompt alone; it doesn't require either feature.

## Suggested V1 output

```
docs/
  index.md
  architecture/
    overview.md
    components.md
  flows/
    <selected-flow>.md
  integrations.md
  documentation-gaps.md
```

For batch-heavy systems, also document one scheduled batch/job flow. Add operations, data model, API catalog, and per-module pages only after V1 is verified.

## Principles

- **Source first:** Trace the behavior to code, configuration, tests, or migration artifacts.
- **No guessing:** Separate verified facts, inference, and unknown operational details.
- **Incremental:** Inventory the whole repository, then explore only the files that are relevant to one topic at a time.
- **Evidence:** Provide repo-relative paths and exact symbols; only use line numbers if verified.
- **Docs only:** Don't change production code, configs, builds, tests, CI, or agent policies.
- **Safe:** Never copy secrets, production credentials, customer data, or sensitive values into docs.
- **No external network:** Don't fetch or transmit repository content externally. Only use read access to files and locally permitted commands.
- **Review:** Have a human validate intended business semantics and deployment claims.

## Useful references

- Claude Code skills: https://code.claude.com/docs/en/skills
- Claude Code subagents: https://code.claude.com/docs/en/sub-agents
- Claude Code exploration and documentation: https://code.claude.com/docs/en/common-workflows
