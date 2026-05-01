---
description: Collect metrics from all sources and validate the hypothesis. Reads Sentry errors, ad platform performance, Intercom customer signal. Delivers verdict: validated / invalidated / inconclusive.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the MEASURE phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate Phase
Read state/{id}/context.json. Check:
1. `marketing` block must exist
2. marketing.approved_by must be set
3. Phase must be "marketing"

If not: print "Cannot run /measure: campaigns must be active. Run /promote {id} first."

## Execute AnalystAgent
Read agents/analyst.md
Invoke Task:
```
{full content of agents/analyst.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

## Post-Execution
Check context.json size. If > 6KB, invoke MemoryAgent.

After analysis completes, suggest next action based on verdict:
- validated: "Consider running /hypothesize with a more specific follow-up hypothesis."
- invalidated: "This hypothesis is invalidated. Document learnings. Run /hypothesize with a pivot."
- inconclusive: "Collect more data or adjust campaign targeting. Re-run /measure in {measurement_window} days."
