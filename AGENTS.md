# Repository Guidelines

## Project Structure & Module Organization

This repository contains independent container deployments under `docker/<project>/` and `podman/<project>/`, and host utilities under `scripts/<utility>/`. Deployment directories hold Compose configuration, environment examples, and optional Dockerfiles, scripts, configuration templates, documentation, and static assets. Some projects include platform-specific integrations; check the affected directory and its README for requirements. There is no central application package or test directory.

## Build, Test, and Development Commands

Run Compose commands from the relevant project directory:

- `cp .env.example .env`: initialize local configuration; edit paths and required variables before use.
- `docker compose config --quiet`: validate the resolved Compose configuration.
- `docker compose build`: build services with a local Dockerfile.
- `docker compose up -d`: create or update the configured services.
- `docker compose logs --tail=100`: inspect service output after a change.

For project-specific commands and host utilities, consult the relevant README. Use available dry-run modes before applying host configuration changes.

## Coding Style & Naming Conventions

Use two-space indentation in YAML and four spaces in Python. Follow nearby shell formatting, preserve the declared interpreter, and keep `/bin/sh` scripts POSIX-compatible. Quote shell variable expansions and use descriptive functions. Environment settings use `UPPER_SNAKE_CASE`; script filenames generally use lowercase words with hyphens. Preserve journald tags in the `<project>-<service>` format. No repository-wide formatter or linter is configured.

## Testing Guidelines

There is no automated test framework or coverage threshold. Validate changed Compose files with configured environment values, check shell syntax with `sh -n path/to/script.sh` (or `bash -n` for Bash scripts), and use available dry-run modes. Smoke-test affected services on a suitable host and inspect logs. Document commands and observed results in the pull request.

## Commit & Pull Request Guidelines

History uses prefixes such as `feat:`, `fix:`, and `build:`. Follow that convention with a specific, concise description. Keep changes scoped to the affected deployment or utility. Pull requests should explain the operational change, configuration or migration steps, and validation results; link relevant issues when available.

## Security & Configuration

Keep credentials, real `.env` files, and generated host configuration out of commits. Use reserved example domains (`example.com`) and documentation IP ranges for examples instead of personal domains or deployment addresses. Preserve functional upstream addresses and package identifiers. Update `.env.example` and the service README when settings change. Review volume permissions, exposed ports, and privileged host operations before deployment.
