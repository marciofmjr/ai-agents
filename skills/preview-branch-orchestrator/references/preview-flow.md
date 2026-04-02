# Shared Preview Flow

## Goal

Maintain one preview branch that can contain multiple in-flight features without deleting or rewriting other developers' work.

## Branch Roles

- `main` or `master`: production-ready code (resolve dynamically per repo)
- `preview`: integration branch for temporary validation in preview environment
- `feature/*`: developer branches with isolated work
- `tmp/preview-*`: temporary branches created by automation (disposable after push or cancellation)

## Recommended Process

1. Fetch latest remote refs with prune.
2. Detect mainline branch (`main` or `master`) from remote HEAD, with fallback checks.
3. Validate `origin/preview` exists. If not, create from `origin/<mainline>`.
4. Identify feature commits: `git log --no-merges <mainline>..feature` (skip merge commits).
5. Identify preview commits: `git log --no-merges <mainline>..origin/preview`.
6. Compare by patch-id to find missing commits. Use `git cherry` or `git log --cherry-pick --right-only --no-merges origin/preview...feature` for built-in comparison.
7. Create temporary integration branch from latest preview tip.
8. Cherry-pick only missing feature commits in chronological order.
9. Validate tests/smoke checks if applicable.
10. Push to preview with `--force-with-lease` only if user explicitly requested push.
11. Delete local temporary branches after completion.

## Safety Guarantees

- `--force-with-lease` avoids blind overwrite when preview changed concurrently.
- Temporary branches preserve original feature branch history.
- Duplicate detection by patch-id avoids replaying already integrated changes.
- No implicit remote push avoids accidental shared-environment disruption.
- The original feature branch is never modified.
- Merge commits are skipped during cherry-pick to avoid conflicts from non-code commits.

## Push Failure Recovery

If `--force-with-lease` is rejected:

1. Another developer pushed to preview concurrently.
2. Re-fetch remote refs and restart the workflow from the beginning.
3. Never fall back to `--force` without lease protection.

## Temporary Branch Cleanup

- After a successful push to preview, delete all `tmp/preview-*` branches created during the session.
- After a cancelled operation, delete all `tmp/preview-*` branches created during the session.
- Periodically clean stale remote `tmp/preview-*` branches if they exist.

## Team Conventions (Optional but Recommended)

- Protect direct pushes to `preview` in CI unless push is from bot/automation.
- Require successful smoke tests before accepting preview update.
- Use short-lived preview updates: once validated, merge feature to main quickly.
- Keep branch naming predictable (`feature/*`) to improve overlap inference quality.

## Conflict Handling

When cherry-pick conflicts happen:

1. Stop immediately before any push.
2. Report the conflicting commit and affected files.
3. Resolve conflicts on temporary branch only. Never modify the original feature branch.
4. After resolution, continue cherry-picking remaining commits.
5. Re-run tests.
6. Push with `--force-with-lease` only after explicit user approval.
