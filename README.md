# Todo Tree Action

A GitHub Action that scans your pull requests for TODO comments and posts a summary as a PR comment.

## Features

- Scans code for TODO, FIXME, BUG, and custom tags
- **Scan only changed files** - Focus on files modified in the PR
- **Show only new TODOs** - Filter to TODOs introduced in this PR (not in base branch)
- Posts a formatted summary comment on PRs
- GitHub annotations on specific lines
- Fail conditions (fail on TODOs, FIXME/BUG, or max count)
- Updates existing comments on subsequent runs
- Configurable file patterns and paths
- Cross-platform support (Linux x86_64/aarch64, macOS x86_64/aarch64)

## Quick Start

Add this to your workflow file (e.g., `.github/workflows/todo-tree.yml`):

```yaml
name: Todo Tree

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  scan-todos:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0  # Required for changed-only and new-only modes

      - name: Scan for TODOs
        uses: alexandretrotel/todo-tree-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Usage Examples

### Scan Only Changed Files

Only scan files that were modified in the PR:

```yaml
- uses: alexandretrotel/todo-tree-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    changed-only: true
```

### Show Only New TODOs

Only show TODOs that are new in this PR (not present in the base branch):

```yaml
- uses: alexandretrotel/todo-tree-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    new-only: true
```

### Fail on Critical TODOs

Fail the workflow if FIXME or BUG comments are found:

```yaml
- uses: alexandretrotel/todo-tree-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    fail-on-fixme: true
```

### Limit Maximum TODOs

Fail if there are more than 10 TODOs:

```yaml
- uses: alexandretrotel/todo-tree-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    max-todos: 10
```

### Custom Tags and File Patterns

Scan specific file types for custom tags:

```yaml
- uses: alexandretrotel/todo-tree-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    tags: "TODO,FIXME,HACK,XXX,OPTIMIZE"
    include-patterns: "*.rs,*.ts,*.tsx"
    exclude-patterns: "*.min.js,dist/**,vendor/**"
```

### Full Example with All Options

```yaml
name: Todo Tree

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  scan-todos:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - name: Scan for TODOs
        id: todos
        uses: alexandretrotel/todo-tree-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          path: "src"
          tags: "TODO,FIXME,BUG,HACK"
          include-patterns: "*.rs,*.ts"
          exclude-patterns: "*.test.ts"
          changed-only: true
          new-only: false
          fail-on-todos: false
          fail-on-fixme: true
          max-todos: 50
          show-annotations: true
          max-annotations: 50
          post-comment: true
          comment-header: "## TODO Report"

      - name: Use outputs
        run: |
          echo "Total TODOs: ${{ steps.todos.outputs.total }}"
          echo "Files with TODOs: ${{ steps.todos.outputs.files_count }}"
          echo "Has TODOs: ${{ steps.todos.outputs.has_todos }}"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token for posting comments | Yes | `${{ github.token }}` |
| `path` | Path to scan | No | `.` |
| `tags` | Comma-separated tags to search for | No | `TODO,FIXME,BUG` |
| `include-patterns` | Comma-separated file patterns to include | No | - |
| `exclude-patterns` | Comma-separated file patterns to exclude | No | - |
| `changed-only` | Only scan files changed in this PR | No | `false` |
| `new-only` | Only show NEW TODOs (not in base branch) | No | `false` |
| `fail-on-todos` | Fail if any TODOs are found | No | `false` |
| `fail-on-fixme` | Fail if FIXME or BUG comments are found | No | `false` |
| `max-todos` | Maximum TODOs allowed before failing | No | - |
| `show-annotations` | Show TODOs as GitHub annotations | No | `true` |
| `max-annotations` | Maximum annotations to show (limit: 50) | No | `50` |
| `post-comment` | Post a summary comment on the PR | No | `true` |
| `comment-header` | Custom header for the PR comment | No | `## TODO Summary` |

## Outputs

| Output | Description |
|--------|-------------|
| `total` | Total number of TODOs found |
| `files_count` | Number of files containing TODOs |
| `has_todos` | Whether any TODOs were found (`true`/`false`) |
| `json` | Full JSON output from todo-tree |

## Example Output

The action posts a comment like this on your PR:

```markdown
## TODO Summary

> Showing only **new** TODOs introduced in this PR

Found **3** TODOs in **2** files:

<details>
<summary>Summary by tag</summary>

| Tag | Count |
|-----|-------|
| TODO | 2 |
| FIXME | 1 |

</details>

<details open>
<summary>Details by file</summary>

#### `src/main.rs`
- 🔵 **TODO** (L42): Implement error handling
- 🔴 **FIXME** (L87): This is a workaround

#### `src/lib.rs`
- 🟡 **TODO** (L15): Add unit tests

</details>
```

## Requirements

- **fetch-depth: 0** - Required in `actions/checkout` for `changed-only` and `new-only` modes
- **permissions** - Requires `pull-requests: write` to post comments

## How It Works

1. **Installation**: Downloads the appropriate `todo-tree` binary for the runner's architecture
2. **Scanning**: Runs `todo-tree` with the specified options
3. **Filtering**: Optionally filters to changed files or new TODOs only
4. **Annotations**: Outputs GitHub annotations for each TODO
5. **Comment**: Posts or updates a summary comment on the PR
6. **Fail Check**: Exits with error if fail conditions are met

## License

MIT
