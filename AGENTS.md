# Agents Guide

## Purpose
This document describes how AI coding agents (like Codex) should interact with this Ansible role repository.

## Expectations
- Keep changes focused on the requested task and respect existing conventions in `defaults/`, `tasks/`, and Molecule scenario files.
- Prefer incremental updates with clear commit messages if committing outside CI jobs.
- **Always use conventional commits format for commit messages** (e.g., `feat: add new feature`, `fix: resolve bug`, `docs: update documentation`, `test: add test cases`, `refactor: improve code structure`).
- Run relevant Molecule scenarios or linting when practical and report anything that could not be executed.

## Workflow
1. Review the current context (open files, Molecule scenario, defaults) before modifying code.
2. Discuss plans for multi-step work when the scope extends beyond trivial edits.
3. Validate templates (`molecule/default/templates/`) render correctly against supplied vars before running Molecule.
4. Summarize modifications and recommended follow-up actions at the end of each session.

## Testing Guidance
- Use `uv run molecule test` for full verification when time permits; otherwise run targeted `uv run molecule converge` and `uv run molecule verify` steps.
- Execute `uv run ansible-lint` after significant task or handler updates.
- Capture command outputs succinctly; highlight failures with next-step suggestions.

## Code Style
- Follow YAML best practices: two-space indentation, lower-case booleans, descriptive variable names.
- When templating docker-compose files, keep environment variables explicit and document assumptions in `README.md`.
- Introduce comments only where necessary to clarify complex logic.

## Communication
- Confirm assumptions early, especially around deployment targets and secrets.
- Note any operations requiring elevated permissions or network access before running them.
- Provide concise summaries with actionable next steps for human reviewers.

## Safety & Secrets
- Never commit secrets or host-specific data; rely on inventory variables or vault integration.
- Respect `.gitignore` and Molecule ignore rules (`molecule/default/files/ignore.conf`).
- Flag any unexpected configuration drift or dirty working tree state you did not introduce.

## Documentation
- Keep `README.md` and `pyproject.toml` aligned with implemented features.
- Update Molecule scenario documentation when adding new verification steps or dependencies.
