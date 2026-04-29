# rewrite-commits

Rewrite commit messages in a given revision range to follow Conventional Commits (Angular style), then apply them to a new branch. Use when you need to clean up commit history for a range of commits.

## Overview

This skill analyzes each commit in a given revision range, rewrites the commit message using Conventional Commits (Angular style) based on the actual diff content, and applies the result to a new branch — fully non-interactive.

## Required Inputs

Before using this skill, the following inputs are needed:

| Parameter | Description | Example |
|---|---|---|
| `FROM_REVISION` | The **exclusive** base revision (not included in range) | `663386282993531e936b09f2a7e6a7b60a2f3290` |
| `TO_REVISION` | The **inclusive** tip revision (last commit to rewrite) | `5f07e2d2bb5805910669668a55b6fc5326cf39eb` |
| `BRANCH_NAME` | Name for the new branch to create | `rewrite-commits-branch` |

## Workflow

1. **Verify & Collect**: Gathers all commits in the range.
2. **Branch Creation**: Creates a new branch starting from `FROM_REVISION`.
3. **Analyze**: Reads the actual diff of each commit.
4. **Generate**: Creates a Conventional Commit message with proper `<type>`, `<scope>`, `<subject>`, and `<body>`.
5. **Apply**: Cherry-picks and amends each commit onto the new branch sequentially.