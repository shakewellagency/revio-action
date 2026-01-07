# Revio AI Code Review Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Revio%20AI%20Review-blue?logo=github)](https://github.com/marketplace/actions/revio-ai-code-review)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Automated AI-powered code review for your pull requests. Get instant feedback on code quality, security vulnerabilities, and best practices.

## Features

- **AI-Powered Analysis** - Leverages Claude AI for intelligent code review
- **Inline Annotations** - Issues appear directly on changed lines in GitHub
- **Security Scanning** - SARIF output integrates with GitHub Security tab
- **Configurable Strictness** - Fail builds on critical issues or any issues
- **Fast & Lightweight** - Composite action with minimal overhead

## Quick Start

Add this workflow to your repository:

```yaml
# .github/workflows/revio-review.yml
name: Revio AI Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: shakewellagency/revio-action@v1
        with:
          api_token: ${{ secrets.REVIO_API_TOKEN }}
```

## Getting Your API Token

1. Sign up at [revio.shakewell.agency](https://revio.shakewell.agency)
2. Go to **Settings** > **API Tokens**
3. Generate a new token with `reviews:write` scope
4. Add it to your repository secrets as `REVIO_API_TOKEN`

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `api_token` | Revio API token | Yes | - |
| `base_branch` | Base branch to compare against | No | PR base branch |
| `fail_on` | When to fail: `none`, `critical`, `any` | No | `none` |
| `exclude` | File patterns to exclude (comma-separated) | No | - |
| `include` | File patterns to include (comma-separated) | No | All files |
| `output_format` | Format: `annotations`, `sarif`, `summary` | No | `annotations` |
| `post_comment` | Post summary as PR comment | No | `true` |
| `working_directory` | Working directory for review | No | `.` |

## Outputs

| Output | Description |
|--------|-------------|
| `review_url` | URL to full review in Revio dashboard |
| `verdict` | Review verdict: `approved`, `needs_changes`, `critical` |
| `issues_count` | Total number of issues found |
| `critical_count` | Number of critical issues |

## Examples

### Fail on Critical Issues

```yaml
- uses: shakewellagency/revio-action@v1
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}
    fail_on: critical
```

### Exclude Test Files

```yaml
- uses: shakewellagency/revio-action@v1
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}
    exclude: '**/*.test.js,**/*.spec.ts,**/tests/**'
```

### SARIF Output for Security Tab

```yaml
- uses: shakewellagency/revio-action@v1
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}
    output_format: sarif
```

This uploads results to GitHub's Security tab under Code Scanning alerts.

### Review Only Specific Files

```yaml
- uses: shakewellagency/revio-action@v1
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}
    include: 'src/**/*.ts,lib/**/*.ts'
```

### Monorepo - Review Specific Package

```yaml
- uses: shakewellagency/revio-action@v1
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}
    working_directory: 'packages/api'
```

### Use Outputs in Subsequent Steps

```yaml
- uses: shakewellagency/revio-action@v1
  id: review
  with:
    api_token: ${{ secrets.REVIO_API_TOKEN }}

- name: Check verdict
  if: steps.review.outputs.verdict == 'critical'
  run: |
    echo "Critical issues found: ${{ steps.review.outputs.critical_count }}"
    echo "View full review: ${{ steps.review.outputs.review_url }}"
```

### Complete Production Example

```yaml
name: CI

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write
  security-events: write  # For SARIF upload

jobs:
  review:
    name: AI Code Review
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: shakewellagency/revio-action@v1
        id: revio
        with:
          api_token: ${{ secrets.REVIO_API_TOKEN }}
          fail_on: critical
          exclude: '*.min.js,vendor/**,node_modules/**'
          output_format: annotations

      - name: Review Summary
        if: always()
        run: |
          echo "Verdict: ${{ steps.revio.outputs.verdict }}"
          echo "Issues: ${{ steps.revio.outputs.issues_count }}"
          echo "Critical: ${{ steps.revio.outputs.critical_count }}"
```

## Comparison with Webhook Reviews

| Feature | GitHub Action | Webhook (Default) |
|---------|--------------|-------------------|
| Trigger | On PR events via workflow | Automatic via GitHub App |
| Setup | Add workflow file | Install GitHub App |
| CI Integration | Native (fail builds) | Separate |
| SARIF Support | Yes | No |
| Customization | Full control | Dashboard settings |

**Recommendation:** Use both! The GitHub App provides automatic reviews, while the Action gives you CI/CD integration with build gates.

## Troubleshooting

### "Authentication failed"
- Verify your `REVIO_API_TOKEN` secret is set correctly
- Check the token hasn't expired
- Ensure the token has `reviews:write` scope

### "No changes to review"
- Make sure you're using `fetch-depth: 0` in checkout
- Verify the base branch exists

### Action is slow
- The first run installs the CLI (cached in subsequent runs)
- Large PRs take longer to analyze

## Support

- **Documentation:** [docs.revio.shakewell.agency](https://docs.revio.shakewell.agency)
- **Issues:** [GitHub Issues](https://github.com/shakewellagency/revio-action/issues)
- **Email:** support@shakewell.agency

## License

MIT License - see [LICENSE](LICENSE) for details.
