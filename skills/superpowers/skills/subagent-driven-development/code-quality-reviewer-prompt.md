# Code Quality Reviewer Task Template

Use this template when dispatching a code quality reviewer task in ToqanClaw.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes (✅).**

```
Task description: "Code quality review for Task N: [task summary]"

Task prompt:
  Use the template at skills/superpowers/skills/requesting-code-review/code-reviewer.md

  Fill in the placeholders:
    {DESCRIPTION}: [task summary, from implementer's report]
    {PLAN_OR_REQUIREMENTS}: Task N from data/<project-slug>/plans/<plan-file>
    {BASE_SHA}: [commit SHA before this task started]
    {HEAD_SHA}: [current HEAD commit SHA]

  Get the SHAs with:
    git log --oneline -10
```

**In addition to standard code quality concerns, the reviewer should check:**
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the file structure from the plan?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment
