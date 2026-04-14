# Now Building Block Updater

<p align="left">
  <img src="https://img.shields.io/badge/version-v1.0.0-0A84FF?style=for-the-badge" alt="Version">
  <img src="https://img.shields.io/badge/github%20action-composite-FF6B00?style=for-the-badge" alt="Composite Action">
  <img src="https://img.shields.io/badge/python-3.10%2B-2E8B57?style=for-the-badge" alt="Python 3.10+">
  <img src="https://img.shields.io/badge/license-MIT-8A2BE2?style=for-the-badge" alt="MIT License">
</p>

Turn your repository activity into a clean, auto-updating "Now Building" section in your README.

This project is for people who want a portfolio-style monthly update without manually editing markdown tables every month.

## Who should use this

- Solo builders who want a visible monthly progress section in README.
- Student developers who want public proof of consistency.
- Teams that want lightweight status storytelling in an open-source repo.

## What you get

| You want | This action gives you |
|---|---|
| Hands-off monthly updates | Scheduled workflow that refreshes the block automatically |
| Clean formatting | Managed markers and table generation |
| Flexibility | Public-only mode or optional private-repo mode |
| Low setup effort | Copy-paste workflow + simple inputs |

## 5-minute setup

1. Add these markers where you want the generated section in your README (or skip this and it will append at the end).

```md
<!-- NOW_BUILDING:START -->
<!-- NOW_BUILDING:END -->
```

2. Create a workflow file in your target repository: `.github/workflows/update-now-building.yml`.

3. Paste this workflow:

```yaml
name: Update Now Building

on:
  schedule:
    - cron: "5 1 1 * *"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  update-now-building:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Update README block
        uses: TridentifyIshaan/now-building-block-updater@v1
        with:
          username: YOUR_GITHUB_USERNAME
          github-token: ${{ secrets.NOW_BUILDING_TOKEN }}
          readme-path: README.md
          months: "3"
          rows-per-month: "2"
          include-private: "false"
          include-forks: "false"
          include-archived: "false"
          note: "Updating this block every month."

      - name: Commit changes
        run: |
          if git diff --quiet; then
            exit 0
          fi
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add README.md
          git commit -m "chore: update now building block"
          git push
```

4. Add repository secret `NOW_BUILDING_TOKEN`.

5. Run workflow manually once from the Actions tab to verify output.

## Token setup (important)

- Public repositories only:
  Use `GITHUB_TOKEN` or a minimal token as `NOW_BUILDING_TOKEN`.
- Private repositories included:
  Use a token from the same account as `username` with access to required private repositories.

If `include-private` is false, private repositories are ignored.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `username` | yes | - | GitHub username to analyze |
| `github-token` | no | `""` | Token with access to repositories you want analyzed |
| `readme-path` | no | `README.md` | README path to update |
| `months` | no | `3` | Number of recent months to include |
| `rows-per-month` | no | `2` | Number of rows to render per month |
| `include-private` | no | `false` | Include private repos when token has access |
| `include-forks` | no | `false` | Include fork repositories |
| `include-archived` | no | `false` | Include archived repositories |
| `note` | no | `Updating this block every month.` | Footer note under the generated table |

## Common configuration patterns

Public-only profile README:

```yaml
with:
  username: YOUR_GITHUB_USERNAME
  include-private: "false"
  months: "3"
```

Broader timeline + more rows per month:

```yaml
with:
  months: "6"
  rows-per-month: "3"
```

Include private repositories:

```yaml
with:
  include-private: "true"
  github-token: ${{ secrets.NOW_BUILDING_TOKEN }}
```

## What gets generated

The managed output looks like this:

```md
<!-- NOW_BUILDING:START -->
## Now Building

| Month | Current Build Track | Shipping Goal |
|---|---|---|
| Apr 2026 | repo-a: core feature development | Ship cleaner milestones with stronger reliability |
|  | repo-b: integration and stability improvements | Improve release readiness by tightening tests and workflows |
| Mar 2026 | repo-c: delivery-focused polish work | Close the month with production-ready demos and clearer docs |

<sub> Updating this block every month.</sub>
<!-- NOW_BUILDING:END -->
```

If markers do not exist, the block is appended to the end of the README.

## Troubleshooting

`No changes needed.`
The generated content is identical to what is already in your README.

`README path does not exist`
Set `readme-path` correctly (for example `profile/README.md` for nested docs).

Private repos are not showing up:
- Confirm `include-private: "true"`.
- Confirm token is valid and has access to those private repositories.
- Prefer a token owned by the same GitHub account in `username`.

Workflow updates README but does not push:
- Ensure workflow permissions include `contents: write`.
- Ensure commit step runs only when there are diffs.

## Local CLI usage (optional)

Use this when you want to preview locally before setting up Actions.

```bash
python -m pip install -e .
now-building-updater \
  --username YOUR_GITHUB_USERNAME \
  --readme-path README.md \
  --months 3 \
  --rows-per-month 2
```

Common options:

- `--include-private`
- `--include-forks`
- `--include-archived`
- `--note "Custom note"`
- `--token-env GITHUB_TOKEN`
- `--dry-run`

## For maintainers of this repository

1. Finalize `action.yml`, `README.md`, and package version.
2. Push to `mainstream`.
3. Create and push tag `v1.0.0`.
4. Move major tag `v1` to the same commit.
5. Publish a GitHub Release with notes.

```bash
git tag -a v1.0.0 -m "v1.0.0"
git tag -f v1
git push origin v1.0.0
git push origin v1 --force
```
