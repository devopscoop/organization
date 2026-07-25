# organization

Documentation for how this organization is run:

- [governance.md](governance.md) — how decisions are made
- [principles.md](principles.md) — the principles behind those decisions
- [repo_architecture.md](repo_architecture.md) — how the org's repositories fit together
- [bootstrap.md](bootstrap.md) — setting up the org's accounts and services (SCM, passwords, email, DNS)
- [claude_github_actions.md](claude_github_actions.md) — the Claude Code workflows in our repos and distributing the `CLAUDE_CODE_OAUTH_TOKEN` secret they need

## Install required packages

Local tools these docs use: `git` (bootstrap.md's multi-remote workflow), `gh` and `claude` (claude_github_actions.md's secret distribution):

- macOS, using [Homebrew](https://brew.sh/) and the `Brewfile`:

  ```shell
  brew bundle
  ```

- Arch Linux, using the `pkglist.txt` (`claude-code` is AUR-only, so use an AUR helper; the rest are in the official repos):

  ```shell
  grep -vE '^(#|$)' pkglist.txt | yay -S --needed -
  ```
