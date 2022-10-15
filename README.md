# first_contact

An [Ansible](https://www.ansible.com) role to configure the ansible user on first contact with a host.

> the ansible user is the user that is used to run ansible commands on the host.

- check if the ansible user exists
- if not connect to host as root.
- create the ansible user.
  - add the ansible user to the sudoers group.
  - add the control nodes public key to the host ansible user.
  - set the ansible user's shell

## Requirements

None.

## Role Variables

```yaml
---
ssh_key_location: "~/.ssh/id_ed25519.pub"
connection_timeout: 3
deploy_user: "deploy"
```

## Dependencies

robertdebock.bootstrap

## Example Playbook

```yaml
---
- name: First Contact
  hosts: all
  roles:
    - dgibbs64.first_contact
```

## License

MIT

## Author Information

- [Daniel Gibbs](https://danielgibbs.co.uk)
