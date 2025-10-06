Deploy with Docker Compose
==========================

This Ansible role prepares a service directory on the target host, renders a docker-compose bundle, and stages any supporting files required to run the stack.

Requirements
------------
- Docker Engine and Docker Compose are expected to be installed and configured on the managed host.
- The role requires Ansible 12+ (see `pyproject.toml` for the tested toolchain).

Role Variables
--------------

| Variable | Default | Description |
| --- | --- | --- |
| `docker_services_root_dir` | `/opt/docker` | Base directory under which service-specific directories are created. |
| `docker_service_name` | `my-service` | Name of the service; used as the directory name under `docker_services_root_dir`. |
| `docker_service_user_name` | `None` | Optional system user whose numeric ID is derived at runtime; overrides `docker_service_user_id` when provided. |
| `docker_service_group_name` | `None` | Optional system group whose numeric ID is derived at runtime; overrides `docker_service_group_id` when provided. |
| `docker_service_user_id` | `1000` | Numeric user ID used when no user name is supplied. |
| `docker_service_group_id` | `1000` | Numeric group ID used when no group name is supplied. |
| `docker_service_dir_mode` | `'0770'` | Mode applied to created directories inside the service path. |
| `docker_service_file_mode` | `'0660'` | Mode applied to files rendered or copied into the service path. |
| `docker_compose_template_vars` | `{}` | Variables passed to the docker-compose template render. |
| `docker_compose_template_path` | `docker-compose.yml.j2` | Absolute path to the template used to produce the docker-compose definition. |
| `docker_service_subdirectories` | `[]` | List of additional directories to create under the service path. |
| `docker_service_copy_files` | `{}` | Mapping of `src: dest` regular files to copy to the service directory. `src` must be an absolute path; `dest` is relative to the service directory. |
| `docker_service_templates` | `{}` | Mapping of `src: dest` templates to render into the service directory. `src` must be an absolute path; `dest` is relative to the service directory. |
| `docker_service_templates_vars` | `{}` | Variable map made available to each entry under `docker_service_templates` when rendering templates. |

> **Note:** This role ships without bundled compose, file, or template assets. Always provide absolute paths (for example `{{ playbook_dir }}/templates/docker-compose.yml.j2`) for `docker_compose_template_path`, every `docker_service_copy_files` source, and each `docker_service_templates` entry.

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
```

License
-------

Apache 2.0
