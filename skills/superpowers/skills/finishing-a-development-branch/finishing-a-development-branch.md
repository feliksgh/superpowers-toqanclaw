---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work. Guides completion of development work by presenting structured options for merge, PR, or cleanup. Always run after subagent-driven-development or executing-plans completes.
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Detect environment → Present options → Execute choice → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

**Note on isolation:** The ToqanClaw sandbox is already isolated — no worktree setup is needed. If you're on a feature branch, treat this as a normal git repo workflow.

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test
# or: cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Detect Environment

**Determine workspace state before presenting options:**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
git branch --show-current
```

This determines which menu to show:

| State | Menu | Notes |
|-------|------|-------|
| `GIT_DIR == GIT_COMMON` (normal repo) | Standard 4 options | Most common case in ToqanClaw |
| `GIT_DIR != GIT_COMMON`, named branch | Standard 4 options | On a worktree/feature branch |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Reduced 3 options (no local merge) | Externally managed workspace |

### Step 3: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or confirm with your human partner: "This branch split from main — is that correct?"

### Step 4: Present Options

**Present exactly these 4 options:**

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**Don't add explanation** — keep options concise.

**Detached HEAD — present exactly these 3 options:**

```
Implementation complete. You're on a detached HEAD (externally managed workspace).

1. Push as new branch and create a Pull Request
2. Keep as-is (I'll handle it later)
3. Discard this work

Which option?
```

### Step 5: Execute Choice

#### Option 1: Merge Locally

```bash
git checkout <base-branch>
git pull
git merge <feature-branch>

# Verify tests on merged result
<test command>

# After merge succeeds: delete branch
git branch -d <feature-branch>
```

#### Option 2: Push and Create PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

#### Option 3: Keep As-Is

Report: "Keeping branch <name>. Changes are committed and ready when needed."

#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

### Step 6: Confirm Completion

Report what was done:
- For merge: "Merged <feature-branch> into <base-branch>. Tests pass."
- For PR: "PR created at <url>."
- For keep: "Branch <name> preserved."
- For discard: "Branch <name> discarded."

## Quick Reference

| Option | Merge | Push | Keep Branch | Delete Branch |
|--------|-------|------|-------------|---------------|
| 1. Merge locally | yes | - | - | yes |
| 2. Create PR | - | yes | yes | - |
| 3. Keep as-is | - | - | yes | - |
| 4. Discard | - | - | - | yes (force) |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code, create failing PR
- **Fix:** Always verify tests before offering options

**Open-ended questions**
- **Problem:** "What should I do next?" is ambiguous
- **Fix:** Present exactly 4 structured options

**No confirmation for discard**
- **Problem:** Accidentally delete work
- **Fix:** Require typed "discard" confirmation

## Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on result
- Delete work without confirmation
- Force-push without explicit request from your human partner

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
