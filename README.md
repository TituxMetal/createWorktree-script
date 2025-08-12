## createWorktree — Git worktree helper

### Overview

`createWorktree` is a small Bash utility that streamlines creating Git worktrees for feature
development. From the root of an existing Git project, it will:

- Create a per-project worktrees directory at `~/webdev/worktrees/{project-name}-worktrees/`.
- Create a new worktree and branch named after your provided feature name.
- Open the new worktree in a new Cursor window (if the `cursor` CLI is available).

This lets you quickly spin up isolated working directories without copying your entire repository.

### Prerequisites

- **Git**: version with `git worktree` support (Git 2.5+).
- **Bash**: script uses `bash`, `set -euo pipefail`, and standard POSIX utilities.
- **Cursor CLI (optional)**: to auto-open the worktree in a new window. If not available, the script
  prints the path to open manually.

### Installation

- Quick install from the project directory:

```bash
./createWorktree --install
```

- Place the script at `~/bin/createWorktree` and make it executable:

```bash
chmod +x "$HOME/bin/createWorktree"
```

- Either add `~/bin` to your `PATH` or define an alias in your shell config (`~/.bashrc`,
  `~/.zshrc`, etc.):

```bash
alias createWorktree="$HOME/bin/createWorktree"
```

- Reload your shell configuration or start a new terminal session.

### Usage

- From the root of your repository directory (e.g., `~/webdev/labo/myapp`), run:

```bash
createWorktree <feature-name>
```

- Example:

```bash
createWorktree feat/login-ui
```

### What the script does

- **Project name detection**: Uses the base name of `PWD` as `{project-name}`.
- **Worktrees location**: Creates (if needed) `~/webdev/worktrees/{project-name}-worktrees/`.
- **Worktree target path**: Places the new worktree at
  `~/webdev/worktrees/{project-name}-worktrees/{feature-name}`. If the feature name contains slashes
  (e.g., `features/name`), only the folder name is normalized by replacing `/` with `-` (e.g.,
  `features-name`) to avoid nested directories. The Git branch keeps the original name with slashes.
- **Base branch selection**: Picks the first available in this order:
  1. `origin/main`
  2. `main`
  3. `origin/master`
  4. `master`
  5. `HEAD` (fallback)
- **Branch creation**:
  - If the branch does not exist, it creates `feature-name` from the selected base ref and adds the
    worktree.
  - If the branch already exists locally, it adds a worktree for that branch.
- **Opening in Cursor**: Attempts `cursor -n <path>` and falls back to `cursor <path>`. If the
  `cursor` CLI is unavailable, it prints a note and the path.

### Safeguards and idempotency

- Fails if the target directory already exists to avoid clobbering:
  `~/webdev/worktrees/{project-name}-worktrees/{feature-name}`.
- Runs `git fetch --all --prune --quiet` to keep remote refs current before choosing a base ref.

### Examples

- New feature branch and worktree from `origin/main` if available:

```bash
createWorktree feature/payment-intents
```

- Bug fix worktree using an existing local branch:

```bash
createWorktree bugfix/issue-1234
```

### Error messages and troubleshooting

- **Error: Not inside a git repository.**
  - Make sure you run the command from inside a Git repo (ideally the repo root).
- **Error: Target path already exists: ...**
  - Choose a different `feature-name`, or remove the existing directory if it’s not needed.
- **Note: 'cursor' CLI not found in PATH...**
  - Install the Cursor CLI or open the printed path manually in Cursor.
- **Base branch not found**
  - If your repo doesn’t have `main` or `master`, the script falls back to `HEAD`.

### Removing a worktree

- To remove a worktree and keep the branch:

```bash
git worktree remove /absolute/path/to/worktree
```

- To delete the branch after removing the worktree (optional):

```bash
git branch -D <feature-name>
```

### Design notes

- Uses camelCase function names and guard/short-circuit style (no `if/elif/else` chains).
- Comments are placed above each line for readability and consistency.
- Indentation is consistent for code and comments within functions.

### Compatibility

- Developed and tested on Linux. Should work on macOS with compatible Git and Bash.

### Updating

- Pull latest changes to your Git repository as needed; this script itself is standalone. If you
  adjust behavior (e.g., worktrees root), remember to update this README accordingly.
