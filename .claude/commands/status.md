---
description: Show current pipeline state for one or all active hypotheses. Displays phase progress, checkpoint status, context size, and Linear issue link.
argument-hint: [hypothesis-id]
allowed-tools: Read, Bash
---

You are displaying pipeline status. Read-only operation.

## Resolve Target
If "$ARGUMENTS" is a hypothesis ID: show status for that ID only.
If "$ARGUMENTS" is empty: show status for all active hypotheses.

## For Each Hypothesis

```bash
ls state/ | grep "^hyp-"
```

For each ID, read state/{id}/context.json and print:

```
┌─────────────────────────────────────────────────────────────┐
│ {id}                                                        │
│ Phase: {phase}           Created: {created_at}              │
├─────────────────────────────────────────────────────────────┤
│ Hypothesis: {hypothesis.statement or "pending"}             │
│ Target: {hypothesis.target_segment or "-"}                  │
│ Success metric: {hypothesis.success_metric or "-"}          │
│ Confidence: {hypothesis.confidence or "-"}                  │
├─────────────────────────────────────────────────────────────┤
│ Completed phases:                                           │
│  ✓ hypothesis   {if exists: "confidence={confidence}"}      │
│  ✓ spec         {if exists: "{product_name}, {scope count} items"} │
│  ✓ arch         {if exists: "stack={stack}"}                │
│  ✓ design-sys   {if exists: "{color_primary} / {font}"}     │
│  ✓ build        {if exists: "repo={repo_path}"}             │
│  ✓ qa           {if exists: "{verdict}: {tests_passed}/{tests_run}"} │
│  ✓ deploy       {if exists: "{environment} @ {url}"}        │
│  ✓ marketing    {if exists: "{platforms}, ${total_budget}"}  │
│  ✓ analysis     {if exists: "VERDICT: {verdict}"}           │
├─────────────────────────────────────────────────────────────┤
│ Checkpoints:                                                │
│  spec:     {approved: ✓ approved | ✗ pending}               │
│  deploy:   {approved: ✓ approved | ✗ pending}               │
│  marketing:{approved: ✓ approved | ✗ pending}               │
├─────────────────────────────────────────────────────────────┤
│ Context: {file_size}KB {if summary: "(compressed)"}         │
│ Linear: {linear_issue_id or "not created"}                  │
└─────────────────────────────────────────────────────────────┘
```

## Summary Line (when showing multiple hypotheses)
After all blocks: "Active hypotheses: {count}. Phases: hypothesis({n}), spec({n}), build({n}), deployed({n}), measuring({n}), complete({n})."

## Next Action Hint
Based on current phase, suggest the next command:
- hypothesis → "Run /spec {id}"
- spec → "Run /arch {id} and /design-system {id} in parallel"
- arch → "Run /build {id} (after design-system completes if has_ui=true)"
- build → "Run /qa {id} to verify the build"
- qa → "Run /ship {id} [staging|production] (if QA verdict is pass/conditional_pass)"
- deploy → "Run /promote {id}"
- marketing → "Run /measure {id} after campaign measurement window closes"
- analysis → "Review verdict in state/{id}/metrics.json"
