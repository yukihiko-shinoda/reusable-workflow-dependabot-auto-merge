# Reusable Workflow Dependabot Auto-merge

[![Dependabot](https://flat.badgen.net/github/dependabot/yukihiko-shinoda/reusable-workflow-dependabot-auto-merge?icon=dependabot)](https://github.com/yukihiko-shinoda/reusable-workflow-dependabot-auto-merge/security/dependabot)

The GitHub Actions reusable workflow that enables auto-merge on pull requests created by Dependabot.

## Advantage

1. Eliminates duplicated auto-merge configuration
2. Merges made by the workflow trigger workflows on push

### 1. Eliminates duplicated auto-merge configuration

Without a reusable workflow, each repository must maintain the same merge conditions individually.
A single `uses:` reference replaces that duplication.

### 2. Merges made by the workflow trigger workflows on push

Merges done with `GITHUB_TOKEN` don't trigger workflows on push to the default branch.
This workflow uses a GitHub App token, so those workflows run as usual.

## Quickstart

### 1. Set up the caller repository

> [!WARNING]
> Require status checks on the default branch before adding this workflow.
> Without required checks, `gh pr merge --auto` merges a pull request immediately, without waiting for CI.

1. Create a GitHub App with these repository permissions, and install it on the caller repository:
   - Contents: Read and write
   - Pull requests: Read and write
   - Workflows: Read and write
2. Register `AUTO_MERGE_APP_CLIENT_ID` and `AUTO_MERGE_APP_PRIVATE_KEY` as **Dependabot secrets**.
3. Enable **Allow auto-merge** and **Allow merge commits** in the repository settings.
4. Add a ruleset or branch protection rule that requires status checks on the default branch.

### 2. Add a workflow file to the caller repository

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: Dependabot auto-merge
on:
  pull_request:
    branches: [main]
permissions:
  contents: read
  pull-requests: read
jobs:
  auto-merge:
    if: github.event.pull_request.user.login == 'dependabot[bot]'
    uses: yukihiko-shinoda/reusable-workflow-dependabot-auto-merge/.github/workflows/workflow.yml@v1
    secrets:
      AUTO_MERGE_APP_CLIENT_ID: ${{ secrets.AUTO_MERGE_APP_CLIENT_ID }}
      AUTO_MERGE_APP_PRIVATE_KEY: ${{ secrets.AUTO_MERGE_APP_PRIVATE_KEY }}
```

<!-- markdownlint-disable-next-line no-trailing-punctuation -->
## How do I...

### How do I know which updates are merged?

| New version | Update type | Result |
| --- | --- | --- |
| 14 digits (e.g. `20261026205000`), not grouped | Any | Auto-merge is enabled |
| Any | Not major | Auto-merge is enabled |
| Not 14 digits, or grouped | Major | Skipped, merge it manually |

Dependabot reports date-based Docker tags as major updates, so versions of 14 digits don't use the update type.

Grouped updates always use the update type, so a date-based Docker tag in a group has to be merged manually.

## API

### Secrets

| Name | Required | Description |
| --- | --- | --- |
| `AUTO_MERGE_APP_CLIENT_ID` | Yes | Client ID of the GitHub App which enables auto-merge |
| `AUTO_MERGE_APP_PRIVATE_KEY` | Yes | Private key of the GitHub App which enables auto-merge |
