# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **GitHub Action** (not a standalone application) that automatically updates draft releases when PRs are merged to maintain a cumulative changelog with automatic versioning.

**Key Concept**: The action groups all vX.Y.Z patch versions under a single draft release titled `vX.Y`, making it easy to accumulate changes before publishing a minor version release.

## Architecture

### Core Implementation: Composite Action

The entire action logic is in `action.yml` (lines 76-161) as a single bash script step. This is a **composite action** that:

1. Creates git tags on merge commits (unless `skip-tagging: true`)
2. Finds or creates draft releases by **title** (vX.Y format)
3. Appends PR entries to the draft release body
4. Updates the tag association and changelog footer

### Key Design Decisions

- **Release Grouping**: Uses release **title** (vX.Y) to group patch versions, not tags
  - Example: Tags v1.2.0, v1.2.1, v1.2.2 all update the same draft release titled "v1.2"
  - The release tag gets updated to the latest version on each merge

- **Idempotent Tagging**: The action safely skips if a tag already exists (line 94-98)
  - Tags point to exact merge commits via `MERGE_COMMIT` SHA

- **Changelog Footer Management**: Automatically updates the "Full Changelog" comparison link (lines 109-113)
  - Old footer is removed before appending new entry
  - Compares against `stable-version` if provided, otherwise shows commits link

## Testing

This action has no automated tests. To test changes:

1. **Test locally with `act`**: Use [nektos/act](https://github.com/nektos/act) to simulate GitHub Actions locally
2. **Test in a fork**: Create a test repository and reference your fork/branch of this action
3. **Manual verification**: Check the action's behavior in `.github/workflows/release.yml` which uses related actions from the same author

## Workflow Integration

The action is designed to work with these companion actions (from the same author):
- `starburst997/auto-version@v1` - Calculates version numbers
- `starburst997/commits-logs@v1` - Alternative changelog generation

See `.github/workflows/release.yml` for this repository's own release workflow.

## Shell Script Notes

The bash script (lines 76-161) uses several critical patterns:

- **`gh` CLI commands**: All GitHub API operations use `gh release` and `gh api`
- **jq parsing**: JSON extraction uses `--jq` flags with `gh` commands
- **Multiline string handling**: Careful use of `printf` and `sed` to preserve formatting
- **Whitespace trimming**: Lines 110-113 clean up bodies before appending

When modifying the bash script:
- Test string manipulation carefully - GitHub release notes formatting is strict
- The `sed` commands removing trailing whitespace are critical for clean output
- Ensure `GITHUB_OUTPUT` format is correct (line 158-160)

## Marketplace Categories

For GitHub Marketplace publishing, appropriate categories would be:
- **Primary**: Continuous integration
- **Secondary**: Utilities, Project management
