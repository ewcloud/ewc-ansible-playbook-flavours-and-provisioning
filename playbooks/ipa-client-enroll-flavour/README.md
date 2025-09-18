# IPA client enrollment flavour

This Ansible Playbook configures an existing virtual machine in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/) to operate as a
client of [IPA services](../ipa-server-flavour/).

IPA provides integrated identity management and DNS services, enabling
centralized user authentication, authorization, and resource discovery.

Suitable for tenant admins and tenant users alike, this template simplifies the
integration of VMs into a [FreeIPA](https://www.freeipa.org/page/Main_Page)-managed
fleet of instances within the EWC environment. Follow the [instructions below](#usage)
to enroll your instance.

## Functionality
The template is designed to:
- Configure a pre-existing virtual machine running Rocky Linux 8/9 or Ubuntu to connect to a
IPA server on the same subnet.
- Enable DNS resolution for discovering private hosts and public addresses.
- Allow remote access to the VM using centrally managed LDAP users via password authentication.

## Prerequisites
>💡 Versions listed correspond to minimal prerequisites.

To successfully run this playbook, the following packages should be available in your work environment:

| Name | Version | License | Home URL |
|------|---------|----- |-----|
| git | 2.0 | GPLv2  | https://git-scm.com/downloads |
| python | 3.9   | PSF | https://www.python.org/downloads  |
| ansible | 2.15 |  GPLv3+ | https://pypi.org/project/ansible  |

## Usage

### 1. Download  Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install -r requirements.yml
```

### 2. Specify the target host and SSH credentials
>💡 To find out which is the default user for your chosen VM image,
checkout the [official EWC documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+VM+images+and+default+users).

Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ipa_client:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add the default user according to your chosen VM image>
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new

```

### 3. Configure and apply the template

#### 3.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml ipa-client-enroll-flavour.yml
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
  -e '{
        "ipa_client_hostname": "ipa-client-1",
        "ipa_domain": "eumetsat.sandbox.ewc",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "ipaadmin",
        "ipa_admin_password": "my-secret-password"
    }' \
  ipa-client-enroll-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ipa_client_hostname | hostname of the target vm where the IPA client will be installed. Example: `ipa-client-1` | `string`| n/a | yes |
| ipa_domain | domain name managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server. Example: `ipa-server-1` | `string`| n/a | yes |
| ipa_admin_username | username of the administrator account from the IPA server. Example: `ipaadmin` | `string` | n/a | yes |
| ipa_admin_password | password of the administrator account from the IPA server. Example: `my-secret-password` | `string` | n/a | yes |

## Dependencies
> ⚠️ Only Ubuntu 22.04 and RockyLinux 8.10 VM images are currently supported.
This is due to constrains imposed by the required
ewc-ansible-role-ipa-client-enroll Ansible Role.

| Name | Version | License |Home URL |
|------|---------|------|-----|
| ewc-ansible-role-ipa-client-enroll | 1.1 | MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-client-enroll |
