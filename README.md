# Auto-Merge Dependabot PRs Action

Automatically merge Dependabot PRs based on configurable rules. This action evaluates Dependabot PRs against your criteria and merges them automatically if they meet your requirements.

## Features

- ✅ **Flexible Configuration**: Define merge rules based on dependency type and update type
- ✅ **Safe Defaults**: Sensible default rules when no configuration is provided
- ✅ **CI Integration**: Optionally wait for other checks to complete before merging
- ✅ **Multiple Merge Methods**: Support for merge, squash, and rebase
- ✅ **Detailed Logging**: Clear output showing why a PR was or wasn't merged

## Usage

### Basic Usage

Create a workflow file (e.g., `.github/workflows/auto-merge.yml`) in your repository:

```yaml
name: Auto-Merge Dependabot PRs

on:
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    
    permissions:
      contents: write
      pull-requests: write
      
    steps:
      - name: Auto-merge Dependabot PRs
        uses: iobroker-bot-orga/action-automerge-dependabot@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Usage

```yaml
steps:
  - name: Auto-merge Dependabot PRs
    uses: iobroker-bot-orga/action-automerge-dependabot@v1
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      config-file-path: '.github/auto-merge.yml'
      merge-method: 'squash'
      wait-for-checks: 'true'
      max-wait-time: '3600'
```

## Configuration

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token with permissions to merge PRs | Yes | `${{ github.token }}` |
| `config-file-path` | Path to the auto-merge configuration file | No | `.github/auto-merge.yml` |
| `merge-method` | Merge method to use (`merge`, `squash`, or `rebase`) | No | `squash` |
| `wait-for-checks` | Wait for other checks to complete before merging | No | `true` |
| `max-wait-time` | Maximum time to wait for checks in seconds | No | `3600` |

### Outputs

| Output | Description |
|--------|-------------|
| `should-merge` | Whether the PR should be merged based on configuration |
| `match-reason` | Reason for the merge decision |
| `checks-passed` | Whether all CI checks passed |

### Auto-Merge Configuration File

Create a file at `.github/auto-merge.yml` in your repository to define merge rules:

```yaml
# Recommended configuration
- match:
    # Only merge patches for production dependencies
    dependency_type: production
    update_type: "semver:patch"
- match:
    # Development dependencies can have minor updates too
    dependency_type: development
    update_type: "semver:minor"
```

#### Configuration Options

- `dependency_type`: `production` or `development`
- `update_type`: `semver:patch`, `semver:minor`, or `semver:major`

#### Examples

Allow all patch updates:
```yaml
- match:
    update_type: "semver:patch"
```

Allow minor updates for development dependencies:
```yaml
- match:
    dependency_type: development
    update_type: "semver:minor"
```

Multiple rules:
```yaml
- match:
    dependency_type: production
    update_type: "semver:patch"
- match:
    dependency_type: development
    update_type: "semver:minor"
- match:
    dependency_type: development
    update_type: "semver:patch"
```

### Default Rules

If no configuration file is provided, the following default rules apply:

- ✅ **Patch updates**: Always merged (both production and development)
- ⚠️ **Minor updates**: Only merged for development dependencies
- ❌ **Major updates**: Never merged automatically

## How It Works

1. **Fetch Metadata**: Retrieves information about the Dependabot PR
2. **Extract Info**: Determines the update type (patch/minor/major) and dependency type (production/development)
3. **Read Configuration**: Loads your auto-merge rules from the config file
4. **Evaluate Rules**: Checks if the PR matches any of your rules
5. **Wait for Checks** (optional): Waits for other CI checks to complete
6. **Merge**: Automatically merges the PR if all conditions are met

## Security Considerations

⚠️ **Important**: This action uses `pull_request_target` which runs in the context of the base branch. This is by design to allow the action to merge PRs, but means:

- The action has access to repository secrets
- You should only use this action with Dependabot PRs (enforced by the `if: github.actor == 'dependabot[bot]'` condition)
- Do not build or run untrusted code from the PR in this workflow

## Requirements

- GitHub Actions workflow with `pull_request_target` event
- Permissions: `contents: write` and `pull-requests: write`
- Python 3 with PyYAML (available by default on `ubuntu-latest` runners)

## Example Repository Setup

See the [example workflow](.github/workflows/auto-merge-example.yml) and [example configuration](.github/auto-merge.example.yml) files in this repository.

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
