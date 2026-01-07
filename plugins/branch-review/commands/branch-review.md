---
description: Review all changes (committed, staged, and unstaged) on your branch vs main before creating a pull request
argument-hint: "[--base <branch>]"
allowed-tools: ["Bash", "Glob", "Grep", "Read", "Task"]
---

# Branch Review Command

Review all code changes on the current branch compared to the base branch (default: main) before creating a pull request. This includes:
- All commits on the branch
- Staged changes (not yet committed)
- Unstaged changes (working directory modifications)

This helps catch issues early, before code review.

## Workflow

### Step 1: Pre-flight Checks

Run these bash commands in parallel:
- `git rev-parse --is-inside-work-tree` - verify we're in a git repo
- `git branch --show-current` - get current branch name
- `git rev-parse --verify main 2>/dev/null || git rev-parse --verify master 2>/dev/null` - determine base branch

If user provided `--base <branch>`, use that as the base branch instead.

Verify:
- We're in a git repo
- We're not on the base branch (main/master)
- There are changes to review (either commits, staged, or unstaged changes vs base)

If no changes found at all, report that and stop.

### Step 2: Gather Context

Run in parallel:
1. **Find CLAUDE.md files**: Use Glob to find all `**/CLAUDE.md`, `**/.claude/settings.json`, and similar guideline files in the repo
2. **Get commit list**: `git log <base>..HEAD --oneline` to see committed changes
3. **Get working tree status**: `git status --short` to see staged and unstaged files
4. **Get full diff** (all changes - committed + staged + unstaged vs base): `git diff $(git merge-base <base> HEAD)`

Also get separate diffs for context:
- `git diff <base>...HEAD` - committed changes only
- `git diff --staged` - staged changes only
- `git diff` - unstaged changes only

Read any CLAUDE.md files found to understand project coding standards.

### Step 3: Get Change Summary

Use the Task tool with a haiku model agent to summarize the changes:
- What files were modified/added/deleted
- High-level description of what the changes do
- Areas of the codebase affected
- Breakdown by change type:
  - Committed changes (X commits)
  - Staged changes (ready to commit)
  - Unstaged changes (work in progress)

### Step 4: Parallel Code Review

Launch 4 review agents in parallel using the Task tool:

**Agents 1-2: CLAUDE.md Compliance**
- Agent 1 reviews first half of changed files
- Agent 2 reviews second half of changed files
- Check for violations of any guidelines found in CLAUDE.md
- Only flag clear, quotable violations (not subjective style preferences)

**Agents 3-4: Bug and Logic Issues**
- Agent 3 Opus bug agent (parallel subagent with agent 4). reviews first half of changed files
- Agent 4 Opus bug agent (parallel subagent with agent 3). reviews second half of changed files
- Look for:
  - Null pointer / undefined access
  - Off-by-one errors
  - Resource leaks
  - Race conditions
  - Error handling gaps
  - Logic errors
  - Security issues (injection, XSS, etc.)

Each agent should return issues in this format:
```
ISSUE:
- File: <filepath>
- Line: <line number or range>
- Type: <bug|security|guideline-violation>
- Confidence: <0-100>
- Description: <clear explanation>
- Code: <relevant code snippet>
```

### Step 5: Issue Validation

For each issue with confidence >= 60, launch a parallel validation agent using the Task tool:
- Re-read the code in question with surrounding context
- Determine if the issue is real or a false positive
- Assign final confidence score (0-100)

Only keep issues with final confidence >= 80.

### Step 6: Report Results

Output a structured report to the terminal:

```markdown
# Branch Review: <branch-name>

## Summary
- Commits reviewed: <count>
- Staged files: <count>
- Unstaged files: <count>
- Total files changed: <count>
- Issues found: <count>

## Changes Overview
<summary from step 3>

## Issues

### Critical (if any)
<issues with confidence 95+>

### High Confidence (if any)
<issues with confidence 80-94>

For each issue, indicate if it's in:
- [committed] - already committed code
- [staged] - staged but not committed
- [unstaged] - working directory changes

## Reviewed Files
<list of files that were reviewed, grouped by status>
```

If no issues found, report:
```markdown
# Branch Review: <branch-name>

## Summary
- Commits reviewed: <count>
- Staged files: <count>
- Unstaged files: <count>

No issues found. The branch looks ready for PR review.
```

## Quality Standards

**Only flag issues that are:**
- Objective runtime bugs or security vulnerabilities
- Clear, quotable CLAUDE.md violations with specific rule references
- Logic errors that would cause incorrect behavior

**IMPORTANT: Only post ONE comment per unique issue. Do not post duplicate comments.**

**Do NOT flag:**
- Pre-existing issues
- Subjective code style preferences
- Pedantic nitpicks that a senior engineer would not flag
- "Could be improved" suggestions
- Issues that linters/formatters would catch
- Missing documentation or comments (unless CLAUDE.md requires them)

## Notes

- Use `git diff <base>` (no dots) to get all changes (committed + staged + unstaged) vs base branch
- Use `git diff <base>...HEAD` (three dots) for committed changes only
- Use `git diff --staged` for staged changes only
- Use `git diff` for unstaged changes only
- When reporting line numbers, use the line numbers from the new version of the file
- Include enough context in code snippets to understand the issue (3-5 lines usually)
- When categorizing issues, check which diff the change appears in to determine if it's committed, staged, or unstaged
- Create a todo list before starting.
