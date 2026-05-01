---
description: Generate product spec from active hypothesis. Defines MVP scope, prototype type, stack, and marketing channels. Creates Figma brief and Confluence page. Requires human approval.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the SPEC phase of the hypothesis pipeline.

## Resolve Hypothesis ID
If "$ARGUMENTS" is empty, find most recent:
```bash
ls state/ | grep "^hyp-" | sort | tail -1
```
Otherwise use $ARGUMENTS as the ID.

## Validate Phase
Read state/{id}/context.json, check "phase" field.
If phase is NOT "hypothesis": print "Cannot run /spec: pipeline is at phase '{phase}'. Run /hypothesize first." and stop.

## Execute SpecAgent
Read agents/spec.md
Invoke Task with:
```
{full content of agents/spec.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

The SpecAgent will handle the CHECKPOINT internally. It will halt and wait for your input.
Type 'approved' or 'revise: {feedback}' when prompted.

## Post-Execution
After approval, check context.json size. If > 6KB, invoke MemoryAgent.
Print next steps based on spec.has_ui value from updated context.json.
