# Ansible role user

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-user) ![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-user) ![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-user)

**Ansible role for managing local user accounts.**

## Description

This Ansible role installs and configures user on supported platforms.

## Prerequisites

This role has no special prerequisites.

### System packages (Fedora)

- `python3` (>= 3.9)

### Python (requirements.txt)

- ansible >= 2.17

## Dependencies (requirements.yml)

This role has no dependencies.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
|-----------|--------------|---------|-----------------|
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest]( https://hub.docker.com/r/jomrr/molecule-almalinux ) |
| Alpine | Alpine | latest | [jomrr/molecule-alpine:latest]( https://hub.docker.com/r/jomrr/molecule-alpine ) |
| Archlinux | Archlinux | latest | [jomrr/molecule-archlinux:latest]( https://hub.docker.com/r/jomrr/molecule-archlinux ) |
| Debian | Debian | oldstable | [jomrr/molecule-debian:oldstable]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| | | stable | [jomrr/molecule-debian:stable]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| | | testing | [jomrr/molecule-debian:testing]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest]( https://hub.docker.com/r/jomrr/molecule-fedora ) |
| | | rawhide | [jomrr/molecule-fedora:rawhide]( https://hub.docker.com/r/jomrr/molecule-fedora ) |
| Suse | OpenSuse Leap | 15 | [jomrr/molecule-opensuse-leap:15]( https://hub.docker.com/r/jomrr/molecule-opensuse-leap ) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest]( https://hub.docker.com/r/jomrr/molecule-ubuntu ) |

## Role Variables

No role default variables specified, see [defaults/main.yml](defaults/main.yml).

## Example Playbook

Example playbooks(s) that show how to use this role.

## Simple example playbook

A simple default example playbook for using jomrr.user.
```yaml
---
# name: "jomrr.user"
# file: "playbook_user.yml"

- name: "PLAYBOOK | user"
  hosts: "user_hosts"
  gather_facts: true
  roles:
    - role: "jomrr.user"
```

## Author(s) and License

- :octocat:                 Author::    [jomrr](https://github.com/jomrr)
- :triangular_flag_on_post: Copyright:: 2024, Jonas Mauer
- :page_with_curl:          License::   [MIT](LICENSE)


---
