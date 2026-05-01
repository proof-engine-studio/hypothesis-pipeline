# Contributing

Thanks for considering a contribution. This pipeline is in early public release — improvements based on real-run experience are especially valuable.

## What We Want

**High value:**
- Real-run reports: what hypothesis you tested, what the verdict was, surprises encountered, total cost
- Agent prompt refinements based on observed failure modes
- New MCP integrations (e.g., Notion for spec, alternative ad platforms)
- Better E2E test patterns for QAAgent across stack types
- Cost optimizations that don't hurt output quality

**Welcome:**
- Documentation improvements
- Bug fixes in command/agent files
- Additional examples in `docs/`

**Probably not:**
- Adding more agents speculatively (the coordinate tax grows fast)
- Generic features without a concrete hypothesis-test use case
- Framework-style abstractions (we prefer concrete, simple files)

## How to Propose Changes

1. **Open an issue first** for non-trivial changes — quick discussion saves rework
2. **One concern per PR** — easier to review and merge
3. **Update relevant docs** — if you change an agent, update its description; if you change the pipeline shape, update CLAUDE.md and README.md

## Agent Prompt Conventions

Each agent file in `agents/` follows this structure:
1. YAML frontmatter (name, description, model, color)
2. Opening role sentence (one sentence)
3. Token Efficiency Rules
4. Input section (which JSON fields it reads)
5. Process steps
6. Self-Check
7. Output (what JSON block it writes + format)

Keep prompts under 250 lines. If yours grows larger, split the responsibility.

## Local Testing

```bash
git clone https://github.com/proof-engine-studio/hypothesis-pipeline.git
cd hypothesis-pipeline
claude
> /hypothesize "test idea"
```

Verify your changes don't break the basic flow before submitting.

## License

By contributing, you agree your contributions are licensed under the MIT License.
