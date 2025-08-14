# Auggie Dependabot (Polyglot, AI-assisted upgrades)

Purpose: Provide a reusable GitHub Actions workflow that runs the Auggie CLI on any repository to:
- Detect and apply dependency/version updates across languages and stacks
- Open PRs automatically
- Attempt safe refactors to address breaking changes and prevent regressions
- When a full migration is required, suggest a plan of attack and execute incremental automation steps where possible

This repository contains a reusable workflow and a composite action that other repositories can call via `workflow_call`.

## Quick start (use in a target repository)

1) In your target repo, create a workflow, e.g. `.github/workflows/auggie-upgrades.yml`:

```yaml
name: Auggie Upgrades

on:
  schedule:
    - cron: '0 3 * * 1'   # every Monday 03:00 UTC
  workflow_dispatch:

jobs:
  upgrades:
    uses: <your-org-or-user>/auggie-dependabot/.github/workflows/auggie.yml@v1
    with:
      pr-labels: 'dependencies,auggie'
      pr-reviewers: ''
      group-strategy: 'auto'        # auto|none|ecosystem
      auggie-config-path: '.auggie.yml'
      auggie-run-args: ''
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

2) (Optional) Add an `.auggie.yml` to the target repo to customize behavior (examples below).

3) Create the `OPENAI_API_KEY` secret in the target repo/org if Auggie requires it.

## What the workflow does
- Checks out the caller repository
- Runs the Auggie CLI (polyglot dependency detection)
- Creates/updates branches with changes
- Opens/updates PRs, labels them, and posts migration plans as PR comments when needed

## Inputs (workflow_call)
- `pr-labels`: comma-separated labels to apply to PRs (default: `dependencies,auggie`)
- `pr-reviewers`: comma-separated GitHub usernames to request review (default: none)
- `group-strategy`: `auto`, `none`, or `ecosystem` (default: `auto`)
- `auggie-config-path`: path to config in caller repo (default: `.auggie.yml`)
- `auggie-run-args`: extra CLI flags

## Secrets
- `GITHUB_TOKEN`: write access token to open PRs (GitHub-provided token is sufficient for same-repo operations)
- `OPENAI_API_KEY`: required if Auggie performs AI-driven refactors/migrations

## Example .auggie.yml (caller repo)
```yaml
# How aggressively to upgrade
upgradePolicy:
  securityOnly: false
  allowPrerelease: false
  maxConcurrentPRs: 5

# Grouping and cadence
grouping:
  strategy: auto   # auto|none|ecosystem
  groups:
    - name: react-suite
      match:
        ecosystems: [npm, yarn, pnpm]
        packages: [react, react-dom, @types/react]

# Migration helpers
migrations:
  enableAutoRefactor: true
  enableCodemods: true
  # Add bespoke codemods commands if needed
  codemods: []

# Exclusions
ignore:
  packages: []
```

## Developing this repo
- Reusable workflow: `.github/workflows/auggie.yml`
- Composite action runner: `.github/actions/auggie-run`
- Tag this repository (e.g. `v1`) and reference it from callers as shown above.

## Notes
- This scaffold assumes an `auggie` CLI is available in a container or installer.
- If your environment uses a different install method, adjust `auggie.yml` inputs accordingly.

