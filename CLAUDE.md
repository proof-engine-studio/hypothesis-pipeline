# AgenticTeamsForDevelopment

Hypothesis testing pipeline: from idea to validated (or invalidated) market signal.

## Pipeline

```
/hypothesize <idea>
    └─► /spec [id]              ← CHECKPOINT: scope approval
            └─► /arch [id]      ←── parallel ──► /design-system [id]  (only if has_ui)
                    └─► /build [id]
                            └─► /qa [id]                          ← BLOCKER: must pass to ship
                                    └─► /ship [id] [staging|production]  ← CHECKPOINT: deploy approval
                                            └─► /promote [id]             ← CHECKPOINT: ad spend approval
                                                        └─► /measure [id]
```

## State

All pipeline state lives in `state/{hypothesis-id}/context.json`.
Schema: `schemas/handoff.schema.json`.
Each agent reads ONLY its designated fields and writes ONLY to its output block.

Hypothesis IDs: `hyp-YYYYMMDD-NNN` (e.g. `hyp-20260501-001`).

## Agent Roster

| Agent | File | Model | Reads | Writes |
|-------|------|-------|-------|--------|
| IdeationAgent | `agents/hypothesis.md` | **opus** | arguments | `hypothesis` block |
| SpecAgent | `agents/spec.md` | sonnet | `hypothesis`, `summary` | `spec` block |
| **ArchAgent** | `agents/arch.md` | **opus 1M** | `hypothesis`, `spec`, `summary` + references | `arch` block |
| **DesignSystemAgent** | `agents/design-system.md` | **opus 1M** | `hypothesis`, `spec`, `summary` + references | `design_system` block |
| BuilderAgent | `agents/builder.md` | sonnet | `hypothesis`, `spec`, `arch`, `design_system`, `summary` | `build` block |
| QAAgent | `agents/qa.md` | sonnet | `spec`, `arch`, `build`, `design_system`, `summary` | `qa` block |
| DeployerAgent | `agents/deployer.md` | sonnet | `build`, `qa`, `summary` | `deploy` block |
| MarketerAgent | `agents/marketer.md` | sonnet | `hypothesis`, `deploy`, `summary` | `marketing` block |
| AnalystAgent | `agents/analyst.md` | sonnet | `hypothesis`, `deploy`, `marketing`, `summary` | `analysis` block |
| MemoryAgent | `agents/memory.md` | haiku | ALL | `summary` field |

**Foundation tier (Opus 1M):** ArchAgent and DesignSystemAgent get the full Opus + 1M context window because their decisions are hardest to reverse downstream. They produce thorough, reasoned foundations that BuilderAgent can implement without architectural questions.

## Token Efficiency Rules (enforced in every agent)

1. Read ONLY the JSON fields listed in the agent's "Input" section
2. Write ONLY to the designated output block
3. Never echo back input — act on it
4. CLI output: use `--json` + `jq`, extract ≤5 fields per call
5. Summarize any tool result to ≤5 facts before reasoning on it
6. If `summary` field exists and covers your needed context — read it instead of upstream blocks
7. Stop immediately when success criteria are met

## Checkpoint Protocol

Three mandatory checkpoints where agent HALTS and waits for `approved`:
- **Spec**: SpecAgent, before creating Figma/Confluence artifacts
- **Deploy**: DeployerAgent, before `git push` triggering GitHub Actions
- **Ad spend**: MarketerAgent, before activating any paid campaign

Agent writes `state/{id}/checkpoint.json`, prints CHECKPOINT block, waits.
Human types `approved` to proceed or `revise: {feedback}` to iterate.

## MCP Integration Map

| Agent | MCP | Operation |
|-------|-----|-----------|
| IdeationAgent | Intercom | customer signal from recent conversations |
| IdeationAgent | Linear | create hypothesis issue |
| SpecAgent | Figma | create design brief |
| SpecAgent | Atlassian | create Confluence spec page |
| SpecAgent | Linear | attach spec to issue |
| ArchAgent | Atlassian | create Confluence arch page |
| DesignSystemAgent | Figma | create design tokens frame |
| BuilderAgent | Linear | update issue progress |
| QAAgent | Chrome | E2E browser tests, accessibility (only if has_ui) |
| QAAgent | Linear | comment with QA verdict and bug count |
| DeployerAgent | Kubernetes | verify cluster, post-deploy check |
| DeployerAgent | Slack | post to #deployments |
| MarketerAgent | Chrome | navigate ad platforms |
| MarketerAgent | Slack | post to #marketing |
| AnalystAgent | Intercom | post-launch conversation themes |
| AnalystAgent | Slack | post report to #metrics |

Note: Sentry operations use `sentry` CLI binary (sentry-cli skill already installed).
Note: GitHub operations use `gh` CLI.

## Deploy Model

Infrastructure: server with Docker + Kubernetes + domain + GitHub hosted runners.
BuilderAgent generates: `Dockerfile`, `.github/workflows/deploy.yml`, `k8s/` manifests.
DeployerAgent: shows plan → waits for `approved` → `git push origin main` → monitors GitHub Actions.

## Behavioral Contracts (Hard + Soft Constraints)

**Hard constraints** (never violate):
- Never deploy to production without human typing 'approved' at checkpoint
- Never activate paid campaigns without human typing 'approved' at checkpoint
- Never write to a context.json block owned by a different agent
- Never pass the full context.json to an agent that doesn't need all of it

**Soft constraints** (follow unless there's a documented reason not to):
- Prefer haiku model for summarization tasks (cost)
- Stop as soon as success criteria are met (don't over-explore)
- Use parallel tool calls when tools are independent (speed)
- Prefer jq + --json flags over parsing text output (reliability)

## Prompt Caching: Static-First Structure

All agent prompts must follow this order to maximize cache hits:
1. Static: role + rules (never changes)
2. Static: tool/MCP list (changes only on tool additions)
3. Semi-static: context.json field list (changes per phase, not per run)
4. Dynamic: actual context.json field values (changes every run)

Never embed dynamic data in the system prompt. Always pass it as the last thing.
Cache hits reduce input tokens to 10% of base price.

## DesignSystemAgent Condition

Only invoked if `spec.has_ui = true`. SupervisorAgent (command logic) checks this before spawning.

## MemoryAgent Trigger

Invoked automatically by command logic after each phase if `context.json` size > 6KB.
Compresses all completed blocks into `summary` field (≤500 chars) using haiku model.
