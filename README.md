# Superpowers for ToqanClaw

> A port of [Superpowers](https://github.com/obra/superpowers) by [@obra](https://github.com/obra) — the complete structured development methodology, adapted for [ToqanClaw](https://toqan.ai).

Superpowers for ToqanClaw gives ToqanClaw generators a battle-tested development workflow built from composable skills. When active, generators automatically brainstorm before coding, write implementation plans before touching files, execute via sub-agents with two-stage code review, and verify before claiming completion.

## Quickstart

Install via the ToqanClaw skills system:

```bash
npx skills add feliksgh/superpowers-toqanclaw --skill superpowers --agent '*' --yes --copy
```

## How it works

Every build request flows through the same disciplined sequence:

1. **Brainstorm** — explore context, propose 2-3 approaches with trade-offs, write a spec, get human approval before touching any code.
2. **Write a plan** — map every file to be changed, write bite-sized tasks with exact code and commands, self-review for gaps.
3. **Execute** — either via sub-agents (each task dispatched as an isolated worker with two-stage review: spec compliance first, then code quality) or inline via the executing-plans skill.
4. **Verify** — fresh verification with actual command output before any completion claim. No "should work", no "seems fine".
5. **Finish** — run the full test suite, present merge/PR/keep/discard options, let the human decide.

Bug reports follow a parallel track: systematic root cause investigation before any fix attempt, with TDD throughout.

## The Workflow

1. **Brainstorming** — scope the problem, clarify requirements, propose approaches, write a spec, get sign-off
2. **Writing plans** — decompose the spec into bite-sized tasks, map file structure, self-review for completeness
3. **Subagent-driven development** — dispatch per-task sub-agents, two-stage review (spec compliance → code quality) between every task
4. **Executing plans** — inline alternative to subagent execution, with checkpoints and blockers escalated to the human
5. **Test-driven development** — red → green → refactor, no production code without a failing test first
6. **Requesting code review** — dispatch a dedicated reviewer sub-agent with git SHAs and requirements context
7. **Verification before completion** — identify the proof command, run it fresh, check exit codes, report evidence
8. **Finishing a development branch** — run full tests, present merge/PR/keep/discard options, execute the chosen path

## What's Inside

### Core Workflow

- **brainstorming** — Structured exploration: read context, scope the request, clarify one question at a time, propose 2-3 approaches with trade-offs, write and self-review a spec, hard-gate on human approval before proceeding
- **writing-plans** — Map all files first, write bite-sized tasks with exact code and commands, self-review for spec coverage and placeholders, present subagent vs inline execution choice
- **subagent-driven-development** — Dispatch isolated sub-agents per task, enforce two-stage review (spec compliance then code quality), handle DONE/BLOCKED/NEEDS_CONTEXT states, continue without interruption until all tasks complete
- **executing-plans** — Inline plan execution with step-by-step discipline, stop-on-blocker policy, full test suite after all tasks
- **finishing-a-development-branch** — Run full tests, detect base branch, present four merge options, execute chosen path

### Quality Gates

- **verification-before-completion** — Iron law: identify the proof command, run it fresh, read full output, check exit code, present evidence. No satisfaction before verification.
- **test-driven-development** — Strict red-green-refactor cycle, testing anti-patterns reference, no production code before a failing test
- **systematic-debugging** — Root cause before any fix: reproduce consistently, check recent changes, trace data flow, one hypothesis at a time, escalate to architecture discussion after 3 failed fixes

### Code Review

- **requesting-code-review** — Get git SHAs, dispatch a dedicated reviewer sub-agent using the code-reviewer template, act on feedback by severity (Critical → fix immediately, Important → fix before proceeding)
- **receiving-code-review** — Read all feedback before reacting, verify technical correctness before implementing, one fix at a time, no performative agreement

### Parallel Work

- **dispatching-parallel-agents** — Group independent failures by domain, dispatch all tasks simultaneously with full context, integrate results and run full suite after all return

## Philosophy

Discipline compounds. Each skill in this library exists because undisciplined shortcuts at that step reliably cause failures two steps later. Brainstorming without a spec leads to implementing the wrong thing. Planning without file mapping leads to architectural collisions. Claiming completion without verification leads to regressions going undetected.

The workflow is deliberately sequential and hard-gated. The gates exist because experienced developers know the failure modes. A generator that skips brainstorming to "save time" will spend more time correcting wrong assumptions. A generator that skips verification to "seem confident" will ship broken work.

Following the workflow does not slow you down. It front-loads the thinking that would otherwise happen as expensive rework.

## Credits

Superpowers for ToqanClaw is based on [Superpowers](https://github.com/obra/superpowers) by [@obra](https://github.com/obra), originally built for Claude Code. The core workflow philosophy, skill structure, and disciplined development methodology are his work. This is a port and adaptation for the ToqanClaw platform.

## License

MIT — same as the original Superpowers project.
