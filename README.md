# first_contact

An [Ansible](https://www.ansible.com) role that on first contact with a host will prepare it to be managed by Ansible.

<p align="center">
  <img src="https://user-images.githubusercontent.com/4478206/202930662-130cf21e-8d0c-4b4d-a9af-682c61b0b62d.png" alt="Ansible First Contact"></a>

<br>
</p>
<p align="center">
<a href="https://app.codacy.com/gh/dgibbs64/ansible-role-first_contact"><img src="https://img.shields.io/codacy/grade/1a892d499efd4dabb73beffa8d64ed01?logo=codacy&style=flat-square" alt="Codacy grade"></a>
<a href="https://github.com/dgibbs64/ansible-role-first_contact/actions/workflows/molecule.yml"><img alt="GitHub Workflow Status" src="https://img.shields.io/github/actions/workflow/status/dgibbs64/ansible-role-first_contact/molecule.yml?label=molecule&logo=ansible&style=flat-square"></a>
<a href="https://galaxy.ansible.com/dgibbs64/first_contact"><img alt="Ansible Quality Score" src="https://img.shields.io/ansible/quality/60605?logo=ansible&style=flat-square"></a>
<a href="https://galaxy.ansible.com/dgibbs64/first_contact"><img alt="Ansible Role" src="https://img.shields.io/ansible/role/d/60605?color=EE0000&logo=ansible&style=flat-square"></a>
<a href="https://galaxy.ansible.com/dgibbs64/first_contact"><img alt="GitHub tag (latest by date)" src="https://img.shields.io/github/v/tag/dgibbs64/ansible-role-first_contact?color=EE0000&label=release&logo=ansible&style=flat-square"></a>
<a href="https://github.com/dgibbs64/ansible-role-first_contact/blob/main/LICENSE.md"><img src="https://img.shields.io/github/license/gameservermanagers/docker-steamcmd?style=flat-square" alt="MIT License"></a>
</p>

## About

This role is designed to perform the following actions upon initial contact with a host:

- Check if the Ansible user (first_contact_deploy_user) already exists on the host.
- If the ansible user does not exist, connect to the host as the root user.
  - Create the ansible user on the host.
  - Add the ansible user to the sudoers group, granting them superuser privileges.
  - Add the public key of the control node to the ansible users' authorized keys, allowing the control node to log in without a password.
  - Set the ansible user's shell to the desired shell program.
  - Bootstrap the host using the [robertdebock.bootstrap](https://galaxy.ansible.com/robertdebock/bootstrap) role.
  - Install necessary dependencies for Ansible using the [robertdebock.core_dependencies](https://galaxy.ansible.com/robertdebock/core_dependencies) role.

> The ansible user (first_contact_deploy_user) is the user that runs ansible commands on the host.

## Example Usage

This role is typically used when creating a new virtual private server (VPS) with providers such as [Linode](https://www.linode.com/docs/products/tools/cloud-manager/guides/manage-ssh-keys/), [Digital Ocean](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/), [AWS](https://docs.aws.amazon.com/opsworks/latest/userguide/security-ssh-access.html). During installation, you can normally specify a public key to be added to the root user. You can use this role to log in using the root user and then create the ansible user.

## Requirements

None.

## Role Variables

```yaml
---
first_contact_bypass_host_identity_check: false
first_contact_bypass_host_key_check: true
first_contact_connection_timeout: 3
first_contact_deploy_user: "deploy"
first_contact_deploy_password: "password"
first_contact_ssh_private_key_file: "~/.ssh/id_ed25519"
first_contact_ssh_public_key_file: "~/.ssh/id_ed25519.pub"
```

`first_contact_deploy_user` is the user that ansible will use to log in to the host. Leave unset to use the user that is running ansible.

`first_contact_deploy_password` is the password for the `first_contact_deploy_user` account. Leave unset to generate a random password.

> The below variables are SSH security related. It is important the implications of each are understood.

From the SSH man page:

> ssh automatically maintains and checks a database containing identification for all hosts it has ever been used with. Host keys are stored in ~/.ssh/known_hosts in the user's home directory. Additionally, the file /etc/ssh/ssh_known_hosts is automatically checked for known hosts. Any new hosts are automatically added to the user's file. If a host's identification ever changes, ssh warns about this and disables password authentication to prevent server spoofing or man-in-the-middle attacks, which could other‐wise be used to circumvent the encryption. The StrictHostKeyChecking option can be used to control logins to machines whose host key is not known or has changed.

If `first_contact_bypass_host_key_check` is _true_ SSH [host key checking](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html#managing-host-key-checking) will be bypassed for first contact only. Subsequent checks will be carried out as per `host_key_checking` [setting](https://docs.ansible.com/ansible/latest/user_guide/connection_details.html#managing-host-key-checking). This removes the requirement to manualy type yes to the below message.

```bash
The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
ECDSA key fingerprint is SHA256:1RG/OFcYAVv57kcP784oaoeHcwjvHDAgtTFBckveoHE.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

If `first_contact_bypass_host_identity_check` is _true_ the below message will automatically be resolved. This is useful if your VPS is being rebuilt regularly. However you will no longer be protected by this ssh feature when running this role.

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
51:82:00:1c:7e:6f:ac:ac:de:f1:53:08:1c:7d:55:68.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending RSA key in /home/user/.ssh/known_hosts:12
RSA host key for 192.168.1.1 has changed and you have requested strict checking.
Host key verification failed.
```

## SSH Root access

> Temporary SSH root access is required to create the ansible user. For security reasons this should be disabled once setup.

If you are manually installing your server you may not have the ability to add a public key to the root user in install. When you login as a standard user you can use the following to allow ssh access to root and add your public key to the root user.

```bash
sudo mkdir -p /root/.ssh; sudo chmod 700 /root/.ssh; sudo touch /root/.ssh/authorized_keys; sudo chmod 600 /root/.ssh/authorized_keys; sudo echo "<insert public key>" > /root/.ssh/authorized_keys; sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config; sudo systemctl restart sshd
```

Once first contact has been established, you can disable root access again. By using the following or a role such as [geerlingguy.security](https://galaxy.ansible.com/geerlingguy/security) as part of your playbook.

```bash
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config; sudo systemctl restart sshd
```

## Dependencies

- robertdebock.bootstrap
- robertdebock.core_dependencies

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
