# git-ops

Everyday git workflow slash commands for Claude Code. Four commands you reach for daily —
no personal data shipped. You bring your own repo config, and the commands set it up for
you on first use.

## Commands

| Command | What it does |
|---|---|
| `/commit <repo>` | Commit staged changes and push on the **current** branch. Never creates a branch; refuses to push to protected branches (`main`/`master`/`staging`). |
| `/sync <repo>` | Stash → checkout base branch → pull latest → restore stash. Updates your local base branch safely. |
| `/ship-it <repo>` | Commit, create a correctly-prefixed feature branch, push, and generate a ready-to-paste PR description from the whole-branch diff. |
| `/rebase <repo>` | Rebase the current branch onto the latest base branch, resolve conflicts intelligently, and force-push with `--force-with-lease`. |

> A personal `sync-io` command is intentionally **not** included — it was specific to one
> developer's dual-clone setup and does not generalize.

## Prerequisites

- [Claude Code](https://code.claude.com/docs)
- `git` installed and on your PATH

## Install

This repo is a self-contained, single-plugin marketplace, so both steps point at it:

```
/plugin marketplace add hasnaties/git-ops
/plugin install git-ops@git-ops
```

### Command names are namespaced

Once installed as a plugin, the commands are prefixed with the plugin name — so you invoke
them as `/git-ops:commit`, `/git-ops:sync`, `/git-ops:ship-it`, and `/git-ops:rebase`
(not bare `/commit`). This prevents clashes with any same-named commands from other plugins.

## First-time setup

The plugin ships **no** configuration by design — none of the author's paths or repos. The
first time you run any command in a project with no config present, it will:

1. Tell you no config was found.
2. Interview you for each repo — a short **name**, the absolute **path**, the **base branch**,
   and the **remote**.
3. Write your answers to `.claude/repo-map.md` in that project root.
4. Continue with the command.

After that, every command reads `.claude/repo-map.md` silently. You can also create it
yourself by copying [`repo-map.sample.md`](repo-map.sample.md). Format:

| Repo | Path | Base branch | Remote |
|---|---|---|---|
| my-app | /Users/you/code/my-app | main | origin |

### Commit vs. gitignore the config

- If `.claude/repo-map.md` holds **machine-specific absolute paths**, add it to `.gitignore`
  (it's personal to your machine).
- If your team shares a common folder layout and the file only carries repo/base/remote,
  you may **commit** it so teammates inherit it.

## Permissions (do this once, or you'll be prompted on every git call)

The commands run git as separate `git -C <path> ...` calls. Allowlist that pattern so they
run without a prompt each time. Copy [`settings.sample.json`](settings.sample.json) into your
project's `.claude/settings.json` (or merge the `allow` entry into an existing one):

```json
{
  "permissions": {
    "allow": ["Bash(git -C *)"]
  }
}
```

Committing this file into your project repo shares the allowlist with the whole team.

## Safety behaviors

- `/commit` **refuses** to push to `main`, `master`, or `staging` — it stops and tells you to
  move the work to a feature branch.
- `/rebase` force-pushes with `--force-with-lease` (safe against clobbering others' work),
  not a bare `--force`.
- Only `/ship-it` creates branches. `/commit` never does.

## Notes & assumptions

- Path format is whatever your OS uses — Windows `C:/Users/you/...` or macOS/Linux `/Users/you/...`.
- Remotes default to `origin` unless your config says otherwise.
- Commands operate by **repo name** looked up in your config, so you can drive several repos
  from one Claude Code session.

## Updating

```
/plugin marketplace update git-ops
```
