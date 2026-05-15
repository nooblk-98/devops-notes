# Auto Release Setup

This project uses a GitHub Actions workflow to automatically create a release with AI-generated release notes on every push to `main`.

## Workflow

Location: `.github/workflows/release-notes.yml`

```yaml
name: Auto Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Delete old release if exists
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        continue-on-error: true
        run: |
          gh release delete "v0.1.${{ github.run_number }}" --yes || true
          git push --delete origin "v0.1.${{ github.run_number }}" || true

      - name: Create GitHub Release
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh release create "v0.1.${{ github.run_number }}" \
            --target "${{ github.sha }}" \
            --title "Release v0.1.${{ github.run_number }}"

      - name: Generate and update release notes
        uses: nooblk-98/copilot-release-notes-action@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          copilot_model: gpt-4o
          tag_name: "v0.1.${{ github.run_number }}"
          target_commitish: "${{ github.sha }}"
```

## How It Works

1. **Trigger:** Every push to `main` starts the workflow.
2. **Checkout:** Full git history is fetched (`fetch-depth: 0`) for accurate changelog generation.
3. **Cleanup:** Any existing release with the same run number is deleted to allow re-runs.
4. **Create Release:** A new GitHub Release is created with tag `v0.1.<run_number>`.
5. **Generate Notes:** The [copilot-release-notes-action](https://github.com/nooblk-98/copilot-release-notes-action) uses GPT-4o to write release notes based on commits since the last release.

## Tag Format

Tags follow the pattern `v0.1.<run_number>` where `run_number` is the GitHub Actions run number, which auto-increments with each workflow execution.

## View Releases

All releases are available at:
`https://github.com/nooblk-98/devops-notes/releases`
