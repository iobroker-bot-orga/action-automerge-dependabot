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
        uses: iobroker-bot-orga/action-automerge-dependabot@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Usage

```yaml
steps:
  - name: Auto-merge Dependabot PRs
    uses: iobroker-bot-orga/action-automerge-dependabot@main
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
| `merge-method` | Merge method to use (`merge`, `squash`, or `rebase`) | No | `merge` |
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
    # Merge patch updates for production dependencies
    dependency_type: production
    update_type: "semver:patch"
- match:
    # Merge patch and minor updates for development dependencies
    dependency_type: development
    update_type: "semver:minor"
```

#### Configuration Options

- `dependency_type`: `production`, `development`, or `package-lock`
  - `production` - Direct production dependencies
  - `development` - Development dependencies
  - `package-lock` - Changes only to a package-lock.json file (in any directory)
- `update_type`: `semver:patch`, `semver:minor`, or `semver:major`
  - **Hierarchical matching**: Higher version changes include all lower changes
    - `semver:patch` - Only patch updates (e.g., 1.0.0 → 1.0.1)
    - `semver:minor` - Patch **and** minor updates (e.g., 1.0.0 → 1.0.1 or 1.0.0 → 1.1.0)
    - `semver:major` - Patch, minor, **and** major updates (e.g., 1.0.0 → 1.0.1, 1.0.0 → 1.1.0, or 1.0.0 → 2.0.0)

#### Examples

Allow all patch updates only:
```yaml
- match:
    update_type: "semver:patch"
```

Allow patch and minor updates for development dependencies:
```yaml
- match:
    dependency_type: development
    update_type: "semver:minor"
```

Allow all updates (patch, minor, and major) for development dependencies:
```yaml
- match:
    dependency_type: development
    update_type: "semver:major"
```

Multiple rules for different dependency types:
```yaml
- match:
    dependency_type: production
    update_type: "semver:patch"  # Only patch for production
- match:
    dependency_type: development
    update_type: "semver:minor"  # Patch and minor for development
```

Allow minor and patch updates for package-lock only changes:
```yaml
- match:
    dependency_type: package-lock
    update_type: "semver:minor"  # Patch and minor for package-lock.json only changes
```

**Note**: With hierarchical matching, you typically only need one rule per dependency type. For example, if you specify `semver:minor`, you don't need a separate rule for `semver:patch` since minor already includes patch updates.

#### Package-Lock.json Only Changes

When Dependabot creates a PR that only changes a `package-lock.json` file (no other files modified), the action automatically detects this and treats the dependency type as `package-lock`. This works for `package-lock.json` in any directory (e.g., `./package-lock.json`, `src-admin/package-lock.json`, etc.). This typically happens when a transitive dependency is updated.

**Behavior:**
- If a config file exists with a rule for `dependency_type: package-lock`, that rule is evaluated
- If a config file exists but has no `package-lock` rule, minor and patch updates are merged by default
- If no config file exists, minor and patch updates are merged by default (same as development dependencies)

### Default Rules

If no configuration file is provided, the following default rules apply:

- ✅ **Patch updates**: Always merged (production, development, and package-lock)
- ⚠️ **Minor updates**: Only merged for development dependencies and package-lock only changes
- ❌ **Major updates**: Never merged automatically

## How It Works

1. **Fetch Metadata**: Retrieves information about the Dependabot PR
2. **Extract Info**: Determines the update type (patch/minor/major) and dependency type (production/development)
3. **Read Configuration**: Loads your auto-merge rules from the config file
4. **Evaluate Rules**: Checks if the PR matches any of your rules
5. **Wait for Checks** (optional): Waits for other CI checks to complete
   - Workflows with `success`, `skipped`, or `neutral` conclusions are treated as successful
   - The final result of all checked workflows is logged for transparency
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

## Versioning

The examples in this README use `@main` to always get the latest version. For production use, you may want to:
- Use `@main` for the latest features and fixes
- Use a specific version tag (e.g., `@v1.0.0`) for stability
- Use a major version tag (e.g., `@v1`) for automatic minor and patch updates

Version tags will be created following [semantic versioning](https://semver.org/).

## License

[MIT](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
