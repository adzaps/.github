# adzaps/.github

Organization-wide GitHub Actions for Claude-powered code review and interactive assistance.

> **This repo must remain public.** GitHub requires the `.github` repo to be public for workflow templates to appear in the org's "New workflow" UI. There is no sensitive content here -- secrets are stored in org settings, not in this repo.

## Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| **Claude Review** | `pull_request` | Automated PR review |
| **Claude Code** | `issue_comment`, `pull_request_review`, `issues` | Interactive `@claude` assistant |

Both workflows are deployed per-repo. Use the workflow templates from the org's "New workflow" UI, or create the caller workflows manually.

### Claude Review

Automatically reviews pull requests when they are opened, updated, or marked ready for review. Posts inline comments focusing on code quality, bugs, security, and performance.

**Deployment**: Add via the "New workflow" UI (Actions > New workflow > "By adzaps"), or create `.github/workflows/claude-review.yml` manually:

```yaml
name: Claude Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  review:
    uses: adzaps/.github/.github/workflows/claude-review.yml@main
    secrets: inherit
```

### Claude Code

Responds to `@claude` mentions in issue comments, PR comments, and PR reviews. Can answer questions, suggest fixes, and interact conversationally.

**Deployment**: Add via the "New workflow" UI, or create `.github/workflows/claude-code.yml` manually:

```yaml
name: Claude Code

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    uses: adzaps/.github/.github/workflows/claude-code.yml@main
    secrets: inherit
```

## Custom Instructions

Pass custom instructions via the `custom_instructions` input when calling the reusable workflow:

```yaml
jobs:
  review:
    uses: adzaps/.github/.github/workflows/claude-review.yml@main
    with:
      custom_instructions: |
        Follow our TypeScript coding standards.
        Always suggest unit tests for new functions.
    secrets: inherit
```

```yaml
jobs:
  claude:
    uses: adzaps/.github/.github/workflows/claude-code.yml@main
    with:
      custom_instructions: |
        You are helping with a Python Django project.
        Follow PEP 8 conventions.
    secrets: inherit
```

## Secrets

Set `CLAUDE_CODE_OAUTH_TOKEN` as an **org-level secret** (Organization Settings > Secrets and variables > Actions > Secrets). Caller workflows use `secrets: inherit` to pass it through automatically.

## Verification Checklist

After pushing this repo:

1. **Claude Review**: Add the workflow template to a repo -> open a PR -> verify Claude review posts
2. **Claude Code**: Add the workflow template to a repo -> comment `@claude hello` on an issue -> verify Claude responds
3. **Review opt-out**: Set `CLAUDE_REVIEW_DISABLED=true` on a repo -> open a PR -> verify review is skipped
4. **Code opt-out**: Set `CLAUDE_CODE_DISABLED=true` on a repo -> mention `@claude` -> verify it's skipped
5. **Custom instructions**: Use caller with `custom_instructions` -> verify instructions are reflected in behavior
