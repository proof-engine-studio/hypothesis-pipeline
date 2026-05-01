---
description: Start a new hypothesis validation pipeline. Generates a structured, falsifiable hypothesis from your idea using market research and customer signals.
argument-hint: <idea or problem statement>
allowed-tools: Read, Write, Bash, Task
---

You are executing the HYPOTHESIZE phase of the hypothesis pipeline.

## Setup
Generate a new hypothesis ID:
```bash
COUNTER=$(ls state/ | grep "^hyp-" | wc -l)
HYP_ID="hyp-$(date +%Y%m%d)-$(printf '%03d' $((COUNTER + 1)))"
echo $HYP_ID
mkdir -p state/$HYP_ID
```

Initialize context.json:
```bash
cat > state/$HYP_ID/context.json << EOF
{
  "id": "$HYP_ID",
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "phase": "new"
}
EOF
```

## Execute IdeationAgent
Read the agent prompt from:
agents/hypothesis.md

Invoke Task with the following prompt (replace {HYPOTHESIS_ID} with the generated ID and {ARGUMENTS} with "$ARGUMENTS"):

```
{full content of agents/hypothesis.md}

HYPOTHESIS_ID: {generated id}
Arguments: $ARGUMENTS
Working directory: .
```

## Post-Execution
After Task completes, check if context.json size > 6KB. If so, invoke MemoryAgent:
Read agents/memory.md and invoke Task with it.

The IdeationAgent will print the next step. If it doesn't, remind the user:
"Run: /spec {HYP_ID}"
