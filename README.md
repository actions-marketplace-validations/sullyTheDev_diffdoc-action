# DiffDoc Delta Summarizer Action

A composite GitHub Action that automates token-efficient code summarization using [DiffDoc](https://github.com/sullyTheDev/diffdoc). Runs a full pipeline: schema validation, delta summarization, manifest pruning, and auto-commit of results.

## Pipeline Steps

1. **Verify Git** — Ensures the repository is checked out with sufficient history (`fetch-depth: 0`). Hard errors on missing `.git` or shallow clones.
2. **Validate** — Checks existing manifest and summary assets against the v2 JSON schema contracts. Warns on failure (allows greenfield repos with no manifest yet).
3. **Summarize** — Runs `diffdoc summarize --mode delta` to process only changed files, minimizing LLM token consumption.
4. **Prune** — Removes orphaned manifest entries for deleted or newly-excluded files.
5. **Commit** — Pushes updated `.diffdoc/` artifacts back to the branch.

## Quick Start

```yaml
name: DiffDoc Summarize
on:
  push:
    branches: [main]

jobs:
  summarize:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: sullyTheDev/diffdoc-action@v2
        with:
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `openai-api-key` | No | — | OpenAI-compatible API key. Optional if using an unauthenticated local LLM. |
| `config-path` | No | — | Path to `.diffdocrc` config file. |
| `base-dir` | No | — | DiffDoc artifact directory path override. |
| `ai-provider` | No | — | AI provider override: `local` or `cloud`. |
| `local-llm-endpoint` | No | — | Local OpenAI-compatible chat endpoint URL. |
| `local-chat-model` | No | — | Local chat model name. |
| `cloud-llm-endpoint` | No | — | Cloud OpenAI-compatible chat endpoint URL. |
| `cloud-chat-model` | No | — | Cloud chat model name. |
| `scan-path` | No | `.` | Repository or code path to scan. |
| `manifest-out` | No | `manifest.json` | Manifest output filename/path under `--base-dir`. |
| `summarize-concurrency` | No | — | Number of files to summarize concurrently. |
| `commit-message` | No | `chore(diffdoc): automated delta summary update [skip ci]` | Commit message for pushed summary updates. |

## Outputs

| Output | Description |
|--------|-------------|
| `files-updated` | Number of files summarized/updated in this run. |
| `files-failed` | Number of files that failed summarization. |
| `files-pruned` | Number of manifest entries pruned. |
| `commit-sha` | SHA of the auto-commit (empty if no changes were made). |

### Using Outputs

```yaml
- uses: sullyTheDev/diffdoc-action@v2
  id: diffdoc
  with:
    openai-api-key: ${{ secrets.OPENAI_API_KEY }}

- if: steps.diffdoc.outputs.files-failed != '0'
  run: echo "Some files failed summarization"
```

## Configuration Precedence

Values resolve in strict priority order:

1. Explicit action inputs (passed as CLI flags)
2. Environment variables (e.g., `OPENAI_API_KEY`)
3. `.diffdocrc` file settings
4. Engine defaults

## Examples

### Cloud Provider (OpenAI)

```yaml
- uses: sullyTheDev/diffdoc-action@v2
  with:
    openai-api-key: ${{ secrets.OPENAI_API_KEY }}
    cloud-chat-model: 'gpt-4o'
```

### Local LLM (Ollama on Self-Hosted Runner)

```yaml
- uses: sullyTheDev/diffdoc-action@v2
  with:
    ai-provider: 'local'
    local-llm-endpoint: 'http://localhost:11434/v1'
    local-chat-model: 'llama3'
```

### Custom Artifact Directory

```yaml
- uses: sullyTheDev/diffdoc-action@v2
  with:
    openai-api-key: ${{ secrets.OPENAI_API_KEY }}
    base-dir: 'docs/summaries'
```

## Schema Contracts

This action validates against the published v2 schemas:

- [Config Schema (`.diffdocrc`)](https://raw.githubusercontent.com/sullyTheDev/diffdoc/main/schemas/v2/diffdocrc.schema.json)
- [Summary Asset Schema](https://raw.githubusercontent.com/sullyTheDev/diffdoc/main/schemas/v2/summary-asset.schema.json)

## Requirements

- Node.js 22 (set up automatically by the action)
- `fetch-depth: 0` on checkout (required for delta mode to detect changes)

## License

See [LICENSE](./LICENSE).
