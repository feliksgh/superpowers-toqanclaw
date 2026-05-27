---
name: executing-plans
description: Use when you have a written implementation plan to execute inline (in the current session) with review checkpoints. Alternative to subagent-driven-development when sub-agent tasks are not available or practical.
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Note:** ToqanClaw supports sub-agent tasks via task dependencies. For maximum quality, use `subagent-driven-development` instead — it gives fresh context per task and two-stage review. Use this skill when inline execution is preferred.

## The Process

### Step 1: Load and Review Plan

1. Read plan file with `run_command`: `cat data/<project-slug>/plans/<plan-file>.md`
2. Review critically — identify any questions or concerns about the plan
3. If concerns: Raise them with your human partner before starting
4. If no concerns: Proceed to execution

### Step 2: Execute Tasks

For each task:
1. Note the task as in progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Note as completed

Read the `test-driven-development` skill before writing any code.

### Step 3: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- Read `skills/superpowers/skills/finishing-a-development-branch/finishing-a-development-branch.md`
- Follow that skill to verify tests, present options, execute choice

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask your human partner for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Your human partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** — stop and ask.

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Read relevant skills when plan says to
- Stop when blocked, don't guess
- Never start implementation on main/master branch without explicit user consent
- The sandbox is already isolated — no worktree setup needed

## Integration

**Required workflow skills:**
- `writing-plans` — Creates the plan this skill executes
- `finishing-a-development-branch` — Complete development after all tasks

**Alternative:**
- `subagent-driven-development` — Higher quality via sub-agent tasks with review gates
