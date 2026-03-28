# dvrd/shared-workflows

Reusable GitHub Actions workflows shared across all dvrd product repos (ariel, hrvst, onlead, donostia.ai, newt).

## Available workflows

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `check-branch-name.yml` | `pull_request` | Validates branch naming convention `<type>/<repo>-<issue>` |
| `issue-check.yml` | `pull_request` | Checks PR body references a GitHub issue or Paperclip ID |
| `issue-refs.yml` | `pull_request` | Checks PR contains both `DON-XX` and `#XX` / `Closes #XX` |
| `notify-pr.yml` | `pull_request` | Sends Telegram notification when PR is ready for review |
| `release.yml` | `push` to `main` | Runs semantic release and updates CHANGELOG.md + VERSION |
| `pr-gate.yml` | `workflow_run` | Notifies board only when CI passes AND preview is healthy |

## Usage

### check-branch-name

```yaml
name: Branch naming check
on:
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  check-branch:
    uses: dvrd/shared-workflows/.github/workflows/check-branch-name.yml@main
```

### issue-check

```yaml
name: Issue Check
on:
  pull_request:
    types: [opened, edited, synchronize, reopened]
jobs:
  check:
    uses: dvrd/shared-workflows/.github/workflows/issue-check.yml@main
```

### issue-refs

```yaml
name: Issue References Check
on:
  pull_request:
    types: [opened, edited, synchronize]
jobs:
  check-refs:
    uses: dvrd/shared-workflows/.github/workflows/issue-refs.yml@main
```

### notify-pr

```yaml
name: Notify PR needs review
on:
  pull_request:
    types: [opened, ready_for_review, review_requested]
jobs:
  notify:
    uses: dvrd/shared-workflows/.github/workflows/notify-pr.yml@main
    secrets: inherit
```

### release

```yaml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    uses: dvrd/shared-workflows/.github/workflows/release.yml@main
    secrets: inherit
```

### pr-gate

Replace `ariel.donostia.ai` with the product's domain.

```yaml
name: PR Gate
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches-ignore:
      - main
      - master
permissions:
  pull-requests: read
jobs:
  gate:
    if: >
      github.event.workflow_run.event == 'pull_request' &&
      github.event.workflow_run.conclusion == 'success'
    uses: dvrd/shared-workflows/.github/workflows/pr-gate.yml@main
    with:
      product_domain: ariel.donostia.ai
      health_path: /health
    secrets: inherit
```

## Required secrets

All product repos must have these repository secrets configured:

- `TELEGRAM_BOT_TOKEN` — bot token for notifications
- `TELEGRAM_CHAT_ID` — chat ID for notifications
