# adzaps/.github

Organization-wide GitHub Actions for Claude-powered code review and interactive assistance.

> **This repo must remain public.** GitHub requires the `.github` repo to be public for workflow templates to appear in the org's "New workflow" UI. There is no sensitive content here -- secrets are stored in org settings, not in this repo.

## Workflows

| Workflow | Trigger | Deployment | Purpose |
|---|---|---|---|
| **Claude Review** | `pull_request` | Org ruleset (zero-config) | Automated PR review |
| **Claude Code** | `issue_comment`, `pull_request_review`, `issues` | Per-repo workflow template | Interactive `@claude` assistant |

### Claude Review

Automatically reviews pull requests when they are opened, updated, or marked ready for review. Posts inline comments focusing on code quality, bugs, security, and performance.

**Deployment**: Add as a required workflow via an org-level ruleset. Every targeted repo gets reviews with zero per-repo configuration.

### Claude Code

Responds to `@claude` mentions in issue comments, PR comments, and PR reviews. Can answer questions, suggest fixes, and interact conversationally.

**Deployment**: Each repo adds a thin caller workflow. Use the workflow template from the org's "New workflow" UI, or create `.github/workflows/claude-code.yml` manually:

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

Available when using the reusable workflow caller approach (not available for required workflow/ruleset deployments):

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

For repos using Claude Review via org rulesets (zero-config path), custom instructions are not available -- they get the default review prompt. Switch to the reusable workflow caller approach if custom instructions are needed.

## Opt-Out

Repos can disable either workflow by setting a **repository variable** to `true`:

| Variable | Effect |
|---|---|
| `CLAUDE_REVIEW_DISABLED` | Skips automatic PR reviews |
| `CLAUDE_CODE_DISABLED` | Skips `@claude` responses |

Set via: **Repo Settings > Secrets and variables > Actions > Variables**

When a required workflow job is skipped via `if`, GitHub reports it as "skipped" which satisfies the required check -- merging is not blocked.

## Secrets

Set `CLAUDE_CODE_OAUTH_TOKEN` as an **org-level secret** (Organization Settings > Secrets and variables > Actions > Secrets).

- Automatically available to required workflows (rulesets)
- For reusable workflow callers, use `secrets: inherit` -- no need to name secrets individually

## Verification Checklist

After pushing this repo and configuring the org:

1. **Org ruleset**: Create a ruleset pointing to `claude-review.yml` -> open a PR in any targeted repo -> verify Claude review posts
2. **Workflow template**: Add Claude Code via the "New workflow" UI -> comment `@claude hello` on an issue -> verify Claude responds
3. **Review opt-out**: Set `CLAUDE_REVIEW_DISABLED=true` on a repo -> open a PR -> verify review is skipped
4. **Code opt-out**: Set `CLAUDE_CODE_DISABLED=true` on a repo -> mention `@claude` -> verify it's skipped
5. **Custom instructions**: Use reusable workflow caller with `custom_instructions` -> verify instructions are reflected in behavior
