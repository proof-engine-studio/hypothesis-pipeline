---
description: Verify BuilderAgent's output — run tests, check feature coverage against spec, validate security mitigations, run E2E browser tests if has_ui. Blocks /ship on critical bugs.
argument-hint: [hypothesis-id]
allowed-tools: Read, Write, Bash, Task
---

You are executing the QA phase of the hypothesis pipeline.

## Resolve ID
If "$ARGUMENTS" empty: `ls state/ | grep "^hyp-" | sort | tail -1`
Otherwise use $ARGUMENTS.

## Validate Phase
Read state/{id}/context.json. Check:
1. `build` block must exist
2. build.repo_path directory must exist on disk
3. Phase must be "build"

If any check fails: print specific failure and stop.

## Execute QAAgent
Read agents/qa.md
Invoke Task:
```
{full content of agents/qa.md}

HYPOTHESIS_ID: {id}
Working directory: .
```

## Post-Execution
Check context.json size. If > 6KB, invoke MemoryAgent.
The QAAgent will print the next step based on verdict (ship, or fix_and_rebuild).
