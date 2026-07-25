# Claude GitHub Actions

Our repos run [Claude Code GitHub Actions](https://github.com/anthropics/claude-code-action). This repo carries the two workflow files in [`.github/workflows/`](.github/workflows/), and the same pair is distributed to the other repos in this org, the 6j0-org org, and the 3uzbcqje personal account:

- `claude.yml` — runs Claude when `@claude` is mentioned in an issue, PR comment, or review.
- `claude-code-review.yml` — automatically reviews PRs when they are opened or updated.

Both authenticate with a `CLAUDE_CODE_OAUTH_TOKEN` Actions secret. The workflows fail silently in any repo that can't see that secret, so distribute it as below.

## Create a token

```shell
claude setup-token
```

Requires [Claude Code](https://claude.com/product/claude-code) and a Claude subscription; prints an OAuth token to paste into the prompts below. The `read -rsp` in each snippet keeps the token off screen and out of shell history — substitute your password manager if you prefer, e.g. `TOKEN=$(pass show claude-oauth-token)`.

## Distribute to a personal account's repos

Personal accounts have no account-level Actions secrets, so set the secret per repo (forks are skipped — the workflows aren't distributed there):

```shell
read -rsp "Token: " TOKEN && echo && \
gh repo list 3uzbcqje --limit 200 --json nameWithOwner,isFork \
  --jq '.[] | select(.isFork | not) | .nameWithOwner' | \
while read -r repo; do
  gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo "$repo" --body "$TOKEN" && echo "set: $repo"
done
```

## Distribute organization-wide

Organizations support org-level Actions secrets, so one secret covers every repo:

```shell
read -rsp "Token: " TOKEN && echo && \
for org in devopscoop 6j0-org; do
  gh secret set CLAUDE_CODE_OAUTH_TOKEN --org "$org" --visibility all --body "$TOKEN" && echo "set: $org"
done
```

- Requires org admin and the `admin:org` scope; if `gh` returns a 403, run `gh auth refresh -s admin:org` first.
- `--visibility all` exposes the secret to all current and future repos, private ones included. Use `--visibility selected --repos repo1,repo2,...` to restrict it.
