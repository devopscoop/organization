# AGENTS.md

Instructions for AI coding agents working in this repo.

## Package manifests

This repo ships a `Brewfile` (macOS: `brew bundle`) and a `pkglist.txt` (Arch Linux) that install every CLI tool the repo uses (currently just `git`). Keep them in sync with the docs:

- If a doc starts instructing readers to run a new local tool, add the package to BOTH files, with a comment noting what uses it.
- When a tool stops being used, remove it from both files.
- Hosted services (SCM platforms, password managers, email, DNS) are not installable binaries and don't belong in the manifests.
- Verify package names before adding them: `brew info <formula>` for Homebrew, and the official repos/AUR for Arch. If a package is AUR-only, note that in pkglist.txt's header instructions.
- Update the "Install required packages" section in README.md if the tool list changes.
