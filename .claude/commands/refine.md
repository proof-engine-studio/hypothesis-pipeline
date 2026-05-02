---
description: Conversational thinking partner — refines a raw idea into a sharp, falsifiable hypothesis before you run /hypothesize. Independent of the pipeline. Outputs a copy-pasteable hypothesis statement, no JSON, no Linear issues.
argument-hint: <rough idea or problem statement>
allowed-tools: Read, Bash, Task
---

You are launching a hypothesis-refinement conversation.

This is NOT part of the pipeline. The user wants to think out loud and sharpen a rough idea before committing to /hypothesize. No state files, no external artifacts, just dialogue.

## Execute

Read agents/hypothesis-coach.md and use it as the prompt for this conversation.

You may invoke it via Task for context isolation, OR run the coach directly in this session if the user wants a back-and-forth dialogue with you. **Direct conversation is preferred** for refine — the user wants iteration, and Task tool returns one shot.

If running directly: adopt the coach persona from agents/hypothesis-coach.md. The user's rough idea: $ARGUMENTS

If $ARGUMENTS is empty: open with "What's the idea? Give me one or two sentences and I'll start asking questions."

## Optional Context

If state/ has prior hypotheses, you may quickly scan them with:
```bash
ls state/ | grep "^hyp-" | head -10
```
Use prior hypothesis statements + verdicts as pattern reference, not as instructions.

## End State

When the user's hypothesis is sharp enough, the coach prints the final statement and tells them to run `/hypothesize "<statement>"`. That's the exit point. No state is written here.
