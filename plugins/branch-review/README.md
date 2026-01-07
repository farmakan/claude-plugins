# Branch Review Plugin

Pre-PR code review plugin for Claude Code. Reviews all changes on your branch compared to main before creating a pull request.

## Usage

```bash
/branch-review           # Review current branch vs main
/branch-review --base develop  # Review current branch vs develop
```

## What It Reviews

Reviews **all changes** compared to the base branch:
- **Committed changes**: All commits on your branch
- **Staged changes**: Files added to git staging area
- **Unstaged changes**: Working directory modifications

## How It Works

1. **Pre-flight checks**: Verifies you're on a feature branch with changes to review
2. **Gathers context**: Finds CLAUDE.md guidelines and gets diffs (committed, staged, unstaged)
3. **Parallel review**: Runs 4 agents in parallel checking for:
   - CLAUDE.md compliance violations
   - Bugs and logic errors
   - Security issues
4. **Validation**: Confirms issues with high confidence (80+)
5. **Reports**: Outputs a structured summary with issues tagged by status (committed/staged/unstaged)

## Quality Standards

Only reports high-signal issues:
- Objective bugs and security vulnerabilities
- Clear CLAUDE.md violations with specific rule references
- Logic errors that would cause incorrect behavior

Does NOT report:
- Subjective style preferences
- Linter-catchable issues
- Hypothetical edge cases

## Installation

```bash
/plugin marketplace add farmakan/claude-plugins
/plugin install branch-review@farmakan-plugins
```
