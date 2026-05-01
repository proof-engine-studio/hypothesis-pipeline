# Usage Guide

## Approach 1 (recommended): Hub directory

This workspace itself is your "shop floor" for hypothesis testing. Don't copy it elsewhere — work directly from here.

```bash
git clone https://github.com/proof-engine-studio/hypothesis-pipeline.git
cd hypothesis-pipeline
claude
```

Then in Claude Code:
```
/hypothesize "Small business will pay $50/month for automated reporting"
```

What happens:
1. `state/hyp-20260501-001/context.json` is created with the hypothesis
2. After `/spec` — `state/hyp-20260501-001/{figma_url, confluence_url}`
3. After `/build` — actual code in `state/hyp-20260501-001/repo/`
4. Each hypothesis = its own subdirectory. History is preserved forever.

**When a prototype takes off:** move `state/{id}/repo/` to a separate repository and continue development there as a normal project. The pipeline has done its job.

```bash
cp -r state/hyp-20260501-001/repo ~/workspace/my-real-product
cd ~/workspace/my-real-product
git init && git remote add origin git@github.com:user/my-real-product.git
```

## Approach 2: Install as Claude Code plugin

Then `/hypothesize`, `/spec`, etc. work from ANY directory.

```bash
# From anywhere
claude
> /plugin install /path/to/hypothesis-pipeline
```

Trade-off: state directory still lives in one place — you'll need to either configure an env var for the state path, or state gets created in CWD each time. More complex to maintain for a solo user.

## Approach 3 (NOT recommended): Copy into each project

Copy `agents/`, `.claude/`, `schemas/` into every new project. Downsides:
- Lose unified hypothesis history
- Duplication when updating agents
- Hard to compare past/current hypotheses

---

## One-time setup

### 1. MCP authentication
These plugins are typically pre-installed but require auth:
```
> /plugin auth design-figma
> /plugin auth design-linear
> /plugin auth design-atlassian
> /plugin auth design-slack
> /plugin auth design-intercom
```

### 2. Sentry CLI
```bash
brew install getsentry/tools/sentry-cli  # if not installed
sentry login
```

### 3. GitHub CLI and kubeconfig
```bash
gh auth status        # must be authenticated
kubectl config view   # cluster context must be configured
```

### 4. Opus 1M access
ArchAgent and DesignSystemAgent use `claude-opus-4-7` with 1M context. Make sure you have:
- Opus 4.7 access enabled in Anthropic Console
- 1M context tier activated (if it requires separate activation)

---

## Typical session

```bash
cd hypothesis-pipeline
claude

# 1. Hypothesis (1-2 minutes, Opus reasoning)
> /hypothesize "Frontend developers will pay $30/month for AI-generated Storybook stories from components"

# 2. Spec (1-2 minutes + your 'approved')
> /spec
# [CHECKPOINT appears — review scope, type 'approved']

# 3. Architecture and design system in parallel (Opus 1M, 3-5 minutes each)
> /arch
> /design-system

# 4. Build (5-15 minutes)
> /build

# 5. QA verification (2-5 minutes) — blocks deploy on critical bugs
> /qa
# [VERDICT: pass / conditional_pass / fail]

# 6. Deploy (with your 'approved' before git push)
> /ship staging
# [CHECKPOINT — type 'approved']

# 7. Marketing (with your 'approved' before spending budget)
> /promote
# [CHECKPOINT — review campaigns, type 'approved']

# 8. Wait 7 days while ads run
# 9. Measure results
> /measure

# Get a VERDICT: validated / invalidated / inconclusive
```

At any time:
```
> /status              # all active hypotheses
> /status hyp-20260501-001   # one specific
```

---

## Cost per cycle (rough)

For an average hypothesis:
- IdeationAgent (Opus): ~$0.50
- SpecAgent (Sonnet): ~$0.10
- ArchAgent (Opus 1M): ~$1.50-3.00 (depending on depth of reference materials)
- DesignSystemAgent (Opus 1M): ~$1.00-2.00 (only if has_ui=true)
- BuilderAgent (Sonnet): ~$0.50-2.00 (depending on MVP size)
- QAAgent (Sonnet): ~$0.30-0.80 (depending on E2E test count)
- DeployerAgent (Sonnet): ~$0.20
- MarketerAgent (Sonnet): ~$0.30
- AnalystAgent (Sonnet): ~$0.30
- MemoryAgent (Haiku): ~$0.01-0.05 per compression

**Total:** ~$4-8 per hypothesis with UI, ~$2-4 without UI. Plus $100-1000 for ad spend.

At 10 hypotheses/month: ~$50-100 in agent costs + ad budgets.

---

## What NOT to do

- Don't manually edit `state/{id}/context.json` — the format is critical for agents
- Don't skip pipeline order (`/build` without `/arch` will error out)
- Don't skip checkpoints — they protect against irreversible spend
- Don't copy agents into every project — you lose history and version control
