---
description: Implement the MVP from spec, arch, and design system. Scaffolds repo, implements only MVP scope, generates Dockerfile + GitHub Actions workflow + K8s manifests. Does NOT deploy.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task
---

You are executing the BUILD phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate Phase
Read state/{id}/context.json. Check:
1. `arch` block must exist (ArchAgent must have completed)
2. If spec.has_ui=true: `design_system` block must also exist
3. Phase should be "arch"

If arch block missing: "Cannot run /build: /arch {id} must complete first."
If design_system missing and has_ui=true: "Cannot run /build: /design-system {id} must complete first (running in parallel with /arch)."

## Execute BuilderAgent
Read agents/builder.md
Invoke Task:
```
{full content of agents/builder.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

## Post-Execution
After Task completes:
Check context.json size. If > 6KB, invoke MemoryAgent.
Print: "Build complete. Next: /qa {id} to verify before /ship."
