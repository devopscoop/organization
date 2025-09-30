# Bootstrapping an Organization

## Source Code Management (SCM)

A decentralized organization should have code in multiple online SCM platforms. We chose:

- [gitlab.com](https://gitlab.com/), because they ["been firm believers in remote work, open source, DevSecOps, and iteration."](https://about.gitlab.com/company/)
- [codeberg.org](https://codeberg.org/), because it's ["an open community of free software enthusiasts providing a humane, non-commercial and privacy-friendly alternative to commercial services such as GitHub."](https://docs.codeberg.org/getting-started/what-is-codeberg/)
- [github.com](https://github.com/), because they're currently the most popular platform, and we want to reach as large an audience as possible.

To do this, we:

1. Created these organizations/groups:
   - https://gitlab.com/devopscoop
   - https://codeberg.org/devopscoop
   - https://github.com/devopscoop
1. Created this `organization` git repo locally with:
   ```
   mkdir organization
   cd organization
   git init
   ```
1. Created empty `organization` repositories/projects in each of the SCMs.
1. Ran this snippet to create git remotes for each of the SCMs:
   ```
   for scm in codeberg.org gitlab.com github.com; do
     git remote add "$scm" "git@${scm}:devopscoop/$(basename $PWD).git"
   done
   git remote -v
   ```

To help manage code across multiple remotes, you can put these aliases in your `${HOME}/.gitconfig`:

```
[alias]

        # Pull all remote main branches, and merge them with local main.
        pull-all-main = "!git checkout main && git fetch --all && for remote in $(git remote); do git merge ${remote}/main; done"

        # Push all branches to all remotes
        push-all-branches = "!for remote in $(git remote); do git push ${remote} --all; done"

```

To fetch code from all remotes use:

```
git pull-all-main
```

To push code to all remotes:

```
git push-all-branches
```

## Issue tracking and merge requests

Managing issues across multiple SCMs is undesirable, so we will be using GitLab as our primary for issue tracking and merge requests.

Issues can be filed here: https://gitlab.com/groups/devopscoop/-/issues

## Password Management

[Bitwarden](https://bitwarden.com)

## Email

[Migadu](https://migadu.com/)

## DNS

We're using the domain registrar's default DNS: https://manage30.encirca.com/