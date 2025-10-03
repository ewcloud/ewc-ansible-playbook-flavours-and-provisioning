# Remote desktop flavour

This Ansible Playbook configures virtual machines within the
[European Weather Cloud (EWC)](https://europeanweather.cloud/) to
operate as a remote desktops, courtesy of [X2Go](https://wiki.x2go.org/doku.php).

X2Go enables secure, graphical access to a desktop environment over low
or high bandwidth connections, providing a seamless user experience for
remote work. This template equips your VM with utility software you
would expect to see in a typical and stable Linux distribution, ideal
efficient and intuitive desktop operation.

Special for tenant users needing remote graphical access in their EWC
environment, this template simplifies the setup of basic cloud development
solution. Follow the [instructions below](#usage) to get started.

## Functionality
The template is designed to:
- Configure a pre-existing Rocky Linux virtual machine (minimum 4GB RAM recommended) with
the [MATE desktop environment](https://mate-desktop.org/).
- Install and set up X2Go for secure remote desktop access over varying network conditions.
- Enable end-users to interact with the VM through a graphical interface using the X2Go client
application.

## Prerequisites

To successfully run this playbook, the following packages should be available in your work environment:

| Name | Version | License | Home URL |
|------|---------|----- |-----|
| git | >=2.0 | GPLv2  | https://git-scm.com/downloads |
| python | >=3.9   | PSF | https://www.python.org/downloads  |
| ansible | >=2.15 |  GPLv3+ | https://pypi.org/project/ansible  |

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
    remote_desktop:
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
ansible-playbook -i inventory.yml remote-desktop-flavour.yml
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
  -e '{"fail2ban_whitelisted_ip_ranges": ""}' \
  remote-desktop-flavour.yml
```

### 4. Install the local client and connect to your remote desktop
>⚠️ When configuring a connection, be sure to select "MATE" (instead of
"KDE" or any other options) in the `Session Type` drop-down list, towards the
bottom of the `Session` tab. This is required for the local client to correctly
communicate with your remote desktop.

Install the remote desktop client on Microsoft Window, Mac OS or Linux by
following the links on the [official X2Go installation page](https://wiki.x2go.org/doku.php/doc:installation:x2goclient). Then follow the [official X2Go client usage page](https://wiki.x2go.org/doku.php/doc:usage:x2goclient)
if you do not know how to configure a new session.

For a session creation
example, representative of a typical EWC environment, checkout the Remote
Desktop section of
[this official EWC documentation page](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETSAT+tenancy%3A+Default+setup).

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `''` | no |


## Dependencies

> ⚠️ Only RockyLinux 8.10 and 9.5 instances are currently supported due
to constrains imposed by the required ewc-ansible-role-remote-desktop Ansible
Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License |Home URL |
|------|---------|-------|---|
| ewc-ansible-role-remote-desktop | 1.2 | MIT |  https://github.com/ewcloud/ewc-ansible-role-remote-desktop |
