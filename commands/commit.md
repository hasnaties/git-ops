---
description: Commit staged changes and push on the current branch for a configured repo. Does NOT create a new branch. Use when the user says /commit or wants to commit and push staged changes.
argument-hint: [repo-name]
---

# Commit

Commit staged changes and push on the current branch for a configured repo. Does **NOT** create a new branch.

## Usage

`/commit <repo-name>`

Example: `/commit my-app`

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

## Protected base branches (do not push here)

Never push commits directly to a protected branch. After committing, read the current
branch: `git -C <path> rev-parse --abbrev-ref HEAD`.

If the branch name is **exactly** one of: `staging`, `main`, `master`, **then:**

- **Do not run `git push`.**
- Tell the user the commit exists **only locally** on that branch.
- Instruct them to move the work to a feature branch and push that instead, for example:
  - `git -C <path> checkout -b feat/<short-description>`
  - `git -C <path> push -u <remote> HEAD`
  - Open a PR to the base branch.

Only proceed to the push step when the current branch is **not** in the list above.

## Steps

Run every git command as a **separate Bash tool call** — do NOT chain them with `&&` or `;`,
and do NOT use the PowerShell tool. Separate, unchained `git -C ...` calls each match the
`Bash(git -C *)` allowlist, so nothing prompts for approval (and you can inspect each result
before the next step).

1. Read the repo name from the user's message. If missing or unrecognised, list the valid names from the config and stop.
2. Look up the path and remote for that repo from the config.
3. Run `git -C <path> diff --staged` to read what changed. Do NOT ask the user to describe it. If nothing is staged, run `git -C <path> status` and stop, informing the user.
4. Derive a commit message from the diff — sentence-case, imperative mood, no trailing period. Commit immediately on the current branch: `git -C <path> commit -m "<message>"`
5. **Protected branch check:** If current branch is `staging`, `main`, or `master`, **skip push** and follow **Protected base branches** above. Otherwise push: `git -C <path> push <remote>`. Use `git push -u <remote> HEAD` when upstream is not set.
