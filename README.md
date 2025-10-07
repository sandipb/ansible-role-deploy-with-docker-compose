Deploy with Docker Compose
==========================

[![Molecule Test](https://github.com/sandipb/ansible-role-deploy-with-docker-compose/actions/workflows/run_molecule.yml/badge.svg)](https://github.com/sandipb/ansible-role-deploy-with-docker-compose/actions/workflows/run_molecule.yml)

This Ansible role prepares a service directory on the target host, renders a docker-compose bundle, and stages any supporting files required to run the stack.

Requirements
------------
- Docker Engine and Docker Compose are expected to be installed and configured on the managed host.
- The role requires Ansible 12+ (see `pyproject.toml` for the tested toolchain).
- The control node must have the `community.docker` collection installed to provide the `docker_compose_v2` module used by this role.
- The path referenced by `docker_services_root_dir` must already exist on the managed host before the role runs.

Role Variables
--------------

| Variable | Default | Required | Description |
| --- | --- | --- | --- |
| `docker_compose_template_path` | `null` | yes | Absolute path (required) to the template used to produce the docker-compose definition. |
| `docker_services_root_dir` | `/opt/docker` | - | Base directory under which service-specific directories are created. |
| `docker_service_name` | `my-service` | - | Name of the service; used as the directory name under `docker_services_root_dir`. |
| `docker_service_user_name` | `null` | - | Optional system user whose numeric ID is derived at runtime; overrides `docker_service_user_id` when provided. |
| `docker_service_group_name` | `null` | - | Optional system group whose numeric ID is derived at runtime; overrides `docker_service_group_id` when provided. |
| `docker_service_user_id` | `1000` | - | Numeric user ID used when no user name is supplied. |
| `docker_service_group_id` | `1000` | - | Numeric group ID used when no group name is supplied. |
| `docker_service_dir_mode` | `'0770'` | - | Mode applied to created directories inside the service path. |
| `docker_service_file_mode` | `'0660'` | - | Mode applied to files rendered or copied into the service path. |
| `docker_compose_template_vars` | `{}` | - | Variables passed to the docker-compose template render. |
| `docker_service_subdirectories` | `[]` | - | List of additional directories to create under the service path. |
| `docker_service_copy_files` | `{}` | - | Mapping of `src: dest` regular files to copy to the service directory. `src` must be an absolute path; `dest` is relative to the service directory. |
| `docker_service_templates` | `{}` | - | Mapping of `src: dest` templates to render into the service directory. `src` must be an absolute path; `dest` is relative to the service directory. |
| `docker_service_templates_vars` | `{}` | - | Variable map made available to each entry under `docker_service_templates` when rendering templates. |
| `docker_pull` | `false` | - | When set to `true`, forces Docker images to be pulled from the registry and recreates containers to use the updated images. Useful for refreshing images with mutable tags like `:latest`. |

Template Guidance
-----------------

### Paths and Assets
This role ships without bundled compose, file, or template assets. Set `docker_compose_template_path` (required) to an absolute path—for example `{{ playbook_dir }}/templates/docker-compose.yml.j2`—and always provide absolute paths for every `docker_service_copy_files` source and each `docker_service_templates` entry.

### Referencing Template Variables
Templates receive their inputs via the dictionaries defined in `docker_compose_template_vars` and `docker_service_templates_vars`. Reference values through those maps (for example `docker_compose_template_vars.image`). To keep templates terse, you can set a local variable at the top—see `molecule/default/templates/docker-compose.yml.j2` for an example using `{% set tpl_vars = docker_compose_template_vars %}`.

Example Playbook
----------------

```yaml
- hosts: docker_hosts
  roles:
    - role: ansible-role-deploy-with-docker-compose
      vars:
        docker_service_name: app
        docker_compose_template_path: "{{ playbook_dir }}/templates/app/docker-compose.yml.j2"
        docker_compose_template_vars:
          image: myorg/app:latest
          replicas: 2
        docker_service_copy_files:
          "{{ playbook_dir }}/files/app.env": config/app.env
        docker_service_templates:
          "{{ playbook_dir }}/templates/app/proxy.conf.j2": config/proxy.conf
        docker_service_templates_vars:
          backend_host: localhost
          backend_port: 8080
        docker_pull: "{{ docker_force_pull | default(false) }}"
```

To force image updates and container recreation, run the playbook with:
```bash
ansible-playbook playbook.yml -e docker_force_pull=true
```

License
-------

Apache 2.0
