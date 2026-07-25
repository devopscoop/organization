# organization

Documentation for how this organization is run:

- [governance.md](governance.md) — how decisions are made
- [principles.md](principles.md) — the principles behind those decisions
- [repo_architecture.md](repo_architecture.md) — how the org's repositories fit together
- [bootstrap.md](bootstrap.md) — setting up the org's accounts and services (SCM, passwords, email, DNS)

## Install required packages

The only local tool these docs require is `git` (bootstrap.md's multi-remote workflow). Package manifests are included for consistency with the org's other repos:

- macOS, using [Homebrew](https://brew.sh/) and the `Brewfile`:

  ```shell
  brew bundle
  ```

- Arch Linux, using the `pkglist.txt` (all packages are in the official repos):

  ```shell
  grep -vE '^(#|$)' pkglist.txt | sudo pacman -S --needed -
  ```
