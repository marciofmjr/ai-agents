---
name: preview-branch-orchestrator
description: Orchestrate safe integration of feature branches into a shared preview branch without overwriting other developers' work. Use when multiple developers need to validate different features in a single preview environment, when preview is ahead of main with temporary feature commits, when rebasing/cherry-picking manually is error-prone, or when the agent must inspect commit history and infer which branches are already represented in preview before pushing.
---

# Preview Branch Orchestrator

## Overview

Prepare and update a shared preview branch by replaying only missing feature commits on top of current preview, with strict anti-overwrite safeguards.

## Execution Mode

Execute Git commands directly from the agent.
Do not ask the user to run scripts.
Do not depend on helper scripts for normal execution.

## Mainline Detection (`main` or `master`)

Resolve the mainline branch with this order:

1. Read remote HEAD:
   `git symbolic-ref --short refs/remotes/origin/HEAD`
2. If result is `origin/main` or `origin/master`, use that branch.
3. If remote HEAD is missing, prefer existing branch in this order:
   - `refs/remotes/origin/main`
   - `refs/remotes/origin/master`
4. If none exist, ask user explicitly which branch is mainline.

## Push Authorization Policy

Never push preview automatically.
Push only if the user explicitly asks to push preview to remote in the current conversation.
Treat ambiguous intents as no-push and stop after local preparation.
Before pushing, summarize what changed and ask for final confirmation if user intent is not explicit enough.

## Branch Safety

The original feature branch MUST NEVER be modified.
All operations (rebase, cherry-pick) happen exclusively on temporary branches.
If any step fails, the user's feature branch remains untouched.

## Standard Workflow

1. **Fetch**: Fetch and prune remote refs.
2. **Resolve mainline**: Detect mainline branch (`main` or `master`) using detection rules above.
3. **Validate branches**:
   - If `origin/preview` does not exist, create it from `origin/<mainline>` and inform the user.
   - Validate target feature branch exists.
4. **Identify feature commits**: List commits unique to the feature branch relative to mainline:
   `git log --no-merges --format='%H' <mainline>..feature`
   Skip merge commits — they carry no unique code changes.
5. **Identify preview commits**: List commits already in preview relative to mainline:
   `git log --no-merges --format='%H' <mainline>..origin/preview`
6. **Detect missing commits**: Compare feature commits against preview commits using patch-id to find commits not yet represented in preview:
   ```
   git patch-id < <(git diff-tree -p <commit>)
   ```
   Alternatively, use `git cherry` or `git log --cherry-pick --right-only --no-merges origin/preview...feature` for built-in patch-id comparison.
7. **Create integration branch**: Create a temporary branch from `origin/preview` tip:
   `git checkout -b tmp/preview-<feature-name>-<timestamp> origin/preview`
8. **Cherry-pick missing commits**: Apply only the missing commits in chronological order onto the integration branch.
   - If a cherry-pick conflict occurs: stop, report the conflict, and resolve on the temporary branch only. Never touch the original feature branch.
9. **Report result**: Output the integration summary (see Output Contract below).
10. **Push**: Push to `origin/preview` only with explicit user request, always using `--force-with-lease`.
11. **Cleanup**: After push succeeds (or user cancels), delete all local temporary branches created during the process.

## Output Contract

After preparation, report:

- detected mainline branch
- preview base commit (short hash)
- number of commits considered from feature
- number of commits skipped (already represented in preview)
- number of commits applied to candidate
- list of applied commits (short hash + first line of message)
- list of skipped commits (short hash + first line of message)
- candidate local branch name for testing

## Conflict Policy

When cherry-pick fails due to conflicts:

1. Stop immediately. Do not attempt further cherry-picks.
2. Report which commit caused the conflict and which files are affected.
3. Resolve conflicts on the temporary branch only. Never modify the original feature branch.
4. After resolution, continue cherry-picking remaining commits.
5. Re-run validation/tests locally if applicable.
6. Push to preview with `--force-with-lease` only after explicit user authorization.

## Push Failure Recovery

If `--force-with-lease` is rejected (another developer pushed to preview concurrently):

1. Inform the user that preview changed since last fetch.
2. Re-run the entire workflow from step 1 (fetch) to incorporate the new changes.
3. Do not attempt a force push without lease protection.

See [preview-flow.md](references/preview-flow.md) for process details and team conventions.
