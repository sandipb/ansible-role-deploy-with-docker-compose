# Role Design

This role deploys one Docker Compose project into a service-specific directory on a managed host. It prepares the project directory, renders `docker-compose.yml`, stages supporting files and templates, optionally pulls images, and runs Docker Compose.

## Write Boundary

The role only writes below:

```text
{{ docker_services_root_dir }}/{{ docker_service_name }}/
```

`docker_services_root_dir` is a host-level prerequisite. It must already exist, must be an absolute path, and must not be `/`. Creating or managing the root directory is outside this role.

`docker_service_name` is a single safe directory name matching:

```text
^[A-Za-z0-9][A-Za-z0-9_.-]*$
```

This prevents the role from writing directly under `docker_services_root_dir`, writing into sibling service directories, or using path traversal through the service name.

## Source Paths and Destination Paths

Source paths for `docker_compose_template_path`, `docker_service_copy_files` keys, and `docker_service_templates` keys use normal Ansible `template` and `copy` lookup behavior. They may be absolute or lookup-relative. The role does not pre-validate source existence; Ansible modules report lookup failures.

Destination paths for `docker_service_copy_files` and `docker_service_templates` are managed-host paths relative to the service directory. They must be safe relative file paths:

- not empty
- not absolute
- not `.`
- no empty path segments
- no `.` or `..` path segments

Dotfiles such as `.env` are allowed because `.env` is a normal file name segment, not the `.` path segment.

## Directory Management

The service root is always created with:

- `docker_service_dir_mode`
- the resolved service owner
- the resolved service group

The role creates one normalized list of subdirectories before copying or rendering files. That list combines:

- directories explicitly declared in `docker_service_subdirectories`
- parent directories inferred from `docker_service_copy_files` destinations
- parent directories inferred from `docker_service_templates` destinations

For a destination such as:

```yaml
docker_service_templates:
  default.conf.j2: config/nginx/conf.d/default.conf
```

The inferred parent directories are:

```text
config
config/nginx
config/nginx/conf.d
```

Explicit entries override inferred entries for the same path. Duplicate explicit `docker_service_subdirectories` paths are rejected.

## Subdirectory API

`docker_service_subdirectories` supports the original string form:

```yaml
docker_service_subdirectories:
  - data
```

It also supports mapping entries with per-directory metadata overrides:

```yaml
docker_service_subdirectories:
  - path: data
    owner: appuser
    group: appgroup
    mode: "0755"
```

Missing `owner`, `group`, or `mode` values fall back to the role defaults after user/group name resolution. Per-directory `owner` and `group` values are passed directly to Ansible's `file` module, so they may be names or numeric IDs. Per-directory `mode` values must be quoted octal-like strings matching `^[0-7]{3,4}$`.

Auto-created parent directories use default metadata unless explicitly listed in `docker_service_subdirectories`. If directory permissions matter for application runtime behavior, declare the directory explicitly.

## User and Group Resolution

The public inputs are:

- `docker_service_user_name`
- `docker_service_group_name`
- `docker_service_user_id`
- `docker_service_group_id`

If a user or group name is provided, the role resolves the numeric ID on the managed host with `getent`. Names override the fallback numeric IDs. The resolved values are stored in internal facts and used for default ownership.

If no name is provided, the corresponding numeric ID must be numeric-looking. Integers and numeric strings are accepted.

## File Ownership and Modes

Rendered and copied files are role-owned deployment artifacts. They always use:

- `docker_service_file_mode`
- the resolved service owner
- the resolved service group

The role does not support per-file ownership or mode overrides. Add that only if a concrete use case requires it.

## Compose Lifecycle

The role renders and stages files before running Compose. The order is:

1. validate inputs and prerequisites
2. resolve effective owner and group
3. create the service root and normalized directories
4. render `docker-compose.yml`
5. copy support files
6. render support templates
7. pull images when `docker_pull` is true
8. run `community.docker.docker_compose_v2`

Compose uses `recreate: always` when any managed compose file, copied file, rendered template, or image pull changed. Otherwise it uses `recreate: auto`.

The role intentionally does not expose arbitrary `docker_compose_v2` pass-through options in this design. Compose lifecycle options can be added later with their own documentation and tests.

## Non-Goals

The role does not:

- install Docker or Docker Compose
- create `docker_services_root_dir`
- manage files outside the service directory
- manage host-level firewall, proxy, DNS, users, groups, or secrets
- support render-only mode
- manage per-file owner/mode overrides
