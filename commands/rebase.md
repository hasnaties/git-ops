---
description: Rebase the current feature branch onto the latest base branch, resolve any merge conflicts intelligently, and force-push. Use when a branch has merge conflicts with the base branch, when rebasing is needed before merging a PR, or when the user says /rebase.
argument-hint: [repo-name]
---

# Rebase

Rebase the current feature branch onto the latest base branch, resolve conflicts, and force-push.

## Usage

`/rebase <repo-name>`

Example: `/rebase my-app`

## Configuration

This command reads your repos from `.claude/repo-map.md` in the current project root.
The plugin ships **no** configuration — it is yours to fill in.

**First-time use:** If `.claude/repo-map.md` is missing or empty, do NOT fail. Instead:

1. Tell the user no config was found and that you'll set it up now.
2. Ask, one repo at a time, for: a short **name**, the absolute **path** to the clone,
   the **base branch** (e.g. `main`, `master`, `staging`), and the **remote** (e.g. `origin`).
3. Write the answers to `.claude/repo-map.md` using the table format below.
4. Then continue with the command.

Config format (`.claude/repo-map.md`):

| Repo | Path | Base branch | Remote |
|---|---|---|---|
| my-app | /Users/you/code/my-app | main | origin |

If the given repo name is not in the file, list the known names and stop.

## Permissions

Each step runs git as a **separate** Bash call of the form `git -C <path> ...`. Add
`Bash(git -C *)` to your `.claude/settings.json` allowlist (see the plugin README and
`settings.sample.json`) so these run without a permission prompt on every call.

## Steps

Run every git command as a **separate Bash tool call** — do NOT chain them with `&&` or `;`,
and do NOT use the PowerShell tool. Separate, unchained `git -C ...` calls each match the
`Bash(git -C *)` allowlist, so nothing prompts for approval (and you can inspect each result
before the next step).

1. Read the repo name from the user's message. If missing or unrecognised, list the valid names from the config and stop.
2. Look up the path, base branch, and remote for that repo from the config.
3. Stash any unstaged changes: `git -C <path> stash`
   - Capture the output. If it prints `No local changes to save`, remember there is **no stash** for
     this repo and SKIP the `stash pop` in step 7.
4. Fetch the latest base branch: `git -C <path> fetch <remote> <base-branch>`
5. Attempt the rebase: `git -C <path> rebase <remote>/<base-branch>`
6. If conflicts arise, for each conflicting file:
   a. List conflicts: `git -C <path> diff --name-only --diff-filter=U`
   b. Read the conflicted file — it contains `<<<<<<<` (current branch), `=======`, and `>>>>>>>` (incoming) markers.
   c. Read the base-branch version of the file: `git -C <path> show <remote>/<base-branch>:<file>` to understand the full context of both sides.
   d. Resolve by keeping the intent of both sides. If the correct resolution is ambiguous, ask the user before proceeding.
   e. Write the resolved file (no conflict markers), then stage it: `git -C <path> add <file>`
   f. Continue the rebase: `GIT_EDITOR="true" git -C <path> rebase --continue`
   g. Repeat from (a) if further conflicts remain.
7. Restore stashed changes — only if step 3 created one: `git -C <path> stash pop`
8. Force-push: `git -C <path> push --force-with-lease <remote> HEAD`
9. Report the result — list which files had conflicts and how they were resolved.
