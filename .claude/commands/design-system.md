---
description: Design visual language for the MVP UI — color palette, typography, spacing, component list. Only runs if spec.has_ui=true. Parallel with /arch.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the DESIGN-SYSTEM phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate
Read state/{id}/context.json.
Check: phase must be "spec" AND spec.has_ui must be true AND spec checkpoint approved.
If spec.has_ui is false: print "Design system not needed: spec.has_ui=false. Skip to /build after /arch completes."
If phase is wrong or checkpoint not approved: print appropriate message and stop.

## Execute DesignSystemAgent
Read agents/design-system.md
Invoke Task:
```
{full content of agents/design-system.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

## Post-Execution
Note: run in parallel with /arch. Both must complete before /build.
Check context.json size. If > 6KB, invoke MemoryAgent.
