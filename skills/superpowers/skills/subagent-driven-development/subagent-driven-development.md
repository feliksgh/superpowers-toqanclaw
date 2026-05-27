---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks. Dispatches a fresh sub-agent task per plan step, with two-stage review after each task (spec compliance first, then code quality). Preferred over executing-plans for quality and isolation.
---

# Subagent-Driven Development

Execute a plan by dispatching a fresh sub-agent task per step, with two-stage review after each: spec compliance review first, then code quality review.

**Why sub-agents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh sub-agent task per step + two-stage review (spec then quality) = high quality, fast iteration

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are: a blocker you cannot resolve, ambiguity that genuinely prevents progress, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

## When to Use

**Use when:**
- You have a written implementation plan
- Tasks are mostly independent (completing one doesn't change the requirements for another)
- You want maximum quality with review gates between tasks

**vs. Executing Plans:**
- Same session (no context switch)
- Fresh sub-agent context per task (no context pollution)
- Two-stage review after each task: spec compliance first, then code quality
- Faster iteration (no human-in-loop between tasks)

**In ToqanClaw:** Sub-agents are tasks created via the task API. Each implementation task, spec review, and code quality review is a separate task with appropriate dependencies. The sandbox is already isolated — no worktree setup needed.

## The Process

### Before Starting

1. Read the plan file completely
2. Extract ALL tasks with their full text and context
3. Note the project context, tech stack, and dependencies
4. Plan your task dispatch sequence

### Per Task

For each task in the plan:

1. **Dispatch implementer task** — use the template at `implementer-prompt.md`
   - Provide the FULL text of the task (don't make the sub-agent read the plan file)
   - Include scene-setting context about where this task fits
2. **Wait for implementer result**
   - If DONE → proceed to spec review
   - If DONE_WITH_CONCERNS → read concerns, address before review if needed
   - If NEEDS_CONTEXT → provide missing context and re-dispatch
   - If BLOCKED → assess and resolve (see Handling Implementer Status)
3. **Dispatch spec reviewer task** — use the template at `spec-reviewer-prompt.md`
   - Provide the task requirements and implementer's report
4. **If spec review fails** → dispatch fix task, then re-review (repeat until ✅)
5. **Dispatch code quality reviewer task** — use the template at `code-quality-reviewer-prompt.md`
   - **Only dispatch AFTER spec compliance is ✅**
6. **If code quality fails** → dispatch fix task, then re-review (repeat until ✅)
7. **Mark task complete**, proceed to next task

### After All Tasks

1. Dispatch a final code reviewer task covering the entire implementation
2. Read `finishing-a-development-branch` skill and follow it

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

**Mechanical implementation tasks** (isolated functions, clear specs, 1-2 files): use a fast, cheap model.

**Integration and judgment tasks** (multi-file coordination, pattern matching, debugging): use a standard model.

**Architecture, design, and review tasks**: use the most capable available model.

**Task complexity signals:**
- Touches 1-2 files with a complete spec → cheap model
- Touches multiple files with integration concerns → standard model
- Requires design judgment or broad codebase understanding → most capable model

## Handling Implementer Status

**DONE:** Proceed to spec compliance review.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If the concerns are about correctness or scope, address them before review. If they're observations (e.g., "this file is getting large"), note them and proceed to review.

**NEEDS_CONTEXT:** The implementer needs information that wasn't provided. Provide the missing context and re-dispatch.

**BLOCKED:** The implementer cannot complete the task. Assess the blocker:
1. If it's a context problem, provide more context and re-dispatch with the same model
2. If the task requires more reasoning, re-dispatch with a more capable model
3. If the task is too large, break it into smaller pieces
4. If the plan itself is wrong, escalate to your human partner

**Never** ignore an escalation or force the same model to retry without changes. If the implementer said it's stuck, something needs to change.

## Prompt Templates

- `implementer-prompt.md` — Dispatch implementer sub-agent task
- `spec-reviewer-prompt.md` — Dispatch spec compliance reviewer task
- `code-quality-reviewer-prompt.md` — Dispatch code quality reviewer task

## Example Workflow

```
[Read plan file: data/my-project/plans/feature-plan.md]
[Extract all 5 tasks with full text and context]

Task 1: Hook installation script

[Dispatch implementer task with full task text + context]

Implementer result:
  - Implemented install-hook command
  - Added tests, 5/5 passing
  - Self-review: Found I missed --force flag, added it
  - Committed
  - Status: DONE

[Dispatch spec compliance reviewer task]
Spec reviewer: ✅ Spec compliant - all requirements met, nothing extra

[Get git SHAs, dispatch code quality reviewer task]
Code reviewer: Strengths: Good test coverage, clean. Issues: None. Approved.

[Mark Task 1 complete]

Task 2: Recovery modes

[Dispatch implementer task]

Implementer result:
  - Added verify/repair modes
  - 8/8 tests passing
  - Status: DONE

[Dispatch spec compliance reviewer task]
Spec reviewer: ❌ Issues:
  - Missing: Progress reporting (spec says "report every 100 items")
  - Extra: Added --json flag (not requested)

[Dispatch fix task]
Implementer: Removed --json flag, added progress reporting

[Spec reviewer re-reviews]
Spec reviewer: ✅ Spec compliant now

[Dispatch code quality reviewer task]
Code reviewer: Strengths: Solid. Issues (Important): Magic number (100)

[Dispatch fix task]
Implementer: Extracted PROGRESS_INTERVAL constant

[Code reviewer re-reviews]
Code reviewer: ✅ Approved

[Mark Task 2 complete]

...

[After all tasks: dispatch final code-reviewer task]
Final reviewer: All requirements met, ready to merge

[Read finishing-a-development-branch skill]
```

## Red Flags

**Never:**
- Start implementation on main/master branch without explicit user consent
- Skip reviews (spec compliance OR code quality)
- Proceed with unfixed issues
- Dispatch multiple implementation sub-agents in parallel (conflicts)
- Make sub-agent read the plan file (provide full text instead)
- Skip scene-setting context (sub-agent needs to understand where task fits)
- Ignore sub-agent questions (answer before letting them proceed)
- Accept "close enough" on spec compliance (spec reviewer found issues = not done)
- Skip review loops (reviewer found issues = implementer fixes = review again)
- Let implementer self-review replace actual review (both are needed)
- **Start code quality review before spec compliance is ✅** (wrong order)
- Move to next task while either review has open issues

**If sub-agent asks questions:**
- Answer clearly and completely
- Provide additional context if needed
- Don't rush them into implementation

**If reviewer finds issues:**
- Implementer (new fix task) fixes them
- Reviewer reviews again
- Repeat until approved
- Don't skip the re-review

**If sub-agent fails task:**
- Dispatch a fix task with specific instructions about what failed
- Don't try to fix it manually in the controller (context pollution)
- The fix task gets fresh context with a clear, narrow scope

## Integration

**Required workflow skills:**
- `writing-plans` — Creates the plan this skill executes
- `requesting-code-review` — Code review template for reviewer sub-agent tasks
- `finishing-a-development-branch` — Complete development after all tasks

**Sub-agent tasks should use:**
- `test-driven-development` — Follow TDD for each task
