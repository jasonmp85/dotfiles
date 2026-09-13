# Dotfiles

Personal macOS shell, Git, SSH, application, and package configuration.

## Set up a new Mac

Install [Homebrew](https://brew.sh), clone this repository, and run:

```sh
script/bootstrap
```

Bootstrap installs the packages and applications in `dotfiles/Brewfile`, then
runs `script/setup`. Some App Store applications may require signing in first.
The optional `Black Sun.terminal` profile can be opened manually to import it
into Terminal.

Run `script/setup` by itself to install newly added links or repair missing
ones. It is safe to run repeatedly and refuses to overwrite unexpected files.

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
