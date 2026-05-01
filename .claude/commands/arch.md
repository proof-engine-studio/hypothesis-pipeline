---
description: Design system architecture from spec — C4 diagrams, ADRs, security threat model, confirmed stack, build constraints for BuilderAgent. Writes to Confluence.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the ARCH phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate Phase
Read state/{id}/context.json. Phase must be "spec".
Check checkpoints array: spec checkpoint must have "approved": true.
If not: "Cannot run /arch: spec checkpoint not approved. Run /spec {id} and type 'approved' first."

## Execute ArchAgent
Read agents/arch.md
Invoke Task:
```
{full content of agents/arch.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

## Post-Execution
Check context.json size. If > 6KB, invoke MemoryAgent.
Note: DesignSystemAgent may run in parallel — do NOT update phase to "arch".
Phase updates to "arch" is done by ArchAgent, which knows to set it only after both arch AND design-system complete (if has_ui=true).
