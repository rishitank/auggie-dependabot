# Auggie Dependabot (Polyglot, AI-assisted upgrades)

Purpose: Provide a reusable GitHub Actions workflow that runs the Auggie CLI on any repository to:
- Detect and apply dependency/version updates across languages and stacks
- Open PRs automatically
- Attempt safe refactors to address breaking changes and prevent regressions
- When a full migration is required, suggest a plan of attack and execute incremental automation steps where possible

This repository contains a reusable workflow that other repositories can call via `workflow_call` (no copy/paste needed).

## Quick start (use in a target repository)

1) In your target repo, create a minimal workflow that calls this reusable workflow (no copy/paste of logic):

```yaml
name: Auggie Upgrades

on:
  schedule:
    - cron: '0 3 * * 1'   # every Monday 03:00 UTC
  workflow_dispatch:

jobs:
  upgrades:
    uses: rishitank/auggie-dependabot/.github/workflows/auggie.yml@v1
    with:
      pr-labels: 'dependencies,auggie'
      pr-reviewers: ''
      auggie-config-path: '.auggie.yml'
      auggie-run-args: '--compact'
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      AUGMENT_SESSION_AUTH: ${{ secrets.AUGMENT_SESSION_AUTH }}
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
- `AUGMENT_SESSION_AUTH`: required to authenticate Auggie with Augment services. Store this value as a repository or organization secret and pass it through in the workflow `secrets` block.

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
- Reusable workflow entrypoint: `.github/workflows/auggie.yml`
- Tag this repository (e.g. `v1`) and reference it from callers as shown above.
- Self-test workflow: `.github/workflows/self-test.yml` (uses the reusable workflow in-place; requires AUGMENT_SESSION_AUTH secret in this repo)

## Notes
- The Auggie CLI is published on npm as `auggie`. This repository’s reusable workflow installs it via `npm i -g auggie` or uses `npx auggie`.
- Provide `AUGMENT_SESSION_AUTH` in the caller repo/org secrets to enable Auggie’s authenticated operations.
- If your environment uses a different install method, adjust `.github/workflows/auggie.yml` accordingly.

