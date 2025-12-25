# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a CircleCI Orb that validates commit messages using AI-powered analysis from the [omni-dev](https://github.com/rust-works/omni-dev) CLI tool. It supports multiple AI providers (Anthropic Claude, OpenAI, Ollama, AWS Bedrock).

## Repository Structure

- `src/orb.yml` - The main orb definition with commands, jobs, executors, and examples
- `.circleci/config.yml` - CI configuration for linting, testing, and publishing the orb
- `README.md` - User documentation with examples and parameter reference

## How It Works

The orb provides:
1. An `install` command that installs `omni-dev` via `cargo install` with caching
2. A `check` command that runs `omni-dev git commit message check` with configurable parameters
3. A `commit-check` job that combines checkout, Rust setup, install, and check

## Key Technical Details

- **Exit codes**: 0 = success, 1 = errors, 2 = warnings (with strict mode), 3 = no commits found
- **Environment variables**: API keys passed via CircleCI contexts (e.g., `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`)
- **Commit range detection**: Auto-detected from branch comparison if not explicitly specified
- **Guidelines discovery**: Defaults to `.omni-dev/commit-guidelines.md` in the target repository

## Validating the Orb

```bash
circleci orb validate src/orb.yml
```

## Commit and PR Guidelines

This project uses conventional commits with required scopes.

**Scopes**: `orb`, `docs`, `ci`

**Example commits**:
```
feat(orb): add verbose output flag
fix(orb): handle missing API key gracefully
docs(docs): add Bedrock usage example
```
