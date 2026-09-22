# Dotfiles

Personal macOS shell, Git, SSH, application, and package configuration.

## Set up a new Mac

Install [Homebrew](https://brew.sh), clone this repository, and run:

```sh
script/bootstrap
```

Bootstrap installs the packages and applications from a Brewfile, then runs
`script/setup`. Some App Store applications may require signing in first, Git
commit signing and SSH use 1Password. The optional `Black Sun.terminal`
profile can be opened manually to import it into Terminal; its Powerline font
is installed by the Brewfile.

Run `script/setup` by itself to install newly added links or repair missing
ones. It is safe to run repeatedly, must be run from a Git clone or worktree,
and refuses to overwrite unexpected files.

### Brewfiles

`dotfiles/Brewfile.core` holds everything wanted on every Mac. It's never
installed directly; `dotfiles/Brewfile.work` and `dotfiles/Brewfile.home` each
pull it in with `instance_eval(File.read(...))` and add machine-class-specific
casks/`mas` entries on top. `brew bundle`'s install/cleanup/list/check
subcommands all see the merged result, since the included file's `brew`/
`cask`/`mas` calls run in the same Ruby `instance_eval` context. `brew bundle
dump` does not — it overwrites whichever file it targets with a flat snapshot
of what's currently installed, so never run `dump` directly against these
three files.

Which file applies is chosen by the `HOMEBREW_BUNDLE_FILE` environment
variable (Homebrew's own mechanism), set per machine in an untracked, local
shell file — e.g. from `~/.zshrc.local`, sourced by the tracked zshrc but
never committed:

```sh
export HOMEBREW_BUNDLE_FILE="$HOME/Repos/dotfiles/dotfiles/Brewfile.work"
```

With that set, plain `brew bundle`, `brew bundle cleanup`, and `brew bundle
add` all operate on the right file with no flags. `script/bootstrap` respects
the same variable, falling back to `Brewfile.core` alone when it's unset.
`brew bundle add` appends new lines to whichever file is targeted; `brew
bundle remove` can only remove a line that's textually present in that same
file, so removing something declared in `Brewfile.core` requires explicitly
targeting it (`--file=dotfiles/Brewfile.core`). `mas` entries can't be added
via `brew bundle add` at all (Homebrew doesn't support it) — add those lines
by hand.

## Python

Use uv to manage development Python interpreters and Python command-line tools.
Homebrew may still install Python as a dependency of its own applications.

After installing the Brewfile, set up the development interpreters:

```sh
uv python install 3.12
uv python install 3.13 --default
uv python pin --global 3.13
```

The shell puts `~/.local/bin` on `PATH`, where uv exposes `python`, `python3`,
and the versioned commands. Install Python CLI tools with
`uv tool install --python 3.13 <package>` and run project commands with
`uv run`. Existing virtual environments must be recreated before removing
the interpreter they reference. Integrations use `ddev`; dd-source uses `bzl`.

## Managed wrappers

Most files under `dotfiles/` are linked directly into the home directory. The
shell and SSH entry points are different because local tools need to modify
them:

| Tracked configuration | Linked location | Real, tool-managed wrapper |
| --- | --- | --- |
| `dotfiles/config/dotfiles/zshrc` | `~/.config/dotfiles/zshrc` | `~/.zshrc` |
| `dotfiles/config/dotfiles/zprofile` | `~/.config/dotfiles/zprofile` | `~/.zprofile` |
| `dotfiles/config/dotfiles/ssh-config` | `~/.config/dotfiles/ssh-config` | `~/.ssh/config` |

The real wrappers source or include the tracked configuration. Dogbrew and
SCFW may update the shell wrappers, and Workspaces may update the SSH wrapper,
without changing files in this repository.

Because tracked configuration is live through symlinks, rewrite repository
history only in a disposable clone or worktree.
