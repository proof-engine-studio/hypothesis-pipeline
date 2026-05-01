# Hypothesis Pipeline

> A multi-agent system for end-to-end product hypothesis testing — from idea to validated market signal — built on Claude Code.

11 specialized agents take a raw idea through structured framing, architecture, design, build, QA, deployment, paid acquisition, and measurement. Each agent has a narrow role, reads only the JSON fields it needs, and hands off via a shared structured context. Three human checkpoints protect against irreversible actions (Figma/Confluence creation, production deploy, ad spend).

```
/hypothesize → /spec ✓ → /arch + /design-system → /build → /qa → /ship ✓ → /promote ✓ → /measure
                  └────── parallel if has_ui ──────┘
```

## Why

Most hypothesis-testing fails in the gap between "I have an idea" and "I have a market signal." The work between those two states — spec, architecture, build, QA, deploy, ads — is mechanical enough to template, but contextual enough that templates alone don't help.

This pipeline turns it into a sequence of specialized agent invocations with structured handoffs and explicit human checkpoints where money or reputation is on the line.

## Quickstart

**Requirements:**
- [Claude Code CLI](https://docs.claude.com/claude-code) installed and authenticated
- Anthropic API access (Opus 4.7 + 1M context for foundation agents)
- `gh`, `git`, `kubectl`, `docker`, `sentry-cli` available locally
- MCP plugins for Linear, Figma, Slack, Atlassian, Intercom, Kubernetes, Chrome (configure via Claude Code plugin marketplace)

**Setup:**
```bash
git clone https://github.com/proof-engine-studio/hypothesis-pipeline.git
cd hypothesis-pipeline
claude
```

**Run:**
```
/hypothesize "Frontend devs will pay $30/month for AI-generated Storybook stories"
/spec                  # → CHECKPOINT: scope approval
/arch                  # ← Opus 1M (foundation)
/design-system         # ← Opus 1M (foundation, parallel with /arch if has_ui)
/build
/qa                    # ← blocks /ship if critical bugs
/ship staging          # → CHECKPOINT: deploy approval
/promote               # → CHECKPOINT: ad spend approval
/measure               # → VERDICT: validated | invalidated | inconclusive
```

State for each hypothesis lives in `state/{hypothesis-id}/context.json`. See [USAGE.md](USAGE.md) for the full practical guide.

## Architecture

### Agent Roster (11 agents)

| Agent | Model | Role |
|-------|-------|------|
| IdeationAgent | Opus 4.7 | Frames raw idea into a falsifiable hypothesis |
| SpecAgent | Sonnet 4.6 | Produces MVP spec; checkpoint before artifacts |
| **ArchAgent** | **Opus 4.7 (1M context)** | C4 diagrams, ADRs, threat model, OpenAPI |
| **DesignSystemAgent** | **Opus 4.7 (1M context)** | Tokens, components, accessibility (if has_ui) |
| BuilderAgent | Sonnet 4.6 | Implements MVP, generates Dockerfile + GHA + k8s |
| QAAgent | Sonnet 4.6 | Tests, coverage, security verification, E2E |
| DeployerAgent | Sonnet 4.6 | Triggers GHA deploy; checkpoint before push |
| MarketerAgent | Sonnet 4.6 | Ad campaigns + influencer brief; checkpoint before spend |
| AnalystAgent | Sonnet 4.6 | Verdict from Sentry + ads + Intercom data |
| SupervisorAgent | Sonnet 4.6 | Phase validation, MemoryAgent invocation, status |
| MemoryAgent | Haiku 4.5 | Selective context compression (≤500 chars) |

The "foundation tier" — ArchAgent and DesignSystemAgent — uses Opus 4.7 with 1M context because their decisions are the hardest to reverse downstream. They produce thorough, reasoned foundations that BuilderAgent can implement without architectural questions.

### Token Efficiency (~50–65% reduction)

Three layers compound:
1. **Field isolation** (35–50%): each agent reads ONLY its designated JSON fields, never the full context
2. **Selective compression via MemoryAgent on Haiku** (29%): triggered when context.json > 6KB; only stores decisions and outcomes, not process
3. **Phase scheduling** (18–20%): agents run sequentially; idle agents receive compressed summaries instead of full state

Plus prompt-caching-friendly structure: static system prompt → static tools → dynamic context. See [docs/research-insights.md](docs/research-insights.md) for sources and details.

### State Contract

All inter-agent communication goes through `state/{id}/context.json`, validated by [`schemas/handoff.schema.json`](schemas/handoff.schema.json). One block per phase. No direct agent-to-agent calls.

### Human Checkpoints (3)

| Phase | Why blocking | What's shown |
|-------|--------------|--------------|
| Spec | Figma/Confluence artifacts created | MVP scope, out-of-scope, target segment |
| Deploy | Live deployment with traffic | Exact commands, environment, image tag |
| Ad spend | Real money on campaigns | Platform, audience, budget, ad variants |

Agent halts and writes `state/{id}/checkpoint.json`. Human types `approved` or `revise: {feedback}` to proceed.

## File Structure

```
hypothesis-pipeline/
├── CLAUDE.md                      # Orchestration manifest (auto-loaded by Claude Code)
├── USAGE.md                       # Practical usage guide
├── README.md                      # This file
├── LICENSE                        # MIT
├── .claude/
│   ├── settings.json              # Project permissions
│   └── commands/                  # 10 slash commands
│       ├── hypothesize.md
│       ├── spec.md
│       ├── arch.md
│       ├── design-system.md
│       ├── build.md
│       ├── qa.md
│       ├── ship.md
│       ├── promote.md
│       ├── measure.md
│       └── status.md
├── .claude-plugin/
│   └── plugin.json                # Plugin metadata
├── agents/                        # 11 agent prompts
│   ├── hypothesis.md
│   ├── spec.md
│   ├── arch.md
│   ├── design-system.md
│   ├── builder.md
│   ├── qa.md
│   ├── deployer.md
│   ├── marketer.md
│   ├── analyst.md
│   ├── supervisor.md
│   └── memory.md
├── schemas/
│   ├── handoff.schema.json        # Canonical inter-agent data contract
│   └── checkpoint.schema.json
├── docs/
│   └── research-insights.md       # Patterns from Anthropic, arXiv, Simon Willison
└── state/                         # Pipeline state (gitignored except .gitkeep)
```

## Cost Estimate

Per hypothesis (rough):
- Foundation tier (Opus 1M for arch + design): $2.50–5.00
- Strategy (Opus for hypothesis): $0.50
- Implementation tier (Sonnet for build, qa, deploy, marketing, analysis): $1.50–3.50
- Compression (Haiku): $0.05–0.20

**Total: $4.50–9 per hypothesis** (plus your ad budget, typically $100–1000 per test).

10 hypotheses/month: ~$60–110 in agent costs.

## Research Behind the Design

Built on patterns from cutting-edge agentic AI research. Key references:
- [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Multi-agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic: Lessons from Claude Code — Prompt Caching](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything)
- [Simon Willison: Building Effective Agents](https://simonwillison.net/2024/Dec/20/building-effective-agents/)
- Multiple arXiv papers on selective memory, DAG execution, behavioral contracts, structured outputs

Full list with applied/not-applied notes in [docs/research-insights.md](docs/research-insights.md).

## Status

Initial public release. The pipeline structure is stable; individual agent prompts will evolve based on real-run feedback.

Known gaps (future work):
- Async streaming between phases (currently sequential)
- Auto-extraction of executable constraints from CLAUDE.md (ContextCov-style)
- Multi-hypothesis parallelism with cross-hypothesis memory

## Contributing

Issues and PRs welcome. Especially valuable: real-run reports (what verdict, what costs, what surprised you), agent prompt improvements, additional MCP integrations.

## License

MIT — see [LICENSE](LICENSE).
