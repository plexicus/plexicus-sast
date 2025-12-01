# TODO UPDATE THIS FILE
# Plexalyzer Action

Official GitHub Action for COVULOR cloud service by Plexicus. Analyze your code directly in your Pull Requests.

## Quick Start

Add this workflow to your repository:

```yaml
name: PLEXALYZER Analysis
on:
  pull_request:
    types: [opened, synchronize]
    branches:
      - <BRANCH>

  push:
    branches:
      - <BRANCH>

jobs:
  analyze:
    if: >
      (github.event_name == 'pull_request' && !startsWith(github.head_ref, 'Plexicus-AI-Remediation-')) ||
      github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get Changed Files in PR
        if: github.event_name == 'pull_request'
        shell: bash
        run: |
          echo "Analyzing changed files in the PR..."
          changed_files=$(git diff --name-only "${{ github.event.pull_request.base.sha }}" "${{ github.event.pull_request.head.sha }}")
          echo "$changed_files" | jq -R -s -c 'split("\n")[:-1]' > files_to_scan.json

      - name: Get All Files for Push
        if: github.event_name == 'push'
        shell: bash
        run: |
          echo "Analyzing all files in the repository..."
          all_files=$(git ls-files)
          echo "$all_files" | jq -R -s -c 'split("\n")[:-1]' > files_to_scan.json

      - name: Set files_path environment variable
        shell: bash
        run: |
          echo "files_path=$(pwd)/files_to_scan.json" >> $GITHUB_ENV

      - name: Run PLEXALYZER Analysis
        id: plexalyzer
        uses: plexicus/plexicus-action@<ACTION_BRANCH>
        with:
          default-owner: ${{ github.event.repository.owner.login }}
          repo-name: ${{ github.repository }}
          branch: ${{ github.event.pull_request.base.ref || github.ref_name }}
          url: ${{ github.event.repository.clone_url }}
          pr-id: ${{ github.event.pull_request.number }}
          files-path: ${{ env.files_path }}
          workspace-path: ${{ github.workspace }}

      - name: Save SARIF output to file
        if: steps.plexalyzer.outcome == 'success'
        shell: bash
        run: |
          printf '%s' "${{ steps.plexalyzer.outputs.sarif-output }}" > results.sarif

      - name: Upload SARIF results to GitHub
        if: steps.plexalyzer.outcome == 'success'
        uses: github/codeql-action/upload-sarif@v4
        with:
          sarif_file: results.sarif
          category: plexalyzer

```

## Setup Instructions

1. Generate your Plexalyzer token from the [COVULOR Connectors]
2. In your repository, go to Settings → Secrets and variables → Actions
3. Add a new repository secret called `PLEXALYZER_TOKEN` with your token
4. Get a repository ID by running the github action once and then accessing your COVULOR account in this URL: `https://app.plexicus.ai/repositories`
5. Add a new repository variable called `COVULOR_REPO_ID` with your repository ID

## Required Inputs

| Input | Description |
|-------|-------------|
| `plexalyzer-token` | Your Plexalyzer authentication token (should be kept secret) |
| `repo-id` | Your repository ID from COVULOR dashboard |

## Features

- Automatic PR analysis
- Direct integration with COVULOR cloud service
- Real-time results in your Pull Requests
- Secure token-based authentication
- SARIF output upload to GitHub Code Scanning

## Requirements

- Active COVULOR subscription
- Valid Plexalyzer authentication token
- Repository ID from COVULOR dashboard
- Repository with Pull Request events enabled

## Support

For support and questions:
- COVULOR Documentation: https://www.plexicus.ai
- Plexicus Support: engineering@plexicus.ai
- GitHub Issues: Create an issue in this repository

## About Plexicus

Plexicus provides enterprise-grade code analysis through the COVULOR cloud service. Learn more at [[Plexicus](https://www.plexicus.ai)].
