---
description: Stash local changes, pull the latest from the base branch, and restore stashed changes for a configured repo. Use when the user says /sync or wants to update their local branch with the latest from the base branch.
argument-hint: [repo-name]
---

# Sync

Stash local changes, pull latest from the base branch, and restore stashed changes for a configured repo.

## Usage

`/sync <repo-name>`

Example: `/sync my-app`

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
3. Stash local changes: `git -C <path> stash`
   - Capture the output. If it prints `No local changes to save`, remember there is **no stash** for
     this repo and SKIP the `stash pop` in step 6.
4. Switch to base branch: `git -C <path> checkout <base-branch>`
5. Pull latest: `git -C <path> pull <remote> <base-branch>`
6. Pop the stash — only if step 3 created one: `git -C <path> stash pop`. Stay on the base branch; do
   NOT checkout any other branch before or after this step.
7. Report the result. Confirm the current branch is `<base-branch>`. If stash pop has conflicts, inform the user clearly.
