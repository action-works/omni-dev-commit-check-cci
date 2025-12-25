# Omni-Dev Commit Check CircleCI Orb

A CircleCI Orb that validates commit messages against project guidelines using AI-powered analysis from [omni-dev](https://github.com/rust-works/omni-dev).

## Features

- AI-powered commit message validation
- Support for multiple AI providers (Anthropic Claude, OpenAI, Ollama, AWS Bedrock)
- Customizable commit guidelines
- Multiple output formats (text, JSON, YAML)
- Configurable severity levels and exit codes
- Batch processing for large commit ranges

## Quick Start

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  commit-validation:
    jobs:
      - omni-dev/commit-check:
          context: anthropic-credentials  # Context with ANTHROPIC_API_KEY
```

## Prerequisites

Set up your API keys as environment variables in a CircleCI context or project settings:

- `ANTHROPIC_API_KEY` - For Anthropic Claude (default provider)
- `OPENAI_API_KEY` - For OpenAI (when `use-openai: true`)
- `ANTHROPIC_AUTH_TOKEN` - For AWS Bedrock (when `use-bedrock: true`)

## Commands

### install

Installs the omni-dev CLI tool.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `version` | string | `latest` | omni-dev version to install |
| `cache` | boolean | `true` | Whether to cache the binary |

### check

Runs commit message validation.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `use-openai` | boolean | `false` | Use OpenAI instead of Anthropic |
| `use-ollama` | boolean | `false` | Use Ollama for local inference |
| `use-bedrock` | boolean | `false` | Use AWS Bedrock for Claude |
| `model` | string | `""` | AI model to use |
| `ollama-base-url` | string | `""` | Ollama server URL |
| `ollama-model` | string | `""` | Ollama model name |
| `bedrock-base-url` | string | `""` | AWS Bedrock endpoint URL |
| `commit-range` | string | `""` | Commit range to check (e.g., `HEAD~5..HEAD`) |
| `base-branch` | string | `""` | Base branch to compare against |
| `guidelines` | string | `""` | Path to custom commit guidelines file |
| `context-dir` | string | `.omni-dev/` | Context directory for loading guidelines |
| `format` | enum | `text` | Output format: `text`, `json`, or `yaml` |
| `strict` | boolean | `false` | Exit with error if warnings found |
| `quiet` | boolean | `false` | Suppress info-level output |
| `verbose` | boolean | `false` | Show detailed analysis |
| `show-passing` | boolean | `false` | Include passing commits in output |
| `no-suggestions` | boolean | `false` | Skip generating suggestions |
| `batch-size` | integer | `4` | Commits per AI request |

## Jobs

### commit-check

A complete job that checks out code, installs dependencies, and runs the commit check.

Accepts all parameters from the `check` command plus:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `executor` | executor | `default` | Executor to use |
| `version` | string | `latest` | omni-dev version to install |

## Examples

### Basic Usage with Claude

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  validate:
    jobs:
      - omni-dev/commit-check:
          context: anthropic-credentials
```

### Using OpenAI

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  validate:
    jobs:
      - omni-dev/commit-check:
          use-openai: true
          model: gpt-4.1
          context: openai-credentials
```

### Strict Mode with Custom Guidelines

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  validate:
    jobs:
      - omni-dev/commit-check:
          strict: true
          guidelines: .github/commit-guidelines.md
          context: anthropic-credentials
```

### Check Specific Commit Range

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  validate:
    jobs:
      - omni-dev/commit-check:
          commit-range: "origin/main..HEAD"
          context: anthropic-credentials
```

### Using Commands for Custom Workflows

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0
  rust: circleci/rust@1.6.1

jobs:
  custom-validation:
    docker:
      - image: cimg/rust:1.83
    steps:
      - checkout
      - rust/install
      - omni-dev/install:
          version: "0.12.0"
      - omni-dev/check:
          format: json
          verbose: true
      - run:
          name: Custom post-processing
          command: |
            echo "Add your custom logic here"

workflows:
  validate:
    jobs:
      - custom-validation:
          context: anthropic-credentials
```

### AWS Bedrock

```yaml
version: 2.1

orbs:
  omni-dev: action-works/omni-dev-commit-check@1.0.0

workflows:
  validate:
    jobs:
      - omni-dev/commit-check:
          use-bedrock: true
          model: anthropic.claude-3-opus-20240229-v1:0
          context: aws-bedrock-credentials
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success: All commits pass |
| 1 | One or more commits have errors |
| 2 | Warnings found (only with `strict: true`) |
| 3 | No commits found in range |

## Custom Guidelines

Create a `.omni-dev/commit-guidelines.md` file in your repository:

```markdown
# Project Commit Guidelines

## Severity Levels

| Severity | Sections |
|----------|----------|
| error    | Format, Subject Line |
| warning  | Content, Body |
| info     | Style |

## Format
- Use conventional commit format: `type(scope): description`
- Valid types: feat, fix, docs, style, refactor, test, chore, ci

## Subject Line
- Keep under 72 characters
- Use imperative mood ("add" not "added")
- Do not end with a period

## Content
- Reference issue numbers where applicable
- Explain the "why" not just the "what"
```

## Development

### Validating the Orb

```bash
circleci orb validate src/orb.yml
```

### Publishing (for maintainers)

```bash
# Development version
circleci orb publish src/orb.yml action-works/omni-dev-commit-check@dev:alpha

# Production version
circleci orb publish promote action-works/omni-dev-commit-check@dev:alpha patch
```

## License

MIT
