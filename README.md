Deploy with Docker Compose
==========================

[![Molecule Test](https://github.com/sandipb/ansible-role-deploy-with-docker-compose/actions/workflows/run_molecule.yml/badge.svg)](https://github.com/sandipb/ansible-role-deploy-with-docker-compose/actions/workflows/run_molecule.yml)

This Ansible role deploys one Docker Compose project into a service directory on the target host. It creates the service directory, renders `docker-compose.yml`, stages supporting files and templates, optionally pulls images, and runs Docker Compose.

Requirements
------------

- Docker Engine and Docker Compose must already be installed and configured on the managed host.
- The path referenced by `docker_services_root_dir` must already exist on the managed host.
- The role requires Ansible 12+ (see `pyproject.toml` for the tested toolchain).
- The control node must have the `community.docker` collection installed.
- Playbooks usually need `become: true` when deploying under system paths such as `/opt/docker`.

Role Variables
--------------

| Variable | Default | Required | Description |
| --- | --- | --- | --- |
| `docker_compose_template_path` | `null` | yes | Template used to render `docker-compose.yml`. Uses normal Ansible template lookup behavior. |
| `docker_services_root_dir` | `/opt/docker` | - | Existing absolute base directory under which service-specific directories are created. Cannot be `/`. |
| `docker_service_name` | `my-service` | - | Service directory name under `docker_services_root_dir`. Must match `^[A-Za-z0-9][A-Za-z0-9_.-]*$`. |
| `docker_service_user_name` | `null` | - | Optional target-system user name whose numeric ID is resolved at runtime; overrides `docker_service_user_id` when provided. |
| `docker_service_group_name` | `null` | - | Optional target-system group name whose numeric ID is resolved at runtime; overrides `docker_service_group_id` when provided. |
| `docker_service_user_id` | `1000` | - | Numeric user ID used when no user name is supplied. |
| `docker_service_group_id` | `1000` | - | Numeric group ID used when no group name is supplied. |
| `docker_service_dir_mode` | `'0770'` | - | Default mode applied to service directories. Must be a quoted octal-like string. |
| `docker_service_file_mode` | `'0660'` | - | Mode applied to files rendered or copied by the role. Must be a quoted octal-like string. |
| `docker_compose_template_vars` | `{}` | - | Mapping made available to the Docker Compose template. |
| `docker_service_subdirectories` | `[]` | - | Additional directories to create under the service path. Supports string entries and mapping entries with `path`, `owner`, `group`, and `mode`. |
| `docker_service_copy_files` | `{}` | - | Mapping of `src: dest` regular files to copy. Sources use normal Ansible copy lookup behavior; destinations are relative to the service directory. |
| `docker_service_templates` | `{}` | - | Mapping of `src: dest` templates to render. Sources use normal Ansible template lookup behavior; destinations are relative to the service directory. |
| `docker_service_templates_vars` | `{}` | - | Mapping made available to each template in `docker_service_templates`. |
| `docker_pull` | `false` | - | When true, pulls Docker images before Compose runs and recreates containers when pulls change images. |
| `docker_service_debug` | `false` | - | Print resolved target user and group IDs during role execution. |

Path Behavior
-------------

The role only writes below:

```text
{{ docker_services_root_dir }}/{{ docker_service_name }}/
```

Copy and template destinations may be nested, such as `config/nginx/default.conf`. The role automatically creates parent directories for copied and rendered files. If a directory's permissions matter to the running application, declare it explicitly in `docker_service_subdirectories`.

Examples:

```yaml
docker_service_subdirectories:
  - data
  - path: cache
    mode: "0755"

docker_service_templates:
  proxy.conf.j2: config/nginx/default.conf
```

In this example, `data` and `cache` are explicit directories. The role also creates `config` and `config/nginx` because they are parents of the rendered template.

Template Guidance
-----------------

Templates receive their inputs through dictionaries:

- `docker_compose_template_vars` for the Docker Compose template
- `docker_service_templates_vars` for supporting templates

For example:

```jinja2
{% set tpl_vars = docker_compose_template_vars %}

services:
  app:
    image: {{ tpl_vars.image }}
```

Supporting templates can use the shared service template map:

```jinja2
server_name {{ docker_service_templates_vars.nginx_server_name }};
```

Example Playbook
----------------

```yaml
- hosts: docker_hosts
  become: true
  roles:
    - role: sandipb.deploy_with_docker_compose
      vars:
        docker_service_name: app
        docker_compose_template_path: "{{ playbook_dir }}/templates/app/docker-compose.yml.j2"
        docker_compose_template_vars:
          image: myorg/app:latest
          replicas: 2
        docker_service_subdirectories:
          - path: data
            mode: "0755"
        docker_service_copy_files:
          "{{ playbook_dir }}/files/app.env": .env
        docker_service_templates:
          "{{ playbook_dir }}/templates/app/proxy.conf.j2": config/proxy.conf
        docker_service_templates_vars:
          proxy_backend_host: localhost
          proxy_backend_port: 8080
        docker_pull: "{{ docker_force_pull | default(false) }}"
```

To force image updates and container recreation, run the playbook with:

```bash
ansible-playbook playbook.yml -e docker_force_pull=true
```


Development
-----------

Common development commands are available through `Taskfile.yml`:

```bash
task lint
task molecule:test
task molecule:converge
task molecule:verify
```

Set `MOLECULE_SCENARIO` to target a non-default scenario. Before tagging a release, run `task release:check VERSION=<version>` to verify `CHANGELOG.md` and `pyproject.toml` are in sync.

Design Notes
------------

For implementation details and design constraints, see `docs/design.md`.

License
-------

Apache 2.0
