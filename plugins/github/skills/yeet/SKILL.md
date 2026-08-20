---
name: "yeet"
description: "Publish already-authorized repository work when a pull request is the expected artifact or the user asks to publish. Run branch setup, staging, commit, push, and PR creation as one continuous workflow without per-step confirmation; never merge the PR."
---

# GitHub Publish Changes

## Overview

Use this skill when already-authorized repository work is expected to produce a pull request or the user explicitly asks to publish changes. A clear PR deliverable authorizes the complete standard flow from the local checkout: branch setup if needed, staging the intended scope, commit, push, and pull-request creation. It does not authorize otherwise-unapproved implementation. Do not pause for separate confirmation of those component steps.

If the user explicitly limits the request to local changes, a patch, review, or planning without publication, do not publish. When a PR is expected, opening it is required unless material scope or target ambiguity, a higher-priority required gate, or a concrete blocker prevents completion. Never merge a pull request or enable auto-merge; hand the PR to the user.

This workflow is hybrid:

- Use local `git` for branch creation, staging, commit, and push.
- Prefer the GitHub app from this plugin for pull request creation after the branch is on the remote.
- Use `gh` as a fallback for current-branch PR discovery, auth checks, or PR creation when the connector path cannot infer the repository or head branch cleanly.

## Prerequisites

- Require GitHub CLI `gh`. Check `gh --version`. If missing, ask the user to install `gh` and stop.
- Require authenticated `gh` session. Run `gh auth status`. If not authenticated, ask the user to run `gh auth login` (and re-run `gh auth status`) before continuing.
- Require a local git repository with a clean understanding of which changes belong in the PR.

## Naming conventions

- Branch: `codex/{description}` when starting from main/master/default.
- Commit: `{description}` (terse).
- PR title: `[codex] {description}` summarizing the full diff.

## Workflow

1. Establish intended scope.
   - Run `git status -sb` and inspect the diff before staging.
   - If the working tree contains unrelated changes, do not default to `git add -A`. Ask the user which files belong in the PR.
2. Determine the branch strategy.
   - If on `main`, `master`, or another default branch, create `codex/{description}`.
   - Otherwise stay on the current branch.
3. Stage only the intended changes.
   - Prefer explicit file paths when the worktree is mixed.
   - Use `git add -A` only when the user has confirmed the whole worktree belongs in scope.
4. Commit tersely with the established description.
5. Run the most relevant checks available if they have not already been run.
   - If checks fail due to missing dependencies or tools, install what is needed and rerun once.
6. Push with tracking: `git push -u origin $(git branch --show-current)`.
7. Open a draft PR.
   - Prefer the GitHub app from this plugin for PR creation after the push succeeds.
   - Derive `repository_full_name` from the remote, for example by normalizing `git remote get-url origin` or by using `gh repo view --json nameWithOwner`.
   - Derive `head_branch` from `git branch --show-current`.
   - Derive `base_branch` from the user request when specified; otherwise use the remote default branch, for example via `gh repo view --json defaultBranchRef`.
   - If the branch is being pushed from a fork or the PR target differs from the remote that was just pushed, prefer `gh pr create` fallback because the connector PR creation flow expects one repository target and may not encode cross-repo head semantics cleanly.
   - If connector-based PR creation cannot infer the repository or branch cleanly, fall back to `gh pr create --draft --fill --head $(git branch --show-current)`.
   - Write the PR body to a temp file with real newlines when using CLI fallback so the markdown renders cleanly.
8. Summarize the result with branch name, commit, PR target, validation, and any concrete blocker or follow-up.

## Write Safety

- Never stage unrelated user changes silently.
- Never push without confirming scope when the worktree is mixed.
- Do not ask for separate branch, stage, commit, push, or PR-creation approvals after the PR deliverable is clear.
- Do not stop after local edits, commit, or push when the expected artifact is a PR.
- Never merge a pull request or enable auto-merge; hand the PR to the user.
- Follow higher-priority repository instructions for draft versus ready-for-review state. Otherwise default to a draft PR unless the user explicitly asks for ready-for-review.
- If the repository does not appear to be connected to an accessible GitHub remote, stop and explain the blocker before making assumptions.

## PR Body Expectations

The PR description should use real Markdown prose and cover:

- what changed
- why it changed
- the user or developer impact
- the root cause when the PR is a fix
- the checks used to validate it
