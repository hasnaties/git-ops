---
description: Commit staged changes, create a correctly-named feature branch, push, and generate a PR description derived from the branch diff. Use when the user says /ship-it or is ready to open a PR.
argument-hint: [repo-name]
---

# Ship It

Commit staged changes and push for a configured repo, then output a ready-to-paste PR description derived from the branch diff.

## Usage

`/ship-it <repo-name>`

Example: `/ship-it my-app`

The user may add optional hints in the same message (screenshot URL for Tests Status, demo video title/link, ticket-style titles, etc.). Use them to fill those sections; otherwise derive everything from git and use short placeholders or omit unknowns.

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
3. Run `git -C <path> diff --staged` to read what changed. Do NOT ask the user to describe it. If nothing is staged, run `git -C <path> status` and stop, informing the user.
4. Derive a branch name from the diff using the correct prefix (`fix/`, `feat/`, `chore/`, `hotfix/`). Check out the base branch, pull the latest from remote to ensure it's up to date, then create the feature branch from it — as three **separate** Bash calls:
   - `git -C <path> checkout <base-branch>`
   - `git -C <path> pull <remote> <base-branch>`
   - `git -C <path> checkout -b <branch-name>`
5. Derive a commit message from the diff — sentence-case, imperative mood, no trailing period. Commit immediately: `git -C <path> commit -m "<message>"`
6. Push: `git -C <path> push -u <remote> HEAD`. Capture the remote output — it often contains a `Create pull request for …` URL. Extract and display that URL as a clickable markdown link immediately after the push so the user can open it directly.
7. Generate a PR description (see below): run `git -C <path> diff <base-branch>...HEAD` and optionally `git -C <path> log <base-branch>..HEAD --oneline` so the write-up reflects the **whole branch**, not only the last staged hunk. Output **one markdown block** for the user to paste into the PR description.

## PR description template

Write for **mixed reviewers** (PM, QA, engineering): plain language first — what people see or get — not a code dump. All sections use **prose paragraphs**, not bullet lists.

### Audience and tone

- Lead with the user-visible symptom or outcome, not internal refactors.
- Prefer everyday terms; spell out acronyms when they might confuse non-authors.
- Reference components, functions, or files **only** when it helps a reviewer verify or test — avoid implementation laundry lists.
- Keep every statement grounded in the actual diff; do not invent detail.

**Title line** (first line of the description body):

```
Task: {Full descriptive title — what was broken or added, in plain language}
```

Derive the title from the branch name, commits, and diff. It should read like a task or ticket title, not a code symbol.

**Evidence line** (second line — omit entirely if the user did not provide a link):

```
Evidence: {video or screenshot link}
```

**Sections** (use plain-text headers with no `##`; include only sections with content):

| Section | Content |
|---|---|
| Problem | 1–3 sentence paragraph describing what was broken, missing, or painful before this change — from the user's perspective. Derived from the diff, commits, and conversation context. No code dumps. |
| Solution | 1–3 sentence paragraph explaining what was changed and why. Reference specific components, functions, or files when they aid understanding. Explain any non-obvious implementation choices (e.g. timing workarounds, fallback logic). |
| Tests Status | Link to a screenshot or one-line status. Use a URL if the user provided one; otherwise omit this section. |

**Trailing tracker lines** (after the sections, if applicable):

```
Fixes {human-readable issue or ticket title}
```

Use issue titles from the branch name or commit messages; do not fabricate ticket IDs.

Do **not** hardcode example content from unrelated PRs — always base content on the actual `git diff` and log for this branch.

## Out of scope

- Creating the PR via API or CLI (paste-only description output here). The PR link from the push output is displayed for the user to open manually.
