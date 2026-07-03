# Agents Guide

## Purpose
This document describes how AI coding agents should work in this Ansible role repository.

## Expectations
- Keep changes focused on the requested task and follow the existing layout in `defaults/`, `tasks/`, `docs/`, and `molecule/`.
- Preserve the role's write boundary: managed-host writes must stay under `docker_services_root_dir/docker_service_name`.
- Do not add host-level setup to this role. Docker installation, user/group creation, secrets, DNS, firewall rules, and creation of `docker_services_root_dir` belong outside the role.
- Use conventional commit messages when committing, such as `feat: add directory overrides` or `fix: validate service paths`.
- Run relevant Molecule scenarios or linting when practical, and report anything that could not be executed.

## Workflow
1. Review current context before modifying code: `README.md`, `docs/design.md`, `defaults/main.yml`, `tasks/`, and the Molecule scenario.
2. Discuss plans for multi-step work when the scope extends beyond trivial edits.
3. Keep user-facing behavior in `README.md` and maintainer rationale in `docs/design.md`.
4. Validate templates in `molecule/default/templates/` against supplied vars before running Molecule.
5. Summarize modifications, validation results, and recommended follow-up actions at the end of each session.

## Testing Guidance
- Use `uv run molecule test` for full verification after significant role behavior changes.
- For faster iteration, use targeted `uv run molecule converge` and `uv run molecule verify` steps.
- Execute `uv run ansible-lint` after significant task, handler, or Molecule updates.
- Capture command outputs succinctly and highlight failures with next-step suggestions.

## Code Style
- Follow YAML best practices: two-space indentation, lower-case booleans, and descriptive variable names.
- Keep validation before filesystem writes.
- Prefer explicit, readable Ansible tasks over dense templating when behavior is safety-critical.
- Keep Docker Compose and supporting templates explicit; document assumptions in `README.md` when they affect users.
- Add comments only where they clarify non-obvious logic.

## Documentation
- Keep `README.md` user-focused: requirements, variables, examples, and operational behavior.
- Keep `docs/design.md` aligned with the implemented design: write boundary, validation rules, directory normalization, ownership/mode behavior, and non-goals.
- Update Molecule verification when changing role behavior.
- Keep `pyproject.toml` aligned with the tested toolchain and release version.
- Keep the Git tag, `pyproject.toml` version, and top `CHANGELOG.md` entry in sync for releases.

## Safety & Secrets
- Never commit secrets or host-specific data; rely on inventory variables or vault integration.
- Respect `.gitignore` and Molecule ignore rules such as `molecule/default/files/ignore.conf`.
- Flag unexpected configuration drift or dirty working tree state that you did not introduce.

## Communication
- Confirm assumptions early, especially around deployment targets, permissions, and secrets.
- Note operations requiring elevated permissions or network access before running them.
- Provide concise summaries with actionable next steps for human reviewers.
