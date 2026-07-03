# Changelog

All notable changes to this role are documented in this file.

## 0.2.0 - 2026-07-03

### Added

- Added strict validation for the service root, service name, destination paths, variable types, file modes, fallback IDs, and boolean-like pull values.
- Added automatic creation of parent directories inferred from copied file and rendered template destinations.
- Added mapping support for `docker_service_subdirectories` entries with per-directory `path`, `owner`, `group`, and `mode` overrides.
- Added `docker_service_debug` to gate resolved user/group ID debug output.
- Added `docs/design.md` with role design constraints, write-boundary rules, directory normalization behavior, and non-goals.
- Added `Taskfile.yml` for common development commands.

### Changed

- Resolved service owner and group IDs are now stored in internal facts instead of overwriting public input variables.
- Explicit subdirectory declarations now override inferred parent directory defaults.
- Molecule coverage now verifies string subdirectories, mapping subdirectories, inferred parent directories, nested template destinations, and HTTP readiness retries.
- README now focuses on user-facing setup, variables, examples, path behavior, and development shortcuts.
- Molecule's Docker daemon uses the `vfs` storage driver to make Docker-in-Docker tests reliable in nested container environments.
- Cleaned up `AGENTS.md` to reflect the role's design boundaries and documentation workflow.

### Fixed

- Fixed README examples that implied nested destinations would work without parent directory handling.
- Fixed Molecule HTTP retry behavior by adding an `until` condition.
- Fixed `.pre-commit-config.yaml` formatting so `ansible-lint` can parse it successfully.
