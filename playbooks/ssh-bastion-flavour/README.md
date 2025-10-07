# SSH bastion flavour
The [SSH](https://en.wikipedia.org/wiki/Secure_Shell) bastion or proxy server
is a barrier between your internal machines (which lack a public or floating IP
address) and the public internet. With the SSH proxy, you'll have an extra layer of
security on top of your instances. It's equipped with
[Fail2ban](https://github.com/fail2ban/fail2ban),
intrusion prevention software designed to prevent brute-force attacks.

This template is for tenant admins wishing to hardening the way tenant users
connect to the [European Weather Cloud (EWC)](https://europeanweather.cloud/),
as well as tenant users whom are mindful about safe-keeping the compute resources
or data withing their work environments.

## Functionality
The template is designed to:

* Configure a pre-existing virtual machine running RockyLinux, with public IP
address, and a minimum recommended 4GB of RAM, as entrypoint for users who
wish to reach private EWC networks, from the public internet, via SSH.

## Prerequisites

To successfully run this playbook, the following packages should be available in your work environment:

| Name | Version | License | Home URL |
|------|---------|----- |-----|
| git | >= 2.0 | GPLv2  | https://git-scm.com/downloads |
| python | >= 3.9   | PSF | https://www.python.org/downloads  |
| ansible | >= 2.15 |  GPLv3+ | https://pypi.org/project/ansible  |

## Usage

### 1. Download  Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install -r requirements.yml
```

### 2. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ssh_bastion:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: cloud-user
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 3. Configure and apply the template

#### 3.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml ssh-bastion-flavour.yml
```

#### 3.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For
example:

```bash
ansible-playbook \
  -i inventory.yml \
  -e '{"fail2ban_whitelisted_ip_ranges":""}' \
  ssh-bastion-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `''` | no |

## Dependencies

> ⚠️ Only RockyLinux 9.5 and RockyLinux 8.10 instances are currently supported due
to constrains imposed by the required ewc-ansible-role-ssh-bastion Ansible
Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License |Home URL |
|------|---------|-------|-------|
| ewc-ansible-role-ssh-bastion | 1.4 | MIT | https://github.com/ewcloud/ewc-ansible-role-ssh-bastion |

## Operation
Checkout the following how-to guides to learn about management of Item after initial setup:
* [How to connect to a VM hidden behind the SSH Bastion](./docs/how-to/how-to-connect-to-a-vm-hidden-behind-the-ssh-bastion.md)
