# AGENTS.md

Instructions for AI coding agents working in this repo. `CLAUDE.md` is a symlink
to this file, so agents that look for that filename load this same guidance.
Edit `AGENTS.md`; never replace the symlink with a second copy.

## What this repo is

Prose documentation for how devops.coop runs itself. There is no application
code, no build, no test suite, and no linter — every change is a Markdown edit,
and "verifying" a change means re-reading the docs it touches for
contradictions. The only automation is the Claude Code workflows in
`.github/workflows/`, which comment on PRs.

Each doc answers a different question, and they are meant to be read in this
order:

- `principles.md` — the values everything else appeals to (a bare list, no prose).
- `governance.md` — the cooperative's bylaws: membership, general assembly,
  board, consensus-first decision making, amendment by two-thirds vote.
- `repo_architecture.md` — a worked thought experiment comparing repo layouts
  for the org's *other* repos, ending in a recommended architecture.
- `bootstrap.md` — the accounts and services actually provisioned (Codeberg /
  GitLab / GitHub orgs, Bitwarden, Migadu, registrar DNS).
- `README.md` — index of the above plus package install instructions.

## Cross-doc invariants

These are the things that silently contradict each other if edited in isolation:

- **Naming rules are derived, not arbitrary.** `repo_architecture.md`'s
  recommendation — app repos named for the app alone (`app1`), team and
  environment only in infra/deploy repos (`team1_prod/` containing `infra/` and
  `deploy/`) — follows directly from `principles.md`'s "Don't put what a thing
  is in its name" and "Name things what a human would actually verbally call
  it", plus the observation that teams change but apps outlive them. Changing a
  naming rule in one file means changing the other.
- **Separation of duties drives the repo split.** Dev approves app code; Ops
  approves prod infra and deploy; Dev may approve non-prod infra and deploy.
  Every "who can approve what" sentence in `repo_architecture.md` must stay
  consistent with that, and with the stated assumption that permissions are
  practical per-repo, sometimes per-branch, and not per-directory.
- **`repo_architecture.md`'s assumptions section is load-bearing.** Trunk-based
  development, Flux over ArgoCD/Helmfile, secrets kept out of the SCM provider,
  one cluster per team per environment — later sections argue from these. Don't
  contradict an assumption further down the file without amending it.
- `README.md` links every top-level doc; a new doc means a new bullet there.

## Multi-remote git

Per `bootstrap.md`, org repos are mirrored to three SCMs (codeberg.org,
gitlab.com, github.com) with one git remote per platform, fetched with
`git pull --all` and pushed with a `git pushall` alias. A given checkout may
have only `origin` configured — check `git remote -v` before assuming a push
reaches every mirror, and don't rewrite the multi-remote instructions to assume
a single remote.

## Package manifests

This repo ships a `Brewfile` (macOS: `brew bundle`) and a `pkglist.txt` (Arch
Linux) that install every CLI tool the repo uses (currently just `git`). Keep
them in sync with the docs:

- If a doc starts instructing readers to run a new local tool, add the package
  to BOTH files, with a comment noting what uses it.
- When a tool stops being used, remove it from both files.
- Hosted services (SCM platforms, password managers, email, DNS) are not
  installable binaries and don't belong in the manifests.
- Verify package names before adding them: `brew info <formula>` for Homebrew,
  and the official repos/AUR for Arch. If a package is AUR-only, note that in
  pkglist.txt's header instructions.
- Update the "Install required packages" section in README.md if the tool list
  changes.

## GitHub Actions

- Every `uses:` ref is pinned to a full commit SHA with a trailing version
  comment (`@11d5960... # v4`). Pin new ones the same way; a tag or branch ref
  will not match the repo's convention.
- Both Claude workflows pin `--model claude-opus-5 --effort xhigh` so CI does
  not drift when the CLI default changes.
- The workflows carry long comments explaining *why* permissions are scoped as
  they are (e.g. `claude.yml` leaves Bash disabled so Claude pushes a branch and
  returns a prefilled PR link instead of opening the PR itself; the review job
  deliberately does not use the code-review plugin because
  `claude-code-action` ignores `ReportFindings`). Read those comments before
  changing permissions or tool allowlists in either file.
