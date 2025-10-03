# IPA server flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/)
to operate as a [FreeIPA](https://www.freeipa.org/page/Main_Page) server.

IPA (acronym for identity, policy and audit), provides integrated identity
management and DNS services, enabling centralized user authentication, authorization,
and resource discovery.

Ideal for tenant administrators, this template simplifies the setup
of a secure, open-source identity and DNS solution in the EWC environment. Follow the
[instructions below](#usage) to configure your server.

## Functionality
The template is designed to:
* Validate that network/subnet configuration in the EWC tenancy
* Configure a pre-existing virtual machine running RockyLinux version 8 or 9,
and with a minimum recommended 4GB of RAM, such that it:
  * Provides DNS resolutions for discovery of resources (i.e. other virtual
  machines)
  * Enables centralized user and credentials creation/edition/deletion/authentication
  * Allows centralized authorization between users and resources
* Automatically update the underlying subnet DNS nameserver to point to the
newly configured IPA server

>⚠️ Successful execution leads to changes of the DNS nameserver(s) in your
OpenStack subnet (includes now only the IP address of the new IPA server).
This can negatively affect existing VMs within your subnet.
To prevent issues, programatically update each VM via the
[IPA Client Enroll Flavour](https://europeanweather.cloud/community-hub/ipa-client-enroll-flavour)
CommunityHub Item. Alternatively, you can manually
[add the new nameserver](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/identity_management_guide/domain-dns)
to their DNS configuration.

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
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ipa_server:
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
ansible-playbook -i inventory.yml ipa-server-flavour.yml
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
      "ipa_domain": "eumetsat.sandbox.ewc",
      "ipa_server_hostname": "ipa-server-1",
      "ipa_admin_username": "ipaadmin",
      "ipa_admin_password": "my-secret-password",
      "ipa_admin_givenname": "EWC",
      "ipa_admin_surname": "IPAADMIN",
      "os_network_name": "private",
      "os_security_group_name": "ipa"
    }' \
  ipa-server-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ipa_domain | domain name to be managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the target vm where the IPA server will be installed | `string`| `ipa-server-1` | yes |
| ipa_admin_username | username of administrator account to replace the default IPA admin | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of administrator account to replace the default IPA admin. Example: `my-secret-password` | `string` | n/a | yes |
| ipa_admin_givenname | given name of the administrator to replace the default IPA admin (not necessarily a real person's name) | `string` | `EWC` | yes |
| ipa_admin_surname | surname of the administrator to replace the default IPA admin (not necessarily a real person's name) | `string` | `IPAADMIN` | yes |
| os_network_name | OpenStack network to which the target virtual machine has access to | `string` | `private` | yes |
| os_security_group_name | OpenStack security group containing all firewall rules required by the IPA server/client communication | `string` | `ipa` | yes |

## Dependencies
> ⚠️ Only RockyLinux 9.5 and RockyLinux 8.10 VM images are currently supported.
This is due to constrains imposed by the required ewc-ansible-role-ipa-server
Ansible Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|------|------|
| ewc-ansible-role-ipa-server | 1.0 |  MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-server |
