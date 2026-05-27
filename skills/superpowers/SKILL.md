---
name: superpowers
description: >
  Complete software development methodology for ToqanClaw: structured brainstorming, implementation planning, sub-agent-driven execution with two-stage code review, TDD, systematic debugging, and evidence-before-claims discipline. Use before any build, implement, fix, or ship request. Ported from Superpowers by obra — the battle-tested development workflow originally built for Claude Code.
---

# Superpowers for ToqanClaw

> The complete structured development workflow for ToqanClaw generators.


> **Note for sub-agent tasks:** If you were dispatched as a sub-agent to execute a specific task (not orchestrate), skip this skill and focus on your assigned work.

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST read and apply that skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## Instruction Priority

Skills override default behavior, but **user instructions always take precedence**:

1. **User's explicit instructions** (AGENTS.md, CONTEXT.md, direct requests) — highest priority
2. **Superpowers skills** — override default behavior where they conflict
3. **Default behavior** — lowest priority

If AGENTS.md says "don't use TDD" and a skill says "always use TDD," follow AGENTS.md.

## How to Access Skills

In ToqanClaw, skills are markdown guides under `/workspace/skills/`. Read them with `run_command` (`cat skills/<name>/<name>.md`) before taking action. No special tool needed — the files are directly readable.

**Before ANY action:** run `cat /workspace/SKILLS.md` to see available skills, then read the relevant ones.

**Check SKILLS.md first:** `cat /workspace/SKILLS.md`

**Read a sub-skill:** `cat /workspace/skills/superpowers/skills/<name>/<name>.md`

**Read supporting files:** `cat /workspace/skills/superpowers/skills/<name>/<file>.md`

## The Rule

**Check for and read relevant skills BEFORE any response or action.** Even a 1% chance a skill applies means you should read it. If you read a skill and it turns out not to apply, that is fine — move on.

## Workflow Overview

### For build / create / implement requests:

---

#### Phase 1 — Brainstorming

**Announce:** "I'm using the brainstorming skill."

1. **Explore project context** — read files, docs, recent commits before asking anything.
2. **Scope check** — if the request covers multiple independent subsystems, flag it immediately and help your human partner decompose into sub-projects. Each sub-project gets its own spec → plan → implementation cycle. Do not start refining details of an oversized project.
3. **Clarifying questions** — ask one at a time. Multiple-choice preferred. Focus on purpose, constraints, and success criteria.
4. **Propose 2–3 approaches** — with trade-offs and your recommendation. Lead with the recommended option and explain why.
5. **Present design in sections** — scale each section to its complexity. Ask "Does this look right so far?" after each section. Revise if needed.
6. **Write spec document** — save to `data/<slug>/specs/YYYY-MM-DD-<topic>-design.md` and commit.
7. **Spec self-review** (inline — do not dispatch):
   - Placeholder scan: any "TBD", "TODO", incomplete sections?
   - Internal consistency: do any sections contradict each other?
   - Scope check: focused enough for one plan, or needs decomposition?
   - Ambiguity check: any requirement interpretable two different ways? Pick one and make it explicit.
   - Fix issues inline. No need to re-review after fixing.

**HARD-GATE — User review:**
> "Spec written and committed to `<path>`. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."

Wait for your human partner's response. If changes are requested, make them and re-run the spec self-review loop. **Only proceed once your human partner explicitly approves.**

---

#### Phase 2 — Writing Plans

**Announce:** "I'm using the writing-plans skill to create the implementation plan."

1. **Scope check** — if the spec covers multiple independent subsystems that weren't split during brainstorming, suggest splitting into separate plans now. Each plan should produce working, testable software on its own.
2. **Map file structure first** — before defining tasks, list every file to be created or modified and what each one is responsible for. This locks in decomposition decisions. One clear responsibility per file.
3. **Write the plan header:**
   ```
   # [Feature Name] Implementation Plan

   > **For agentic workers:** Read the `subagent-driven-development` skill (recommended)
   > or the `executing-plans` skill before implementing this plan task-by-task.

   **Goal:** [one sentence]
   **Architecture:** [2–3 sentences]
   **Tech Stack:** [key technologies]
   ---
   ```
4. **Write bite-sized tasks** — each step is one action (2–5 minutes). TDD built in: write failing test → run it → implement minimal code → run it → commit. Every step has exact file paths, complete code, and exact commands with expected output. No placeholders, no "TBD", no "similar to Task N".
5. **Plan self-review** (inline — do not dispatch):
   - Spec coverage: skim each spec requirement — can you point to a task that implements it? List gaps and add tasks for any missing ones.
   - Placeholder scan: look for TBD, TODO, "add appropriate error handling", "write tests for the above", steps that describe without showing code.
   - Type consistency: do type names, method signatures, and property names used in later tasks match what was defined in earlier tasks?
   - Fix issues inline.
6. **Save plan** to `data/<slug>/plans/YYYY-MM-DD-<feature>.md`.

**HARD-GATE — Execution choice:**
> "Plan complete and saved to `data/<slug>/plans/<filename>.md`. Two execution options:
>
> **1. Subagent-Driven (recommended)** — Dispatch a fresh sub-agent task per task step, two-stage review between tasks (spec compliance first, then code quality), high quality and isolation.
>
> **2. Inline Execution** — Execute tasks in this session using the executing-plans skill, batch execution with checkpoints.
>
> Which approach?"

Wait for your human partner's choice before proceeding.

---

#### Phase 3a — Subagent-Driven Development (if chosen)

**Announce:** "I'm using the subagent-driven-development skill."

**Before starting:**
1. Read the plan file completely.
2. Extract ALL tasks with their full text and context upfront.
3. Note project context, tech stack, and dependencies.
4. Plan the task dispatch sequence.

**Per task — repeat until all tasks are done:**

1. **Dispatch implementer sub-agent** using `implementer-prompt.md` template.
   - Provide the FULL text of the task (do not make the sub-agent read the plan file).
   - Include scene-setting context: where this task fits in the overall plan.
   - Answer any sub-agent questions before allowing work to start.
2. **Wait for implementer result:**
   - `DONE` → proceed to spec compliance review.
   - `DONE_WITH_CONCERNS` → read concerns; if about correctness or scope, address before review; if observations only, proceed to review.
   - `NEEDS_CONTEXT` → provide missing context, re-dispatch.
   - `BLOCKED` → assess: context problem → provide context and re-dispatch; needs more reasoning → re-dispatch with more capable model; task too large → break it down; plan is wrong → escalate to your human partner.
3. **Dispatch spec compliance reviewer** using `spec-reviewer-prompt.md` template.
   - Provide task requirements and implementer's report.
4. **If spec review fails** → dispatch fix task → re-review. Repeat until spec review passes (✅). Do not proceed to code quality review until spec compliance is ✅.
5. **Dispatch code quality reviewer** using `code-quality-reviewer-prompt.md` template. Only dispatch AFTER spec compliance is ✅.
6. **If code quality fails** → dispatch fix task → re-review. Repeat until code quality passes (✅).
7. **Mark task complete.** Proceed to next task.

**Do not pause between tasks to check in with your human partner.** Execute all tasks continuously. The only reasons to stop are: a blocker you cannot resolve, ambiguity that genuinely prevents progress, or all tasks complete.

**After all tasks:**
1. Dispatch a final code reviewer task covering the entire implementation (use `requesting-code-review` skill — get git SHAs with `git rev-parse`).
2. Proceed to Phase 4.

---

#### Phase 3b — Inline Execution (if chosen)

**Announce:** "I'm using the executing-plans skill to implement this plan."

1. Read the plan file with `cat data/<slug>/plans/<plan-file>.md`.
2. **Review critically** — identify questions or concerns before starting. If concerns exist, raise them with your human partner before starting. If no concerns, proceed.
3. Read the `test-driven-development` skill before writing any code.
4. **Execute each task step by step** — follow each step exactly. Run verifications as specified. Do not skip steps.
5. **Stop immediately if blocked** — missing dependency, test fails unexpectedly, instruction unclear, or verification fails repeatedly. Ask your human partner for clarification rather than guessing.
6. After all tasks complete: proceed to Phase 4.

---

#### Phase 4 — Verification Before Completion

**The iron law:** No completion claims without fresh verification evidence.

Before claiming work is complete:

1. **Identify** — what command proves this claim?
2. **Run** — execute the full command fresh (do not rely on previous runs).
3. **Read** — full output, check exit code, count failures.
4. **Requirements checklist** — re-read the spec, create a line-by-line checklist, point to a task or commit for every spec requirement. Report any gaps.
5. **Only then** — make the claim, with evidence.

**Never:**
- Use "should", "probably", "seems to"
- Express satisfaction before verification ("Great!", "Perfect!", "Done!")
- Trust agent success reports without independently verifying VCS diff
- Rely on partial verification

---

#### Phase 5 — Finishing

**Announce:** "I'm using the finishing-a-development-branch skill to complete this work."

1. **Verify tests pass** — run the project's full test suite. If tests fail, stop and fix before presenting options.
2. **Detect environment:**
   ```bash
   git branch --show-current
   git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
   ```
3. **Present exactly these 4 options to your human partner:**
   > "Implementation complete. What would you like to do?
   >
   > 1. Merge back to `<base-branch>` locally
   > 2. Push and create a Pull Request
   > 3. Keep the branch as-is (I'll handle it later)
   > 4. Discard this work
   >
   > Which option?"
4. **Execute the chosen option:**
   - Option 1: `git checkout <base> && git pull && git merge <feature>` → verify tests → delete branch.
   - Option 2: `git push -u origin <feature>` → `gh pr create` with summary and test plan.
   - Option 3: Report branch name and that changes are committed.
   - Option 4: **Require typed "discard" confirmation** before `git branch -D <feature>`.

---

### For bug reports:

**Announce:** "I'm using the systematic-debugging skill."

**The iron law:** No fixes without root cause investigation first.

**Phase 1 — Root cause investigation (complete before any fix):**
1. Read error messages carefully — don't skip past errors or warnings.
2. Reproduce consistently — exact steps, every time. If not reproducible, gather more data.
3. Check recent changes — git diff, recent commits, new dependencies, config changes.
4. In multi-component systems: add diagnostic instrumentation at each component boundary (log inputs, log outputs, verify config propagation). Run once to gather evidence showing WHERE it breaks, then analyze.
5. Trace data flow — where does the bad value originate? Keep tracing up the call stack to the source.

**Phase 2 — Pattern analysis:**
1. Find working examples — locate similar working code in the codebase.
2. Compare against references — read reference implementations completely, every line.
3. Identify differences — list every difference between working and broken, however small.
4. Understand dependencies — config, environment, assumptions.

**Phase 3 — Hypothesis and testing:**
1. Form ONE hypothesis: "I think X is the root cause because Y." Write it down. Be specific.
2. Make the SMALLEST possible change to test it. One variable at a time.
3. If it worked → Phase 4. If it didn't → form NEW hypothesis. Do not stack fixes.

**Phase 4 — Implementation:**
1. Write a failing test that reproduces the bug (read `test-driven-development` skill — TDD cycle applies here too).
2. Implement ONE fix addressing the root cause.
3. Verify fix: test passes, no other tests broken, issue actually resolved.
4. **If fix doesn't work:**
   - Count fixes attempted.
   - If fewer than 3: return to Phase 1 with new information.
   - **If 3 or more fixes have failed: STOP. Question the architecture.** Each fix revealing a new problem in a different place signals an architectural issue, not a symptom. Discuss with your human partner before attempting any further fixes.

---

### For any implementation (within any phase):

**Read the `test-driven-development` skill before writing any production code.**

**The iron law:** No production code without a failing test first.

1. **RED** — Write one minimal failing test showing what should happen. Run it. Confirm it fails for the right reason (feature missing, not a typo). If the test passes immediately, you are testing existing behavior — fix the test.
2. **GREEN** — Write the simplest code that makes the test pass. Do not add features. Do not refactor other code.
3. **Verify GREEN** — Run the test. Confirm it passes. Confirm other tests still pass.
4. **REFACTOR** — Remove duplication, improve names, extract helpers. Keep tests green. Do not add behavior.
5. Repeat for the next failing test.

**If you wrote code before the test:** delete it. Start over. No exceptions.

---

### For code review:

**Requesting review:**
1. Read the `requesting-code-review` skill.
2. Get git SHAs: `BASE_SHA=$(git rev-parse HEAD~1)` and `HEAD_SHA=$(git rev-parse HEAD)`.
3. Dispatch a code reviewer sub-agent task using the `code-reviewer.md` template — provide description, plan/requirements, BASE_SHA, HEAD_SHA.
4. Act on feedback: fix Critical issues immediately; fix Important issues before proceeding; note Minor issues.

**Receiving review:**
1. Read the `receiving-code-review` skill.
2. Read ALL feedback completely before reacting.
3. If any item is unclear: stop, ask for clarification on all unclear items before implementing anything.
4. For each item: verify it is technically correct for this codebase before implementing. If reviewer is wrong, push back with technical reasoning.
5. Implement one item at a time, test each fix individually.
6. No performative agreement ("You're right!", "Great point!") — just fix and show in the code.

---

### For multiple independent problems:

**Read the `dispatching-parallel-agents` skill.**

Use when:
- 2 or more independent failures in different test files or subsystems.
- Each problem can be understood without context from others.
- No shared state between investigations (agents would not edit the same files).

**Do not use when** failures are related (fixing one might fix others), or agents would interfere with each other.

**Pattern:**
1. Group failures by independent domain.
2. Create one focused task per domain — specific scope, all context needed, clear expected output, constraints ("do not change other code").
3. Dispatch all tasks simultaneously (no dependencies between them).
4. When tasks return: review each summary, check for conflicts, run full test suite, integrate all changes.

---

## Red Flags

These thoughts mean STOP — you are rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "I can check files quickly" | Files lack task context. Check for skills. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Skills prevent this. |
| "I know what that means" | Knowing the concept ≠ using the skill. Read it. |

## Skill Priority

When multiple skills could apply:

1. **Process skills first** (brainstorming, debugging) — these determine HOW to approach the task
2. **Implementation skills second** — these guide execution

"Let's build X" → brainstorming first, then implementation skills.
"Fix this bug" → systematic-debugging first, then domain-specific skills.

## Skill Types

**Rigid** (TDD, debugging): Follow exactly. Don't adapt away from the discipline.

**Flexible** (patterns): Adapt principles to context.

The skill itself tells you which type it is.

## Sub-Agents in ToqanClaw

In ToqanClaw, sub-agents are implemented as tasks created via the task API (`create_task` with dependencies). See `skills/superpowers/skills/subagent-driven-development/subagent-driven-development.md` and `skills/superpowers/skills/dispatching-parallel-agents/dispatching-parallel-agents.md` for how to structure these.

The sandbox is already isolated — no worktree setup is needed.

## User Instructions

Instructions say WHAT, not HOW. "Add X" or "Fix Y" doesn't mean skip workflows. Always check for applicable skills.
