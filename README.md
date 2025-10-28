# Update Draft Release

A GitHub Action that automatically updates draft releases when PRs are merged, maintaining a cumulative changelog with automatic versioning.

## Features

- **Cumulative Changelog**: Automatically appends PR entries to existing draft releases
- **Automatic Tagging**: Tags merge commits with version numbers
- **Smart Versioning**: Groups changes by minor version (vX.Y)
- **Full Changelog Links**: Generates comparison URLs between versions
- **Idempotent**: Safe to run multiple times, skips existing tags

## Quick Start

```yaml
name: Update Draft Release
on:
  pull_request:
    types: [closed]
    branches: [dev, main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get version
        id: version
        run: |
          # Your versioning logic here
          echo "future-version=1.2.3" >> $GITHUB_OUTPUT
          echo "stable-version=1.2.0" >> $GITHUB_OUTPUT

      - name: Update draft release
        uses: starburst997/draft-release@v1
        with:
          future-version: ${{ steps.version.outputs.future-version }}
          stable-version: ${{ steps.version.outputs.stable-version }}
```

## How It Works

1. **PR Merged**: When a PR is merged to your target branch
2. **Tag Created**: Creates a git tag (e.g., `v1.2.3`) on the merge commit
3. **Find/Create Release**: Looks for an existing draft release with title `v1.2`
4. **Update Changelog**: Appends the PR info to the draft release body
5. **Update Tag**: Associates the release with the latest version tag

The action groups all v1.2.x versions under a single draft release titled `v1.2`, making it easy to accumulate changes before publishing.

## Example Output

Draft release titled **v1.2** with body:

```markdown
## What's Changed

- Add user authentication by @alice in https://github.com/owner/repo/pull/42
- Fix login redirect by @bob in https://github.com/owner/repo/pull/43
- Update dependencies by @charlie in https://github.com/owner/repo/pull/44

**Full Changelog**: https://github.com/owner/repo/compare/v1.1.0...v1.2.3
```

## Inputs

| Input              | Required | Default               | Description                                          |
| ------------------ | -------- | --------------------- | ---------------------------------------------------- |
| `token`            | No       | `${{ github.token }}` | GitHub token with repo and contents permissions      |
| `future-version`   | Yes      | -                     | Version for the new tag (without 'v', e.g., `1.2.3`) |
| `stable-version`   | No       | `''`                  | Latest stable version for changelog comparison       |
| `pr-title`         | No       | Auto                  | Pull request title                                   |
| `pr-number`        | No       | Auto                  | Pull request number                                  |
| `pr-url`           | No       | Auto                  | Pull request URL                                     |
| `pr-user`          | No       | Auto                  | Pull request author                                  |
| `merge-commit-sha` | No       | Auto                  | Merge commit to tag                                  |
| `repository`       | No       | Auto                  | Repository in `owner/repo` format                    |
| `skip-tagging`     | No       | `false`               | Skip git tag creation                                |

## Outputs

| Output          | Description                                          |
| --------------- | ---------------------------------------------------- |
| `release-tag`   | The tag that was created or updated (e.g., `v1.2.3`) |
| `release-title` | The release title (e.g., `v1.2`)                     |
| `release-url`   | URL to the draft release                             |

## Integration with Auto-Version

Works seamlessly with [starburst997/auto-version](https://github.com/starburst997/auto-version):

```yaml
- name: Calculate version
  id: version
  uses: starburst997/auto-version@v1

- name: Update draft release
  uses: starburst997/draft-release@v1
  with:
    future-version: ${{ steps.version.outputs.future-version }}
    stable-version: ${{ steps.version.outputs.stable-version }}
```

## Advanced Usage

### Skip Tagging

If you're managing tags separately:

```yaml
- uses: starburst997/draft-release@v1
  with:
    future-version: 1.2.3
    skip-tagging: true
```

### Custom PR Information

Override PR details for custom workflows:

```yaml
- uses: starburst997/draft-release@v1
  with:
    future-version: 1.2.3
    pr-title: "Custom change description"
    pr-user: "bot-user"
    pr-url: "https://example.com/custom-link"
```

## Requirements

- GitHub CLI (`gh`) is available in GitHub Actions runners by default
- Repository must have `fetch-depth: 0` if using git tags
- Token needs `contents: write` permission for creating tags and releases

## License

MIT - See [LICENSE](LICENSE) for details

## Author

Created by [JD Boivin](https://github.com/starburst997)
