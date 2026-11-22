# git-pushall

Finds every git repository under your home (or the directories you name), pushes each
branch that is ahead of its upstream, and prints a summary of what was pushed, what was
skipped and why. Only fast-forward pushes are made; there is no force option.
With `--pull` it also fast-forwards branches whose upstream is ahead.

One self-contained Python script. Requires Python 3.11+ and git.

## Install

```sh
./git-pushall --install
```

Copies the script to `~/.local/bin/git-pushall` (so `git pushall` works too) and writes
`~/.config/git-pushall/config.toml`. Run it again after updating the script.

## Use

```sh
git-pushall                 # fetch, push what is ahead, summarise
git-pushall -n              # dry run: report only
git-pushall --no-fetch      # compare against what git last saw (also --fetch=false)
git-pushall --pull          # also fast-forward branches that are behind (also --pull=true)
git-pushall ~/code ~/work   # search these directories instead of the configured roots
```

Other options: `-j N` (parallel repositories), `-c FILE` (another config), `-v` (also list
up-to-date repositories), `-q` (summary only), `--color auto|always|never`.
See `git-pushall --help`.

Exit status: 0 when everything is in sync, 1 when something was skipped or failed,
2 for a usage or config error.

## What it does per branch

| State | Default | With `--pull` |
| --- | --- | --- |
| ahead of upstream | pushed | pushed |
| behind upstream | listed under Notes | fast-forwarded¹ |
| ahead and behind (diverged) | skipped | skipped |
| upstream deleted on the remote | skipped | skipped |
| no upstream | skipped (or pushed with `-u` if `push_new_branches = true`) | same |

¹ The checked-out branch is only fast-forwarded when the working tree is completely clean
and no merge, rebase, cherry-pick, revert or bisect is in progress. Other branches are
moved without checking them out.

Uncommitted changes never block a push (only commits are pushed) unless
`skip_dirty = true`; they are listed under Notes.

## Configuration

`~/.config/git-pushall/config.toml` (or `$XDG_CONFIG_HOME/git-pushall/config.toml`) is
created on the first run with every setting commented out at its default and documented.
Uncomment a line to change a default; command-line options take precedence. Delete the
file to regenerate it, for example after an update adds settings.

Useful ones: `remote_patterns = ["*github.com*"]` to consider only your GitHub remotes,
`roots` and `exclude` for where to search, `skip_hidden = false` if you keep repositories
in hidden directories.

Fetches and pushes never prompt for passwords: they run without a terminal, so a remote
that needs interactive credentials shows up under Errors instead of hanging the run.
