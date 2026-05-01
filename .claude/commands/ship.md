---
description: Deploy the MVP to staging or production via GitHub Actions. Shows exact plan, requires human approval, then triggers CI/CD and monitors until URL is live.
argument-hint: [hypothesis-id] [staging|production]
allowed-tools: Read, Write, Bash, Task
---

You are executing the SHIP (deploy) phase of the hypothesis pipeline.

## Resolve ID and Environment
Parse "$ARGUMENTS":
- First token matching "hyp-*": hypothesis ID. If missing, find most recent.
- Token "production" or "staging": environment. Default: staging.

## Validate Phase
Read state/{id}/context.json. Check:
1. `build` block must exist
2. `qa` block must exist with verdict in ["pass", "conditional_pass"]
3. build.dockerfile_path file must exist on disk
4. build.github_workflow_path file must exist on disk
5. Phase must be "qa" (or "build" if user explicitly skips QA — discouraged)

If qa block missing: print "Cannot run /ship: QA not run. Run /qa {id} first." and stop.
If qa.verdict = "fail": print "QA verdict is FAIL. Critical bugs:\n{list critical_bugs}\nFix in {build.repo_path}, re-run /build, then /qa." and stop unless user passes "--force-deploy".
If any other check fails: print specific failure and stop.

## Execute DeployerAgent
Read agents/deployer.md
Invoke Task:
```
{full content of agents/deployer.md}

HYPOTHESIS_ID: {id}
ENVIRONMENT: {staging|production}
Working directory: .
```

The DeployerAgent will handle the CHECKPOINT internally.
Type 'approved' or 'abort' when the agent presents the deployment plan.

## Post-Execution
Check context.json size. If > 6KB, invoke MemoryAgent.
