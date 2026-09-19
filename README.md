# Ansible Role: user

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-user)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-user)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-user)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-user/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-user/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-user/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-user/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for managing local accounts, passwords, and authorized SSH keys.

## Purpose

Manage local Linux accounts and password hashes with ansible.builtin.user,
then apply authorized SSH keys with ansible.posix.authorized_key.
Account and public key lists are independent, allowing key management for
existing accounts.

## Scope

### Managed

- Platform-specific account management utilities.
- Local account attributes, password hashes, password locks, and account
  removal.
- Public keys in each selected account's home directory under
  .ssh/authorized_keys.

### Not Managed

- SSH key pair generation, private key distribution, and SSH server
  configuration.
- Group creation, sudo permissions, and directory service accounts.

## Requirements

- Gather Ansible facts so the role can select account management packages for
  the operating system family.
- Primary and supplementary groups named in user_accounts must already exist.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `user_accounts`

Type: `list`. Required: `false`.

Local accounts managed with ansible.builtin.user; each entry requires name.
Optional account properties are state, uid, comment, group, groups, append,
home, create_home, shell, system, password, password_lock, update_password, and
remove.
State defaults to present. Password accepts an existing Linux password hash;
omitting it leaves an existing password unchanged. Groups must already exist.
Append, create_home, update_password, and remove inherit the corresponding
role-wide policy unless explicitly overridden in the account entry.

Default:

```yaml
user_accounts: []
```

### `user_authorized_keys`

Type: `list`. Required: `false`.

One key batch per existing or newly managed account, requiring name and keys.
Keys is a list of public key lines, optionally including authorized_keys
options. State defaults to present; absent removes the listed keys.
Exclusive inherits user_authorized_keys_exclusive. With exclusive true and state
present, the batch replaces all keys; an empty keys list clears authorized_keys.
Do not include accounts removed by user_accounts. Omitted accounts retain their
authorized keys.

Default:

```yaml
user_authorized_keys: []
```

### `user_create_home`

Type: `bool`. Required: `false`.

Create home directories unless overridden per account.

Default:

```yaml
user_create_home: true
```

### `user_append`

Type: `bool`. Required: `false`.

Preserve unlisted supplementary group memberships unless overridden per account.

Default:

```yaml
user_append: true
```

### `user_remove`

Type: `bool`. Required: `false`.

Remove home directories and mail spools when deleting accounts unless overridden
per account.

Default:

```yaml
user_remove: false
```

### `user_update_password`

Type: `str`. Required: `false`.

Apply changed password hashes always or only when creating accounts, unless
overridden per account.

Default:

```yaml
user_update_password: always
```

### `user_authorized_keys_exclusive`

Type: `bool`. Required: `false`.

Remove authorized keys outside each present batch unless overridden per account.

Default:

```yaml
user_authorized_keys_exclusive: false
```

## Check Mode

Account and key modules support check mode for existing accounts.

- On a fresh host, key management for accounts only predicted by check mode can
  fail because the accounts do not yet exist. Run convergence before checking
  their authorized keys.

## Service Behavior

Changes take effect without restarting the SSH service.

## Security Notes

- Password values must be Linux password hashes, for example SHA-512 crypt or
  yescrypt hashes supported by the target. Store hashes in Ansible Vault. The
  role does not hash cleartext passwords.
- Account tasks process password hashes with no_log enabled. Public key tasks
  use a separate list containing no password values and remain visible in
  Ansible output.
- Password locks affect password authentication; remove authorized keys
  separately to revoke public key access.

## Operational Notes

- By default both input lists are empty. Omitted accounts and their passwords
  and keys are left unchanged. A supplied password is updated on existing
  accounts unless update_password is on_create.
- Use one user_authorized_keys entry per account. All keys in an entry are
  passed together, so exclusive mode retains every specified key. Public key
  lines can include per-key restrictions.
- With state present and exclusive true, only the supplied keys remain. An empty
  keys list removes all authorized keys. With state absent, only the listed keys
  are removed; exclusive is not used.
- Accounts referenced by user_authorized_keys must exist after account
  management. Remove key entries when deleting the corresponding account.
  Account deletion retains home directories unless remove is true.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Manage a password and authorized keys

Apply a Vault-provided password hash and replace Alice's authorized keys
with the supplied public keys.

```yaml
---
- name: Manage local users
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.user
      user_accounts:
        - name: alice
          comment: Alice Example
          shell: /bin/bash
          password: "{{ vault_alice_password_hash }}"
      user_authorized_keys:
        - name: alice
          exclusive: true
          keys:
            - "{{ lookup('ansible.builtin.file', 'public_keys/alice.pub') }}"
            - "{{ lookup('ansible.builtin.file', 'public_keys/spare.pub') }}"

```

### Revoke public key access or remove an account

Clear authorized keys for an existing account and delete a different
account together with its home.

```yaml
---
- name: Revoke local access
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.user
      user_accounts:
        - name: retired
          state: absent
          remove: true
      user_authorized_keys:
        - name: alice
          keys: []
          exclusive: true
```

## References

- [Ansible user module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/user_module.html)
- [Ansible authorized_key module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/authorized_key_module.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024 Jonas Mauer.
