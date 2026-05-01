# Research Insights: Cutting-Edge Agentic AI Patterns

Sources: Anthropic Engineering Blog, Simon Willison, Lilian Weng, arXiv papers (2024-2025).
These patterns are incorporated into the agent designs in this pipeline.

---

## Incorporated Patterns

### 1. Prompt Caching: Static-First Ordering (Anthropic)
Any change invalidates the entire cache after that point.
**Required order**: static system prompt → static tools → project context → session context → dynamic messages.
Never modify tools mid-conversation — add state changes to message history instead.
**Result**: reduces input tokens to 10% of base price on cache hits.

Applied: all agent prompts are structured static (instructions + rules) → dynamic (context.json fields).

### 2. Chain-of-Thought Before JSON Output
Models that reason explicitly about schema mapping before outputting JSON achieve 62.41% accuracy vs naive approaches.
**Pattern**: "Here's the data → here's how it maps to fields → here's the JSON" (not just the JSON).

Applied: each agent has a reasoning step before writing to context.json.

### 3. Sub-agent Context Isolation (~70% context reduction)
Specialized agents returning 1k-2k token summaries to coordinator outperform monolithic agents with massive contexts.

Applied: each agent reads ONLY its designated fields, writes ONLY its block.

### 4. Parallel Tool Execution (~90% research time reduction)
Running 3+ tools in parallel in a single agent turn (web search, Intercom, Linear simultaneously) vs serial calls.

Applied: IdeationAgent runs market research + customer signal in parallel.

### 5. Selective Memory for Parallel Teams (55% runtime reduction)
Don't share all intermediate results — use a lightweight controller deciding what merits cross-team sharing.

Applied: MemoryAgent uses selective compression; only decisions and outcomes, not process details.

### 6. DAG-Based Parallel Execution (35% fewer steps)
Model task dependencies as a directed acyclic graph. Independent tasks run in parallel; results inform which subsequent tasks to trigger.

Applied: /arch and /design-system run in parallel after /spec; /build waits for both.

### 7. Proactive Error Correction (29.68% token reduction)
A meta-agent detecting errors in real-time prevents cascading failures without full re-execution.

Applied: SupervisorAgent does pre-flight validation before each phase starts.

### 8. Agent Self-Assessment
Agents with access to their own output and rubrics for self-evaluation produce better results.

Applied: each agent has a self-check step before writing final output.

### 9. Behavioral Contracts: (p, δ, k)-satisfaction
Probabilistic safety: probability threshold p, soft deviation δ, recovery window k steps.
Hard constraints (never do X) + soft constraints (prefer Y, recover if Z).

Applied: CLAUDE.md specifies hard constraints (never deploy without approval) and soft constraints (prefer haiku for summarization, stop early when done).

### 10. Simplicity First (Simon Willison)
"Find the simplest solution possible, and only increasing complexity when needed."
For structured pipelines (known hypothesis space), workflows beat autonomous agents.

Applied: pipeline is a structured workflow with explicit phases; agents are specialized, not general-purpose.

---

## Patterns Not Yet Implemented (Future Work)

### Asynchronous Agent Execution
Anthropic research: streaming results as they arrive (don't wait for batch completion) could enable 10x parallelism.
Current: agents run sequentially within each phase. Future: stream arch + design-system results to builder as they arrive.

### LAMaS Layer-wise Parallelism
Remove intra-layer dependencies entirely; each agent consumes from the previous layer directly.
Currently blocked by checkpoint requirements and sequential phase validation.

### Agent-as-Judge Hypothesis Validation
Give agents tools to score their own output against structured rubrics before submitting.
Would improve output quality at the cost of 10-20% more tokens per agent turn.

### Constraint Coverage (ContextCov)
Auto-extract executable constraints from CLAUDE.md and agent prompts.
Detect violations at runtime rather than only at development time.

---

## Sources

1. [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) — Anthropic
2. [Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic
3. [Multi-agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system) — Anthropic
4. [Lessons from Claude Code: Prompt Caching](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything) — Anthropic
5. [Stop Wasting Tokens](https://arxiv.org/html/2510.26585v1) — SupervisorAgent pattern
6. [Selective Memory for Parallel Agents](https://arxiv.org/html/2602.05965) — 55% runtime reduction
7. [Flash-Searcher: DAG Execution](https://arxiv.org/html/2509.25301v1) — 35% fewer steps
8. [LAMaS: Layer-wise Parallelism](https://arxiv.org/html/2601.10560) — latency-aware orchestration
9. [Think Inside the JSON](https://arxiv.org/html/2502.14905v1) — CoT before structured output
10. [Agent Behavioral Contracts](https://arxiv.org/html/2602.22302v1) — probabilistic safety
11. [Simon Willison on Building Effective Agents](https://simonwillison.net/2024/Dec/20/building-effective-agents/) — simplicity first
12. [Towards a Science of Scaling Agent Systems](https://arxiv.org/html/2512.08296v1) — coordinate tax
