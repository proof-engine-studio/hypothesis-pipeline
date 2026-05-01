---
description: Create ad campaigns and influencer briefs to test the hypothesis in market. Supports Meta, Google, Telegram, LinkedIn + influencer platforms. Requires human approval before any spend.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the PROMOTE phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate Phase
Read state/{id}/context.json. Check:
1. `deploy` block must exist
2. deploy.url must be set (non-empty)
3. deploy checkpoint must be approved
4. Phase must be "deploy"

If deploy.url is empty: "Cannot run /promote: deployment URL not set. Run /ship {id} first."

## Execute MarketerAgent
Read agents/marketer.md
Invoke Task:
```
{full content of agents/marketer.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

The MarketerAgent will handle the CHECKPOINT internally.
Type 'approved', 'revise: {feedback}', or 'skip-{platform}' when prompted.

## Post-Execution
Check context.json size. If > 6KB, invoke MemoryAgent.
