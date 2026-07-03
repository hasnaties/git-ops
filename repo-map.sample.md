# Repo Map

Per-project config for the `git-ops` plugin. Copy this file to `.claude/repo-map.md` in
your project root and fill in your own repos — one row per repo. The plugin ships **no**
real config, so this is yours to own.

The `commit`, `sync`, `ship-it`, and `rebase` commands read this file to resolve each
repo's **path**, **base branch**, and **remote**. The first time you run any command with
no config present, it will interview you and write this file for you automatically.

| Repo | Path | Base branch | Remote |
|---|---|---|---|
| my-app | /Users/you/code/my-app | main | origin |
| my-api | /Users/you/code/my-api | staging | origin |

Notes:
- **Path** is the absolute path to the clone on *your* machine (Windows: `C:/Users/you/code/my-app`).
- **Base branch** is the branch features merge into (e.g. `main`, `master`, `staging`).
- **Remote** is usually `origin`.
- `commit` only uses **Path** and **Remote** (it never switches base branches).
- If this file contains machine-specific absolute paths, add `.claude/repo-map.md` to your
  `.gitignore`. If your team shares a common layout and only needs repo/base/remote, you may
  commit it instead.
